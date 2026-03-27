---
name: sf-agentforce
description: Build and configure Salesforce Agentforce — AI agents, prompt templates, agent actions, Einstein Trust Layer, agent testing, Models API, and governance. Aligned to AI Specialist and AI Associate certifications. Agentforce is GA as of Winter '25.
user-invocable: true
---

# Salesforce Agentforce

Build, configure, test, and govern AI agents on the Salesforce platform. Agentforce is generally available as of Winter '25.

## Arguments

- `agent <name>` — create or configure an Agentforce agent (Service, Sales, or Custom)
- `action <name>` — author an agent action (Flow, Apex, Prompt, External Service)
- `prompt-template <name>` — create a Prompt Template (Flex, Sales Email, Field Generation)
- `test <agent-name>` — design test scenarios using Agentforce Testing Center
- `trust` — review Einstein Trust Layer configuration and audit setup
- `models` — compare model options (standard, BYOM), configure Models API
- `governance` — review Trust Layer policies, audit logs, escalation routing

Invoke without arguments for a full Agentforce implementation walkthrough.

---

## Agent Architecture

```
Agent (GenAiAgent)
├── Topics — scope what the agent can handle
│   ├── Instructions — natural language guidance per topic
│   ├── Actions — what the agent can DO
│   │   ├── Standard Flow Actions (invocable flows)
│   │   ├── Apex Actions (@InvocableMethod)
│   │   ├── Prompt Template Actions
│   │   ├── External Service Actions (named credentials)
│   │   └── MCP Server Actions (external tools via Model Context Protocol)
│   └── Guardrails — what the agent must NOT do
├── Channels — where the agent is deployed
│   ├── Messaging (web chat, WhatsApp, SMS via Messaging for In-App and Web)
│   ├── Voice (Einstein Voice via Amazon Connect / Genesys)
│   ├── Slack (Agentforce for Slack)
│   ├── Email (Agentforce for Email)
│   └── API (headless, for custom front-ends)
└── Einstein Trust Layer — safety and compliance wrapper
    ├── Data masking (PII before LLM)
    ├── Toxicity scoring
    ├── Audit trail
    └── Zero data retention enforcement
```

### Agent Types (GA)

| Type | Use Case | Primary Builder |
|---|---|---|
| Service Agent | Case deflection, customer self-service, knowledge lookup | Agent Builder (declarative) |
| Sales Development Rep (SDR) | Prospect outreach, lead qualification, meeting scheduling | Agent Builder + Sales Cloud |
| Sales Coach | Opportunity coaching, call debrief, deal review | Agent Builder + Einstein Conversation Insights |
| Pipeline Inspection Agent | Forecast risk, pipeline updates, rep coaching prompts | Agent Builder + Revenue Intelligence |
| Custom Agent | Domain-specific internal or external workflows | Agent Builder + code actions |

**Einstein Bots migration:** Existing Einstein Bots can be migrated to Agentforce Service Agents. Use the Bot-to-Agent Migration Tool in Setup > Einstein Bots to convert dialog flows to topics and actions.

---

## Agent Builder (Declarative)

### Topic Design

Each topic represents a distinct capability domain. The agent's reasoning model selects the correct topic based on user intent.

```yaml
Topic: Order Management
  Description: "Help customers check order status, modify orders, and process returns"
  Instructions:
    - "Always verify customer identity before sharing order details"
    - "For returns, check if the order is within the 30-day return window"
    - "Escalate to a human agent if the customer expresses frustration or asks to speak to a person"
    - "Never process refunds over $500 without routing to a human agent"
  Actions:
    - Get_Order_Status (Flow)
    - Process_Return (Flow)
    - Update_Shipping_Address (Apex)
  Guardrails:
    - "Never share another customer's order information"
    - "Do not make commitments about delivery dates you cannot verify"
```

**Topic design rules:**
- One topic per capability domain — do not overload topics
- Instructions: specific, imperative ("Do X", "Never Y", "Always Z")
- 3–5 actions per topic maximum (action selection accuracy degrades above 5)
- Every topic that touches customer data must have an escalation path
- Guardrails in the topic supplement the Trust Layer — they are not a replacement

### Human Escalation (Omnichannel Handoff)

For Agentforce for Service, configure handoff to a human agent via Omnichannel:

```yaml
Escalation Action: Transfer_to_Agent
  Trigger conditions:
    - Customer requests a human
    - Agent cannot resolve after 2 attempts
    - Sentiment score below threshold
  Handoff payload:
    - Conversation transcript
    - Customer record (contact/account)
    - Suggested queue or skill group
  Target: Omnichannel routing (Skill-Based or Queue-Based)
```

**Rules:**
- Always pass the conversation transcript on handoff — agents should not repeat questions
- Route to the correct skill group based on conversation context
- Set `transferToAgentAction` in the topic's escalation instruction

---

## Agent Actions

### Action Types

| Type | Best For | Latency |
|---|---|---|
| Flow Action | Declarative logic, SOQL, DML, no-code steps | Medium |
| Apex Action | Complex logic, external callouts, bulkified operations | Medium |
| Prompt Template Action | LLM-assisted summarization or generation within an action | Higher |
| External Service Action | Third-party REST APIs via Named Credential + OpenAPI spec | Variable |
| MCP Server Action | External tools via Model Context Protocol (experimental) | Variable |

### Flow Actions (preferred for declarative)

```
Flow: Get_Order_Status
  Type: Autolaunched
  Input Variables:
    - orderId {Text} — Available for Input
    - customerId {Text} — Available for Input
  Steps:
    1. Get Records: Order WHERE Id = {orderId}
    2. Decision: order found? → null check
    3. Get Records: OrderItems WHERE OrderId = {orderId}
    4. Assignment: build output string
  Output Variables:
    - orderStatus {Text} — Available for Output
    - itemList {Text}
    - estimatedDelivery {Text}
  Fault Path: assign error message to output variable; do not throw unhandled fault
```

### Apex Actions

```apex
public class OrderActions {
    @InvocableMethod(
        label='Process Return'
        description='Initiates a return for a given order. Use when the customer wants to return an item.'
        category='Order Management'
    )
    public static List<ReturnResult> processReturn(List<ReturnRequest> requests) {
        List<ReturnResult> results = new List<ReturnResult>();
        for (ReturnRequest req : requests) {
            results.add(processOneReturn(req));
        }
        return results;
    }

    private static ReturnResult processOneReturn(ReturnRequest req) {
        ReturnResult result       = new ReturnResult();
        result.orderId            = req.orderId;
        result.status             = 'Return initiated';
        result.confirmationNumber = 'RET-' + String.valueOf(Datetime.now().getTime());
        return result;
    }

    public class ReturnRequest {
        @InvocableVariable(required=true label='Order ID' description='Salesforce ID of the order to return')
        public Id orderId;
        @InvocableVariable(label='Return Reason' description='Customer-provided reason for the return')
        public String reason;
    }

    public class ReturnResult {
        @InvocableVariable(label='Order ID')
        public Id orderId;
        @InvocableVariable(label='Status' description='Result of the return initiation')
        public String status;
        @InvocableVariable(label='Confirmation Number')
        public String confirmationNumber;
    }
}
```

**Apex action rules:**
- `label` and `description` on `@InvocableMethod` drive action selection — make them precise
- `description` on every `@InvocableVariable` — the agent uses these to understand what to pass
- Bulkify: accept `List<InputClass>`, return `List<OutputClass>`
- One action = one capability; avoid multi-purpose actions
- Return structured, human-readable output the LLM can interpret
- Handle exceptions internally and return an error status — never let an uncaught exception surface to the agent

### External Service Actions

1. Create Named Credential (principal: Named Principal or Per-User)
2. Create External Credential with the authentication scheme
3. Import OpenAPI spec via External Services in Setup
4. Imported operations become available as agent actions
5. Map input/output fields in Agent Builder

---

## Prompt Templates

Prompt Builder is the GA tool for creating, testing, and versioning prompt templates.

### Template Types

| Type | Use Case |
|---|---|
| **Flex** | Flexible templates for agent actions, custom logic, and ad-hoc generation |
| **Sales Email** | Outreach emails grounded in Lead/Contact/Opportunity data |
| **Field Generation** | Auto-populate CRM fields from surrounding context |
| **Record Summary** | Summarize records for agent handoff or rep briefing |

### Flex Template Example

```
Template: Summarize_Case_History
  Type: Flex
  Resource: Case (Id, Subject, Description, Status, Priority, AccountId)
  Related Resources:
    - CaseComments (CommentBody, CreatedDate, CreatedBy.Name) — limit 10, DESC
    - Account (Name, Industry)

  Prompt body:
  ---
  You are a helpful service assistant. Summarize the following case for a human agent
  who is picking it up mid-conversation.

  Case: {!Case.Subject} | Status: {!Case.Status} | Priority: {!Case.Priority}
  Account: {!Account.Name} ({!Account.Industry})
  Description: {!Case.Description}

  Recent activity (newest first):
  {!CaseComments}

  Provide a 2–3 sentence summary covering: current status, what has been tried,
  and the recommended next step.
  ---
```

### Data Cloud Grounding

When the org has Data Cloud, ground Flex templates on unified profiles:

```
Grounding Source: Data Cloud — Unified Individual
  Fields: LTV (Calculated Insight), LastPurchaseDate, ChurnRiskScore, PreferredChannel

  Template snippet:
  "Customer lifetime value: {!Individual.LTV}
   Last purchase: {!Individual.LastPurchaseDate}
   Preferred channel: {!Individual.PreferredChannel}"
```

Requires `DataCloudIndividualId` populated on the Contact. See `/sf-dc-agentforce` for the full grounding pipeline.

### RAG Patterns

| Pattern | Mechanism | Use When |
|---|---|---|
| Knowledge Base RAG | Knowledge Action → inject article context into prompt | Structured knowledge articles exist |
| Data Cloud Vector Search | Embed query → nearest-neighbor → inject chunks | Large unstructured corpus in DC |
| CRM-Grounded Response | SOQL retrieval → serialize records as context | Answering from specific CRM records |

**Rules:**
- Always include a fallback instruction: "If no relevant context is found, say so."
- Test retrieval quality in Prompt Builder preview before attaching to an agent topic
- Cap vector search Top K at 5

### Prompt Builder Workflow

1. Open Prompt Builder (Setup > Prompt Builder)
2. Select template type; add resource records and related lists
3. Write the prompt body using `{!Object.Field}` merge fields
4. Preview with a real record — validate output and Trust Layer masking
5. Save and version (name with version suffix for iteration)
6. Attach to agent action or expose as a Prompt Action in Agent Builder

---

## Einstein Trust Layer

The Trust Layer is always active for Agentforce; it cannot be disabled.

### Configuration Checklist

- [ ] **Data masking rules** — define PII fields to mask before LLM calls
- [ ] **Toxicity detection** — enabled for all customer-facing agents; threshold set
- [ ] **Audit trail** — enabled, retention period set (default 30 days, max 1 year)
- [ ] **Zero data retention** — verified with LLM provider
- [ ] **Grounding rules** — only approved data sources accessible per agent
- [ ] **Human escalation** — every customer-facing agent has a defined escalation path

### Data Masking

| Mask Type | Example | Masked As |
|---|---|---|
| Credit card | 4111 1111 1111 1234 | [CREDIT_CARD] |
| SSN | 123-45-6789 | [SSN] |
| Email | user@example.com | [EMAIL] |
| Phone | +1-555-867-5309 | [PHONE] |
| Custom regex | Internal employee ID | [CUSTOM_PII] |

Custom masking rules: Setup > Einstein > Trust Layer > Data Masking > New Masking Rule.

### Toxicity Scoring

Scores both input and output on 0–1. Above threshold → block and substitute a safe response. Categories: hate speech, harassment, self-harm, sexual content, violence.

### Audit Trail

Logs every interaction: timestamp, channel, session ID, user identity, topic selected, actions invoked, prompt sent (after masking), response, Trust Layer scores, escalation events.

Query via `GenAiAuditEvent` or Setup > Einstein > Trust Layer > Audit Trail.

### Grounding Security

```
Access hierarchy (running user permissions apply):
  Public Knowledge — no auth required
  → CRM Records — sharing rules + FLS of the running user
  → Data Cloud Profiles — Data Cloud permission set required
  → External APIs — Named Credential scope required
```

**Rules:**
- Agents run as the assigned service user — that user's sharing + FLS is the ceiling
- Guest-facing agents: scope to Knowledge articles and explicit allow-listed fields only
- Service agents: run as a dedicated service user, not the end customer

---

## Models API and BYOM

### Standard Platform Models

| Model | Best For |
|---|---|
| Einstein (default) | General agent reasoning, action selection — no additional cost |
| GPT-4o (Azure OpenAI) | Complex reasoning, 128K context — requires Azure resource |
| Claude (AWS Bedrock) | Nuanced language, 200K context — requires Bedrock integration |

### BYOM Configuration

1. Setup > Einstein > Models > External Model Configuration
2. Select provider (Azure OpenAI, AWS Bedrock, Vertex AI, custom endpoint)
3. Provide endpoint URL and Named Credential for auth
4. Map the model's input/output schema to the platform's expected format
5. Run the model compatibility test
6. Assign the model to a specific agent or topic in Agent Builder

**Rule:** The Trust Layer still masks PII before any prompt is sent to an external model. Zero-retention must be configured with the external provider independently.

---

## Agentforce for Service

### Einstein Bot Migration

```
Migration path:
  Existing Einstein Bot (Dialog Flow)
    ↓ Bot-to-Agent Migration Tool (Setup > Einstein Bots > Migrate)
    ↓ Dialogs → Topics (auto-mapped, review required)
    ↓ Dialog variables → Action inputs/outputs
    ↓ SFDC Apex bot actions → Invocable Apex actions
  Resulting: Agentforce Service Agent
```

Post-migration checklist:
- [ ] Review auto-mapped topics for accuracy
- [ ] Replace hard-coded dialog logic with topic instructions
- [ ] Test every former dialog path as a conversation scenario in Testing Center
- [ ] Validate omnichannel handoff routing rules

### Case Deflection Pattern

```
1. Customer describes issue → agent identifies topic
2. Topic: Troubleshooting
   Action: Search_Knowledge → present top 3 articles; ask if resolved
3. If resolved → close interaction, log deflection metric
4. If not resolved:
   Action: Check_Open_Cases
   → Existing case: update with new context, offer status
   → No case: Create_Case action, confirm with customer
5. Escalation: transfer to human with transcript + case record
```

---

## Agentforce for Sales

### SDR Agent Topics

```
Topics: Prospect Outreach, Lead Qualification, Meeting Scheduling, Follow-Up Cadence
Data sources: Lead/Contact/Account records, Activity history, Einstein Activity Capture,
              Data Cloud unified profile (if available)
```

### Coach Agent

```
Actions: Analyze_Call_Recording, Generate_Coaching_Brief, Update_Next_Steps, Schedule_Coaching_Session
Instructions:
  - "Focus feedback on MEDDIC gaps: Metrics, Economic Buyer, Decision Criteria, Decision Process,
     Identify Pain, Champion"
  - "Reference specific moments in the call transcript when possible"
  - "Do not share coaching notes with the rep before manager review"
```

### Pipeline Inspection Apex Action

```apex
public class PipelineActions {
    @InvocableMethod(
        label='Get At-Risk Opportunities'
        description='Returns opportunities closing in 30 days with no activity in 14+ days'
        category='Pipeline Management'
    )
    public static List<PipelineResult> getAtRiskOpportunities(List<PipelineRequest> requests) {
        List<PipelineResult> results = new List<PipelineResult>();
        for (PipelineRequest req : requests) {
            List<Opportunity> atRisk = [
                SELECT Id, Name, Amount, CloseDate, ForecastCategoryName,
                       LastActivityDate, StageName, Owner.Name
                FROM Opportunity
                WHERE OwnerId = :req.ownerId
                  AND CloseDate <= :Date.today().addDays(30)
                  AND ForecastCategoryName IN ('Commit', 'Best Case')
                  AND LastActivityDate <= :Date.today().addDays(-14)
                WITH USER_MODE
                ORDER BY Amount DESC
                LIMIT 10
            ];
            PipelineResult result    = new PipelineResult();
            result.opportunityCount  = atRisk.size();
            result.opportunitySummary = buildSummary(atRisk);
            results.add(result);
        }
        return results;
    }

    private static String buildSummary(List<Opportunity> opps) {
        List<String> lines = new List<String>();
        for (Opportunity o : opps) {
            lines.add(o.Name + ' — $' + o.Amount
                + ' — Close: ' + o.CloseDate.format()
                + ' — Last activity: ' + (o.LastActivityDate != null
                    ? o.LastActivityDate.format() : 'Never'));
        }
        return String.join(lines, '\n');
    }

    public class PipelineRequest {
        @InvocableVariable(required=true label='Owner ID' description='Sales rep User ID')
        public Id ownerId;
    }

    public class PipelineResult {
        @InvocableVariable(label='At-Risk Count')
        public Integer opportunityCount;
        @InvocableVariable(label='Opportunity Summary' description='Human-readable list of at-risk deals')
        public String opportunitySummary;
    }
}
```

---

## Agentforce Testing Center

### Test Scenario Categories

| Category | What to Test |
|---|---|
| Happy path | Standard request, all data present, expected action and response |
| Edge case | Ambiguous input, missing data, partial matches |
| Guardrail adherence | Attempt to violate a guardrail — expect refusal |
| Escalation trigger | Scenario that should route to a human |
| Multi-turn context | Conversation requiring memory across 3+ turns |
| Action selection accuracy | Multiple valid topics — does the agent pick the right one? |
| Grounding accuracy | Response reflects actual record data, not a hallucination |

### CLI Commands

```bash
# Generate test suite from agent configuration
sf agent generate test --agent-name "Service_Agent" -o <org-alias> --output-dir ./agentforce-tests/

# Run full test suite
sf agent test run -o <org-alias> --result-format human

# Run a specific test set
sf agent test run -o <org-alias> --test-set-name "Order_Management_Tests"

# CI-friendly synchronous run
sf agent test run -o <org-alias> --synchronous --result-format json > test-results.json
```

### Acceptance Criteria Format

```yaml
TestCase: Order_Return_Happy_Path
  Input: "I want to return order 12345"
  Expected:
    - topic_selected: Order Management
    - action_invoked: Process_Return
    - response_contains: confirmation number
    - response_does_not_contain: another customer
  Acceptance:
    - topic_classification_accuracy: 100%
    - action_selection_accuracy: 100%
    - no_hallucinated_data: true
    - grounding_verified: response references actual order record
```

### Evaluation Metrics

| Metric | Target |
|---|---|
| Topic classification accuracy | > 95% |
| Action selection accuracy | > 90% |
| Grounding accuracy | > 90% (manual review) |
| Guardrail adherence | 100% |
| Resolution rate (service) | > 70% |
| Escalation rate (service) | < 30% |
| Conversation simulation pass rate | > 85% |

---

## Governance

### Trust Layer Policies

```
Policy: Customer_Facing_Agent_Baseline
  Applies to: Service Agent, SDR Agent
  Rules:
    - Toxicity threshold: 0.7 (block above)
    - PII masking: Email, Phone, SSN, Credit Card, NPI
    - Audit logging: All interactions, 90-day retention
    - Zero retention: Verified with LLM provider
    - Approved grounding: CRM (USER_MODE), Knowledge, Data Cloud Profile
    - Restricted: No DML on financial objects without human approval
```

### Human-in-the-Loop Pattern

For high-stakes actions (refunds > $500, contract changes, PII updates):

```
1. Agent identifies high-stakes request
2. Action: Submit_For_Approval → creates ApprovalRequest, notifies supervisor queue
3. Agent tells customer: "I've submitted your request for supervisor review."
4. Supervisor approves/rejects in Salesforce
5. Approval action triggers customer notification
```

### Audit Log Queries

```soql
-- Interactions with escalations (last 7 days)
SELECT SessionId, AgentName, TopicName, ActionName,
       TrustLayerScore, EscalationReason, CreatedDate
FROM GenAiAuditEvent
WHERE CreatedDate = LAST_N_DAYS:7
  AND EscalationReason != null
ORDER BY CreatedDate DESC
LIMIT 200

-- Toxicity score distribution (last 30 days)
SELECT TrustLayerScore, COUNT(Id)
FROM GenAiAuditEvent
WHERE CreatedDate = LAST_N_DAYS:30
  AND TrustLayerScore > 0.5
GROUP BY TrustLayerScore
```

---

## Metadata and Deployment

### Metadata Types

| Component | Metadata API Name | Path |
|---|---|---|
| Agent definition | `GenAiAgent` | `force-app/main/default/genAiAgents/` |
| Agent topic | `GenAiAgentTopic` | `force-app/main/default/genAiAgentTopics/` |
| Prompt template | `GenAiPromptTemplate` | `force-app/main/default/genAiPromptTemplates/` |
| Trust Layer policy | `TrustLayerPolicy` | `force-app/main/default/trustLayerPolicies/` |
| External model config | `ExternalModelConfig` | `force-app/main/default/externalModelConfigs/` |

### Retrieve and Deploy

```bash
# Retrieve agent and all related metadata
sf project retrieve start \
  --metadata "GenAiAgent:Service_Agent" \
  --metadata "GenAiAgentTopic" \
  --metadata "GenAiPromptTemplate" \
  -o <org-alias>

# Deploy to target org
sf project deploy start \
  --source-dir force-app/main/default/genAiAgents \
  --source-dir force-app/main/default/genAiAgentTopics \
  --source-dir force-app/main/default/genAiPromptTemplates \
  --test-level RunSpecifiedTests --tests AgentActionTest \
  -o <target-org>

# Dry-run validation
sf project deploy validate \
  --source-dir force-app/main/default/genAiAgents \
  -o <target-org>
```

**Deployment notes:**
- Agents deploy in **inactive** state — activate manually after validation in target org
- Trust Layer policies must be deployed before the agent that references them
- Test Apex actions with `sf apex run test` before deploying dependent agents

### Agent Builder vs. Metadata API

| Task | Agent Builder UI | Metadata API |
|---|---|---|
| Initial design / prototyping | Preferred | Verbose XML |
| Conversation testing | Required (Testing Center) | N/A |
| Source control and CI/CD | Export after design | Preferred |
| Multi-org promotion | Manual re-config | Retrieve + deploy |

**Recommended workflow:** Design and iterate in Agent Builder in sandbox. Once stable, retrieve metadata and check into source control. Promote via `sf project deploy`.

---

## Output Rules

- Never use `...` as a placeholder in any section — prose, YAML, code blocks, or configuration examples. Write every section out completely.
- Do not add meta-commentary about being an AI or being evaluated.
- Every topic, action, and configuration block must be fully specified, not stubbed.

## Context7 References

Query `/damecek/salesforce-documentation-context` for:
- "Agentforce agent builder topics actions trust layer"
- "Prompt Builder flex template grounding merge fields"
- "Einstein Trust Layer data masking audit trail"
- "sf agent CLI test run generate"

Query `/salesforcecli/cli` for: "sf agent" — current CLI command reference.
