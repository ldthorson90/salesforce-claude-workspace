---
name: sf-data-cloud
description: Configure Salesforce Data Cloud — data streams, data model objects, identity resolution, segmentation, activation, and calculated insights. Aligned to Data Cloud Consultant certification.
user-invocable: true
---

# Salesforce Data Cloud

Configure and manage Data Cloud for unified customer profiles.

## Arguments

- `design <use-case>` — design a Data Cloud implementation
- `connect <source>` — configure a data stream/connector
- `model <object>` — create or map a Data Model Object (DMO)
- `identity <ruleset>` — configure identity resolution
- `segment <name>` — create a segment
- `activate <name>` — configure an activation target
- `calculated-insight <name>` — create a calculated insight

## Data Cloud Architecture

```
Data Sources → Ingest → Model → Harmonize → Unify → Activate
     │            │        │         │          │         │
  CRM, APIs,   Data     DMOs    Identity    Unified   Segments,
  Files, MKT  Streams  (schema) Resolution  Profile   Activations,
  Cloud, etc.          mapping   rules                 Insights
```

### Implementation Phases

| Phase | What | Key Decisions |
|---|---|---|
| **1. Connect** | Ingest data from sources | Which connectors, refresh frequency, volume |
| **2. Prepare** | Map source fields to DMOs | Standard vs custom DMOs, field mapping |
| **3. Harmonize** | Link records across sources | Identity resolution rules, match criteria |
| **4. Segment** | Define audiences | Segment criteria, refresh schedule |
| **5. Activate** | Push segments to targets | Marketing Cloud, Ads, CRM, custom |
| **6. Analyze** | Calculated insights, dashboards | Metrics, aggregations, trends |

## Phase 1: Connect (Data Streams)

### Connector Types

| Source | Connector | Frequency |
|---|---|---|
| Salesforce CRM | CRM Connector (native) | Near real-time (CDC-based) |
| Marketing Cloud | MC Connector | Scheduled |
| Amazon S3 / GCS | Cloud Storage | Scheduled |
| External APIs | Ingestion API | Real-time push |
| Files (CSV) | File Upload | On-demand |
| MuleSoft | MuleSoft Connector | Configurable |

### CRM Connector Setup
- Select objects to sync (Account, Contact, Lead, Opportunity, Case, custom)
- Maps automatically to standard DMOs where possible
- Uses Change Data Capture for near real-time sync
- **Rule:** Only sync objects you'll actually use in segmentation/activation

### Ingestion API (for custom sources)
```
POST /api/v1/ingest/sources/{sourceApiName}/{objectName}
Content-Type: application/json
Authorization: Bearer {access_token}

[
  {"email": "john@example.com", "first_name": "John", "purchase_amount": 150.00},
  {"email": "jane@example.com", "first_name": "Jane", "purchase_amount": 75.50}
]
```

## Phase 2: Prepare (Data Model Objects)

### Standard DMOs (use when possible)

| DMO Category | Examples |
|---|---|
| Individual | Individual, Contact Point Email, Contact Point Phone |
| Account | Account |
| Sales | Sales Order, Sales Order Product |
| Service | Case, Service Channel |
| Engagement | Campaign, Campaign Member |
| Product | Product, Price Book |
| Loyalty | Loyalty Program, Loyalty Member |

### Custom DMOs
Create when no standard DMO fits:
- Name: `<Category>_<Purpose>` (e.g., `Engagement_WebVisit`)
- Always include: primary key field, timestamp, source identifier
- Map relationships to Individual or Account DMO

### Field Mapping
```
Source: CRM Account
  Source Field          → DMO Field              → Type
  Account.Name         → Account.Name            → Text
  Account.Industry     → Account.Industry__c     → Picklist
  Account.AnnualRevenue → Account.Revenue__c     → Currency
  Account.CreatedDate  → Account.Created_Date__c → DateTime
```

**Rules:**
- Map to standard DMO fields first, custom only when no standard match
- Preserve data types (don't map Currency to Text)
- Include a source system identifier for lineage tracking

## Phase 3: Harmonize (Identity Resolution)

### Match Rules
```
Ruleset: Customer_Identity
  Rule 1 (Exact): Email address exact match (highest confidence)
  Rule 2 (Exact): Phone + Last Name exact match
  Rule 3 (Fuzzy): First Name fuzzy + Last Name exact + City exact
  Rule 4 (Normalized): Email normalized (lowercase, remove dots in Gmail)
```

### Reconciliation Rules
When multiple source records merge into one unified profile:
- **Most Recent** — use the newest value (good for addresses, phone)
- **Source Priority** — CRM wins over marketing data
- **Most Frequent** — use the most common value across sources

### Identity Resolution Best Practices
- Start with high-confidence exact matches, add fuzzy later
- Always include an email-based rule (highest match rate)
- Test match results before deploying (review merge candidates)
- Monitor merge rate — too high = overly aggressive matching
- Set up reconciliation rules for EVERY mapped field

## Phase 4: Segment

### Segment Types
- **Batch Segments** — refresh on schedule (hourly, daily). Use for most marketing.
- **Streaming Segments** — evaluate in real-time. Use for triggered journeys.

### Segment Builder
```
Segment: High_Value_Customers
  Include: Individual WHERE
    - Unified Account.Revenue__c > 100000
    - AND has Sales Order in last 90 days
    - AND Contact Point Email is not null
  Exclude:
    - Account.Status = 'Churned'
  Refresh: Daily at 6 AM
```

### Segmentation Rules
- Keep segments simple (3-5 criteria max for maintainability)
- Use calculated insights for complex scoring, then segment on the score
- Always include an email/phone contact point check (otherwise can't activate)
- Name: `<Audience>_<Purpose>` (e.g., `HighValue_RetentionCampaign`)

## Phase 5: Activate

### Activation Targets

| Target | Use Case |
|---|---|
| Marketing Cloud | Email/SMS journeys |
| Google/Meta Ads | Audience matching for ad targeting |
| Salesforce CRM | Write back enriched data to Account/Contact |
| Amazon S3 | Export for data warehouse/analytics |
| Webhook | Trigger external system actions |

### CRM Activation (Write-Back)
- Enrich CRM records with unified profile data
- Map Data Cloud fields back to CRM fields
- Set conflict resolution (Data Cloud wins vs CRM wins)
- Schedule or trigger on segment entry/exit

## Phase 6: Calculated Insights

```sql
-- Example: Customer Lifetime Value
SELECT
  Individual__c.Id__c AS IndividualId,
  SUM(SalesOrder__c.TotalAmount__c) AS LifetimeValue,
  COUNT(SalesOrder__c.Id__c) AS OrderCount,
  MAX(SalesOrder__c.OrderDate__c) AS LastOrderDate,
  DATEDIFF(day, MAX(SalesOrder__c.OrderDate__c), CURRENT_DATE) AS DaysSinceLastOrder
FROM Individual__c
JOIN SalesOrder__c ON SalesOrder__c.Individual__c = Individual__c.Id__c
GROUP BY Individual__c.Id__c
```

Use calculated insights for:
- Customer Lifetime Value (CLV)
- Engagement scoring
- Churn risk
- Product affinity
- Recency/Frequency/Monetary (RFM) analysis

## Metadata and CLI

```bash
# Data Cloud CLI (community plugin)
sf data360 connect list -o <org-alias>
sf data360 model describe -o <org-alias>
```

## Context7 References

- `/damecek/salesforce-documentation-context` — "Data Cloud identity resolution segmentation activation"
