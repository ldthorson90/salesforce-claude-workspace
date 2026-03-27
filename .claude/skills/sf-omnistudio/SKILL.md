---
name: sf-omnistudio
description: Build with Salesforce OmniStudio — OmniScripts, FlexCards, Integration Procedures, DataRaptors, and decision matrices. Aligned to OmniStudio Consultant certification.
user-invocable: true
---

# Salesforce OmniStudio

Build guided experiences and integrations using OmniStudio components.

## Arguments

- `omniscript <name>` — design an OmniScript (guided process)
- `flexcard <name>` — design a FlexCard (contextual UI)
- `integration-procedure <name>` — design an Integration Procedure
- `dataraptor <name>` — design a DataRaptor (data transform)
- `decision-matrix <name>` — create a Decision Matrix

## Component Overview

```
User Interaction Layer
  └── OmniScript (guided multi-step process)
  └── FlexCard (contextual data display)

Orchestration Layer
  └── Integration Procedure (server-side orchestration)

Data Layer
  └── DataRaptor Extract (read data)
  └── DataRaptor Transform (reshape data)
  └── DataRaptor Load (write data)
  └── DataRaptor Turbo Extract (high-performance read)

Decision Layer
  └── Decision Matrix (lookup table logic)
  └── Expression Set (calculation engine)
```

## OmniScript

### What It Is
A declarative guided process — multi-step forms with branching, validation, data lookups, and integration calls. Think "wizard builder" with enterprise-grade features.

### Design Patterns

**Step Structure:**
```
OmniScript: NewCustomerOnboarding
  Type: Industry
  SubType: CustomerOnboarding
  Language: English

  Step 1: Personal Information
    - Text: FirstName, LastName (required)
    - Email: Email (validated)
    - Phone: Phone
    - Conditional: Show Business fields if Type = 'Business'

  Step 2: Address
    - Address block with auto-complete
    - DataRaptor Extract: verify address via integration

  Step 3: Product Selection
    - Selectable Items from Integration Procedure
    - Pricing calculation via Expression Set

  Step 4: Review & Submit
    - Summary of all inputs
    - Integration Procedure: create records + send confirmation

  Step 5: Confirmation
    - Confirmation number, next steps
```

### Element Types

| Element | Purpose |
|---|---|
| Text Block | Display instructions/information |
| Input Fields | Collect data (text, number, date, picklist) |
| Selectable Items | Radio/checkbox selections from data source |
| Type Ahead | Autocomplete search |
| Edit Block | Edit existing record fields inline |
| DataRaptor Action | Read/write data during the process |
| Integration Procedure Action | Call server-side orchestration |
| Conditional | Show/hide based on prior inputs |
| Step | Group elements into wizard pages |
| Navigate Action | Redirect after completion |

### Best Practices
- Keep steps to 5-7 max (user fatigue)
- Pre-populate known data (don't ask for what you already have)
- Use conditional blocks to hide irrelevant sections
- Validate per-step, not just at submit
- Save progress: enable resume capability for long processes
- Test with screen reader for accessibility

## FlexCard

### What It Is
A contextual UI card that displays data from multiple sources in a compact, configurable format. Embeddable in Lightning pages, OmniScripts, or Experience Cloud.

### Design Pattern
```
FlexCard: CustomerSummary360
  Data Sources:
    - DataRaptor: Get_Account_Details (Account fields)
    - DataRaptor: Get_Recent_Cases (last 5 cases)
    - DataRaptor: Get_Open_Opportunities (active deals)

  Layout:
    ┌─────────────────────────────────┐
    │ Customer Name    │ Tier: Gold   │
    │ Account #: 12345 │ Since: 2019  │
    ├─────────────────────────────────┤
    │ Recent Cases          │ Pipeline│
    │ Case-001: Open (High) │ $150K  │
    │ Case-002: Resolved    │ 3 deals│
    └─────────────────────────────────┘
    [View Account] [Create Case] [New Opportunity]
```

### Configuration
- Data sources: DataRaptor, SOQL, REST, Apex
- Conditional states: show different layouts based on data values
- Child FlexCards: nest cards for related data
- Actions: buttons that launch OmniScripts, Flows, or navigate

## Integration Procedure

### What It Is
Server-side orchestration that chains multiple data operations, transformations, and external calls. Runs in Apex context with governor limits.

### Design Pattern
```
Integration Procedure: CreateCustomerAccount
  1. DataRaptor Extract: Check for existing account (dedup)
  2. Decision Matrix: Determine account tier based on revenue
  3. DataRaptor Load: Create Account record
  4. DataRaptor Load: Create Contact record
  5. HTTP Action: Call external CRM for sync
  6. DataRaptor Load: Create integration log record
  7. Response Action: Return new Account Id
```

### Best Practices
- Use for multi-step server logic (not simple CRUD — use DataRaptor directly)
- Enable caching for frequently called procedures
- Set timeout for HTTP actions
- Error handling: use Try/Catch blocks within the procedure
- Chain Integration Procedures for complex workflows

## DataRaptor

### Types

| Type | Direction | Use Case |
|---|---|---|
| Extract | Read from SF | Query records, feed FlexCards/OmniScripts |
| Transform | Reshape | Map between JSON structures |
| Load | Write to SF | Create/update records |
| Turbo Extract | Read (high perf) | Large dataset reads, bypass some limits |

### Extract Example
```
DataRaptor: Get_Account_Details
  Object: Account
  Filter: Id = {AccountId}
  Fields: Name, Industry, AnnualRevenue, Owner.Name
  Output: AccountName, AccountIndustry, Revenue, OwnerName
```

### Transform Example
```
DataRaptor: Map_External_To_SF
  Input: External API response JSON
  Mapping:
    $.customer.name → AccountName
    $.customer.email → ContactEmail
    $.customer.phone → ContactPhone
    $.orders[0].total → LastOrderAmount
```

## Decision Matrix

### What It Is
A lookup table that returns values based on input criteria. Replaces complex IF/THEN logic.

```
Decision Matrix: Account_Tier_Assignment
  Input: AnnualRevenue (Currency), NumberOfEmployees (Number)

  | Revenue Min | Revenue Max | Employees Min | → Tier    | → SLA_Hours |
  |-------------|-------------|---------------|-----------|-------------|
  | 0           | 99999       | 0             | Bronze    | 48          |
  | 100000      | 999999      | 0             | Silver    | 24          |
  | 1000000     | *           | 0             | Gold      | 8           |
  | 1000000     | *           | 500           | Platinum  | 4           |
```

### Use Cases
- Pricing/discount matrices
- Territory assignment
- SLA determination
- Product eligibility
- Risk scoring
