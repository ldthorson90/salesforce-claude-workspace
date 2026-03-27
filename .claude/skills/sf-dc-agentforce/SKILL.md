---
name: sf-dc-agentforce
description: Design and implement the Data Cloud → Agentforce activation pipeline — unified profile grounding, segment-gated agents, calculated insights as agent signals, activation triggers, identity alignment, Trust Layer data masking, and end-to-end pipeline testing.
user-invocable: true
---

# Data Cloud → Agentforce Pipeline

Connect Data Cloud unified profiles, segments, and calculated insights to Agentforce agents.
This skill covers the integration layer between the two products — the parts that don't
belong cleanly in `/sf-data-cloud` or `/sf-agentforce` alone.

## Arguments

- `design` — architecture overview and connection checklist
- `grounding` — configure Data Cloud as a grounding source in Prompt Builder
- `segments` — use segment membership to gate agent availability or behavior
- `activations` — trigger Agentforce actions from Data Cloud activation targets
- `trust` — Trust Layer masking rules for Data Cloud PII in prompt templates
- `test` — validate the end-to-end pipeline: grounding preview, segment conditions, DC query performance

---

## Architecture Overview (`design`)

```
Data Cloud                             Agentforce
──────────────────────────────────     ──────────────────────────────────
Unified Profile                ──────→ Prompt Builder grounding source
  └── Individual fields                  └── Flex template merge fields
  └── Related DMO data                   └── RAG / vector search

Segments                       ──────→ Agent Builder entry conditions
  └── Segment membership                  └── Topic availability gates
  └── Streaming segments                  └── Real-time eligibility

Calculated Insights            ──────→ Apex @InvocableMethod actions
  └── Churn score                         └── Einstein Next Best Action
  └── LTV, RFM, affinity                  └── Action input variables

Activation Targets             ──────→ Platform Events / Flow triggers
  └── Segment entry/exit                  └── Agentforce workflow entry
  └── CRM write-back                      └── Proactive outreach agents
```

### Prerequisites

| Requirement | Why |
|---|---|
| Data Cloud provisioned and licensed | Grounding, segmentation |
| Agentforce provisioned and licensed | Agent Builder, Prompt Builder |
| CRM Connector active | Identity alignment (Individual → Contact/Lead) |
| Identity resolution ruleset run | Unified Individual ID exists before agents query it |
| Einstein Trust Layer enabled | Required for any prompt template that touches DC data |

### Decision Points

| Question | Implication |
|---|---|
| Do agents need real-time profile data? | Use streaming segments + Ingestion API; batch segments add latency |
| Is PII flowing into prompts? | Trust Layer masking rules are required before go-live |
| Are insights pre-computed or on-demand? | Calculated insights (pre-computed) are faster; Apex queries are flexible |
| Who runs the agent? | Running user's permissions govern what DC data the agent can read |

---

## Data Cloud Grounding (`grounding`)

### Connect Data Cloud as a Grounding Source

1. In Prompt Builder, open a Flex template.
2. Under **Data Sources**, select **+ Add Data Source → Data Cloud**.
3. Choose the Data Model Object (DMO) to expose — start with `Individual__dlm`.
4. Map fields needed in the template (only expose fields the LLM needs).
5. Set the **join key**: typically `Individual__dlm.Id__c` matched to `{!Record.DataCloudIndividualId}` or a Contact/Lead field that holds the resolved Individual ID.

### Unified Profile Fields in Flex Templates

```
Prompt Template: Customer_Context_Summary
  Type: Flex
  Inputs:
    - contactId: Id
  Data Sources:
    - Contact: Id, Name, AccountId
    - Individual__dlm (Data Cloud): LTV__c, ChurnScore__c, PreferredChannel__c,
        LastPurchaseDate__c, TotalOrders__c
  Join: Contact.DataCloudIndividualId → Individual__dlm.Id__c
  Template:
    "You are assisting a service agent. Here is context for {!Contact.Name}:
     - Lifetime value: {!Individual__dlm.LTV__c}
     - Churn risk score: {!Individual__dlm.ChurnScore__c} (0–100, higher = higher risk)
     - Preferred channel: {!Individual__dlm.PreferredChannel__c}
     - Last purchase: {!Individual__dlm.LastPurchaseDate__c}

     Use this context to personalize your response. Do not quote scores verbatim
     to the customer."
```

### RAG with Data Cloud Vector Search

Use when grounding requires retrieving from a large corpus (product catalog, knowledge,
policy documents ingested into Data Cloud):

1. Create a Data Model Object with a vector field (`Text_Embedding__c`, type: VectorField).
2. Ingest documents via Ingestion API with pre-computed embeddings (or use Einstein embedding).
3. In Prompt Builder, select **Retrieve** action → Data Cloud Vector Search.
4. Configure:
   - **Index**: the DMO + vector field
   - **Top K**: 3–5 (diminishing returns above 5 for most prompts)
   - **Query source**: the customer's incoming message or a reformulated search query
5. Inject retrieved chunks into the template as `{!VectorSearchResults}`.

**Rules:**
- Never expose a vector field directly as a merge field — it is for retrieval only.
- Cap Top K at 5; larger retrieval sets inflate token cost without improving accuracy.
- Always include a fallback instruction: "If no relevant documents are found, say so."
- Test retrieval quality in Prompt Builder preview before attaching to an agent topic.

### Grounding Field Exposure Rules

- Expose only fields the LLM needs for the current topic — not the full unified profile.
- Exclude fields governed by data masking rules (see `trust` section).
- Do not expose raw match keys (email, phone) in templates — use display-safe fields only.
- Document which DMO fields are exposed per template in the solution design.

---

## Segments as Agent Conditions (`segments`)

### Segment Membership as Entry Condition

**Pattern A: CRM Write-Back (recommended for most cases)**

1. In Data Cloud, create an **Activation Target** of type **Salesforce CRM**.
2. Map segment membership to a Contact/Lead field:
   - Create custom field: `Contact.DC_Segment__c` (Multi-Select Picklist or Text)
   - Or per-segment Boolean fields: `Contact.Is_Gold_Tier__c`, `Contact.Is_ChurnRisk__c`
3. Configure CRM activation to write segment membership on entry/exit.
4. In Agent Builder, add a Topic **Entry Condition**:
   ```
   Topic: Upgrade Offer Agent
   Entry Condition: {!Contact.Is_Gold_Tier__c} = true
                    AND {!Contact.Is_ChurnRisk__c} = false
   ```

**Pattern B: Real-Time Segment Check via Apex Action**

Use when segments refresh too slowly for write-back or for streaming segments:

```apex
public class DataCloudSegmentChecker {
    @InvocableMethod(
        label='Check Data Cloud Segment Membership'
        description='Returns true if the unified individual is a member of the named segment'
    )
    public static List<SegmentResult> checkSegment(List<SegmentRequest> requests) {
        List<SegmentResult> results = new List<SegmentResult>();
        for (SegmentRequest req : requests) {
            List<Contact> contacts = [
                SELECT Id, DataCloudIndividualId, DC_Segment__c
                FROM Contact
                WHERE Id = :req.contactId
                WITH USER_MODE
                LIMIT 1
            ];
            Boolean isMember = !contacts.isEmpty()
                && contacts[0].DC_Segment__c != null
                && contacts[0].DC_Segment__c.contains(req.segmentName);
            results.add(new SegmentResult(req.contactId, isMember));
        }
        return results;
    }

    public class SegmentRequest {
        @InvocableVariable(required=true description='Contact or Lead ID to check')
        public Id contactId;
        @InvocableVariable(required=true description='Segment API name to check membership')
        public String segmentName;
    }

    public class SegmentResult {
        @InvocableVariable public Id      contactId;
        @InvocableVariable public Boolean isMember;
    }
}
```

### Segment Refresh Timing

| Segment Type | Refresh | Use Pattern |
|---|---|---|
| Batch (daily) | Up to 24h lag | Pattern A write-back |
| Batch (hourly) | Up to 1h lag | Pattern A write-back |
| Streaming | Near real-time | Pattern B Apex check |

**Rules:**
- Never gate a time-sensitive offer on a daily batch segment.
- CRM write-back fields must have FLS granted to the agent's running user.
- Include segment exit logic: clear CRM write-back fields on segment exit.
- Name write-back fields with `DC_` prefix to distinguish from CRM-native fields.

---

## Calculated Insights as Agent Signals (`activations`)

### Surface Insight Values to Agents

**Option 1: CRM Write-Back (simplest)**

```
Data Cloud Calculated Insight: Customer_LTV
  Fields: IndividualId, LifetimeValue, OrderCount, ChurnScore
  Activation: CRM Write-Back
    → Contact.DC_LTV__c        (Currency)
    → Contact.DC_OrderCount__c (Number)
    → Contact.DC_ChurnScore__c (Percent)
  Refresh: Daily at 5 AM
```

**Option 2: Apex Action**

```apex
public class CustomerInsightsAction {
    @InvocableMethod(
        label='Get Customer Insights'
        description='Returns Data Cloud calculated insight values for the customer'
    )
    public static List<InsightResult> getInsights(List<Id> contactIds) {
        List<InsightResult> results = new List<InsightResult>();
        List<Contact> contacts = [
            SELECT Id, DC_LTV__c, DC_ChurnScore__c, DC_OrderCount__c,
                   DC_PreferredChannel__c, DC_LastPurchaseDate__c
            FROM Contact
            WHERE Id IN :contactIds
            WITH USER_MODE
        ];
        for (Contact c : contacts) {
            InsightResult r     = new InsightResult();
            r.contactId         = c.Id;
            r.lifetimeValue     = c.DC_LTV__c;
            r.churnScore        = c.DC_ChurnScore__c;
            r.orderCount        = (Integer) c.DC_OrderCount__c;
            r.preferredChannel  = c.DC_PreferredChannel__c;
            r.lastPurchaseDate  = c.DC_LastPurchaseDate__c;
            r.churnRiskLabel    = c.DC_ChurnScore__c >= 70 ? 'High'
                                : c.DC_ChurnScore__c >= 40 ? 'Medium' : 'Low';
            results.add(r);
        }
        return results;
    }

    public class InsightResult {
        @InvocableVariable public Id      contactId;
        @InvocableVariable public Decimal lifetimeValue;
        @InvocableVariable public Decimal churnScore;
        @InvocableVariable public Integer orderCount;
        @InvocableVariable public String  preferredChannel;
        @InvocableVariable public Date    lastPurchaseDate;
        @InvocableVariable public String  churnRiskLabel;
    }
}
```

**Rules:**
- Pre-compute complex scores in Data Cloud; never compute at agent runtime.
- Derive human-readable labels in Apex — don't pass raw floats to the LLM.
- Include a staleness check: if `DC_LastSyncDate__c` > 48h, flag the data as stale.
- Return null-safe values; unified profiles may be incomplete for some individuals.

### Activation Targets that Trigger Agentforce Workflows

```
Data Cloud Activation Target: Churn_Risk_Entry
  Type: Platform Event
  Trigger: Segment entry (ChurnRisk_High)
  Payload: IndividualId, ContactId, SegmentName, EntryTimestamp

Platform Event: DC_SegmentEvent__e
  Fields: IndividualId__c, ContactId__c, SegmentName__c, EventType__c (entry/exit)

Receiving Flow:
  Trigger: DC_SegmentEvent__e
  → Get Contact by ContactId
  → Evaluate tier / eligibility
  → Create Agentforce task or trigger outbound agent
```

---

## Identity Resolution Alignment

Correct identity mapping is the prerequisite for everything in this skill.
If the Individual ID is wrong, agents get the wrong profile.

### Alignment Checklist

- [ ] CRM Connector active and syncing Contact and Lead to Data Cloud
- [ ] Identity resolution ruleset covers email, phone, and name+address rules
- [ ] `Contact.DataCloudIndividualId` populated after identity resolution runs
- [ ] Reconciliation rule for `DataCloudIndividualId` set to **Most Recent** (CRM wins)
- [ ] Guest/anonymous profiles excluded from agent grounding (no Individual ID = no DC grounding)

### Verify Alignment

```apex
List<Contact> unresolved = [
    SELECT Id, Name, Email
    FROM Contact
    WHERE DataCloudIndividualId = null
    AND CreatedDate = LAST_N_DAYS:30
    WITH USER_MODE
    LIMIT 50
];
System.debug('Unresolved contacts (last 30d): ' + unresolved.size());
```

**Rules:**
- Run identity resolution on a schedule before the agent's peak usage window.
- Contacts without `DataCloudIndividualId` must degrade gracefully — agent still works, just without DC enrichment.
- Never use email address as the join key in prompt templates — use the resolved Individual ID.

---

## Trust Layer for Data Cloud Data (`trust`)

### Fields Requiring Masking

| Field Category | Examples | Masking Type |
|---|---|---|
| Direct identifiers | Email, Phone, SSN | Full mask or tokenize |
| Financial | Credit score, account balance | Partial mask or range label |
| Health/sensitive | Medical flags, behavioral scores | Full mask |
| Precise location | GPS coordinates, full address | City/region only |

### Configure Masking Rules

1. Setup → Einstein Trust Layer → Data Masking
2. Add rule by DMO field API name
3. Select strategy: **Mask**, **Tokenize**, or **Replace with label**
4. Assign to prompt templates that use the field
5. Verify in Prompt Builder preview — masked fields show `[MASKED]`, not raw values

### Audit Trail

```
Einstein Trust Layer → Audit Trail
  Enabled: Yes
  Log: All grounded responses (Data Cloud grounding)
  Retention: 90 days
```

Captures: template invoked, DC fields injected (names only if masked), LLM response, running user, timestamp.

### Trust Layer Rules

- Masking rules must be configured before UAT — not after.
- Audit trail must be enabled before any DC-grounded agent goes to production.
- Guest user agents: DC grounding is not permitted.
- Zero-data retention: verify it's active on the specific model used for DC grounding.
- Document which fields are masked and why in the solution design for compliance handoff.

---

## Testing the Pipeline (`test`)

### 1. Prompt Builder Preview

```
Prompt Builder → [Template] → Preview with a Contact that has DataCloudIndividualId set
  [ ] DC fields render with real values (not blank)
  [ ] Masked fields show [MASKED]
  [ ] Executes within 10-second timeout
  [ ] No "null" strings in output (null-safe merge fields)
```

If DC fields are blank: verify `DataCloudIndividualId` is populated, check FLS on DMO fields, confirm identity resolution has run since last data stream refresh.

### 2. Segment Conditions in Agent Testing Center

| Test Case | Input | Expected |
|---|---|---|
| In-segment customer | `Is_Gold_Tier__c = true` | Topic available |
| Out-of-segment customer | `Is_Gold_Tier__c = false` | Topic unavailable, graceful fallback |
| Missing Individual ID | No `DataCloudIndividualId` | Agent works without DC enrichment |
| Streaming segment | Real-time entry | Topic available within 30 seconds |

```bash
sf agent test run -o <org-alias> --result-format human
```

### 3. Insight Values Spot-Check

```apex
List<CustomerInsightsAction.InsightResult> results =
    CustomerInsightsAction.getInsights(new List<Id>{ '<contact-id>' });
System.debug('LTV: ' + results[0].lifetimeValue);
System.debug('Churn: ' + results[0].churnScore + ' (' + results[0].churnRiskLabel + ')');
```

### 4. End-to-End Smoke Test

```
1. Ingest test record via Ingestion API → confirm in DC
2. Verify identity resolution merges with existing Contact
3. Confirm Contact.DataCloudIndividualId is populated
4. Confirm calculated insight write-back fields are set
5. Confirm segment membership write-back field is correct
6. Prompt Builder preview with that Contact → DC fields render
7. Agent conversation as that Contact → topic available
8. Audit trail entry appears in Einstein Trust Layer
```

### Governor Limits

| Limit | Value | Implication |
|---|---|---|
| Prompt Builder grounding timeout | 10 seconds | DC queries must complete in < 8s |
| Platform Events per transaction | 150 | Activation webhooks safely within limit for batch activations |
| SOQL rows per transaction | 50,000 | CRM write-back queries scoped to segment members |
| Streaming segment lag | ~30 seconds | Not suitable for instant eligibility decisions |

---

## Metadata Locations

| Artifact | Location |
|---|---|
| Prompt Templates (DC grounding) | `force-app/main/default/genAiPromptTemplates/` |
| Agent Actions (Apex) | `force-app/main/default/classes/` |
| Platform Event (activation trigger) | `force-app/main/default/objects/<EventName__e>/` |
| CRM write-back fields | `force-app/main/default/objects/Contact/fields/DC_*.field-meta.xml` |

## Context7 References

Query `/damecek/salesforce-documentation-context` for:
- "Data Cloud Agentforce grounding prompt builder unified profile"
- "Einstein Trust Layer data masking audit trail"
- "Data Cloud activation target segment membership CRM write-back"
