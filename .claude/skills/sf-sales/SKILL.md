---
name: sf-sales
description: Design and implement Sales Cloud — opportunity management, forecasting, territories, products/price books, lead management, and sales processes. Aligned to Sales Cloud Consultant certification.
user-invocable: true
---

# Salesforce Sales Cloud

Design and implement sales operations on the Salesforce platform.

## Arguments

- `design <requirements>` — design a sales solution
- `opportunity-process` — configure opportunity stages, sales processes, guided selling
- `forecast` — set up collaborative forecasting
- `territory` — design territory management
- `products` — configure products and price books
- `lead-process` — configure lead management and conversion

## Opportunity Management

### Sales Process Design

```
Sales Process: B2B_Enterprise
  Stages:
    Prospecting (10%) → Qualification (20%) → Needs Analysis (40%) →
    Proposal (60%) → Negotiation (80%) → Closed Won (100%) / Closed Lost (0%)

Sales Process: B2B_Renewal
  Stages:
    Renewal Pending (50%) → In Discussion (70%) → Renewed (100%) / Churned (0%)
```

### Configuration
- One Sales Process per business model (new, renewal, upsell, partner)
- Map Sales Process to Opportunity Record Type
- Configure Path component for visual stage tracking
- Guidance for Success: add tips/required fields per stage
- Key Fields per stage (via Dynamic Forms or validation rules):
  - Qualification: Budget confirmed, Decision maker identified
  - Proposal: Products added, Quote generated
  - Negotiation: Discount approved (if > threshold)

### Opportunity Teams
- Define team roles: Account Executive, Solutions Engineer, Executive Sponsor
- Default opportunity teams from Account teams
- Team member access levels: Read Only, Read/Write

### Products & Price Books

```
Standard Price Book
  └── Product: Enterprise License ($10,000/year)
  └── Product: Professional License ($5,000/year)
  └── Product: Implementation Services ($200/hour)

Custom Price Book: EMEA Pricing
  └── Product: Enterprise License (€9,000/year)
  └── Product: Professional License (€4,500/year)
```

**Rules:**
- Every product must have a Standard Price Book entry first
- Custom price books for: regions, partner tiers, promotional pricing
- Use Opportunity Products (OpportunityLineItem) for deal-level pricing
- Schedules: Revenue schedules (recognition) and Quantity schedules (delivery)

## Forecasting

### Collaborative Forecasting Setup
1. Enable Collaborative Forecasting
2. Select forecast types (Opportunity Amount, Quantity, custom measure)
3. Set forecast hierarchy (follows Role Hierarchy)
4. Configure forecast categories:
   - Pipeline, Best Case, Commit, Closed, Omitted
5. Enable forecast adjustments (managers can adjust rep forecasts)

### Forecast Categories Mapping

| Stage | Default Category | Meaning |
|---|---|---|
| Prospecting, Qualification | Pipeline | Early stage, not in forecast |
| Needs Analysis, Proposal | Best Case | Possible but not committed |
| Negotiation | Commit | Rep believes it will close |
| Closed Won | Closed | Revenue recognized |
| Closed Lost | Omitted | Excluded from forecast |

### Forecasting Best Practices
- Weekly forecast submission cadence
- Manager adjustment visibility: private or shared with team
- Quota management: set per user per forecast period
- Historical trend tracking for forecast accuracy measurement

## Territory Management

### Enterprise Territory Management (ETM)

```
Territory Model: FY2026
  └── North America
      ├── US West
      │   ├── California (Assignment Rule: State = 'CA')
      │   └── Pacific Northwest (Assignment Rule: State IN ('WA', 'OR'))
      └── US East
          ├── Northeast (Assignment Rule: State IN ('NY', 'NJ', 'CT', 'MA'))
          └── Southeast (Assignment Rule: State IN ('FL', 'GA', 'NC'))
```

### Configuration
1. Enable Enterprise Territory Management
2. Create Territory Type (Geographic, Named Account, Industry)
3. Build Territory Hierarchy
4. Create Assignment Rules (criteria-based auto-assignment)
5. Assign users to territories
6. Deploy the Territory Model (activate)

### Territory Assignment Rules
- Account-to-Territory: based on account fields (state, industry, revenue)
- Opportunity-to-Territory: inherits from account or custom rules
- Manual overrides: for named/strategic accounts

## Lead Management

### Lead Process
```
Lead Status: New → Contacted → Qualified → Converted / Disqualified

Conversion Mapping:
  Lead.Company → Account.Name
  Lead.Name → Contact.Name
  Lead.Email → Contact.Email
  Lead.Title → Contact.Title
  Lead.Source → Opportunity.LeadSource (if creating Opportunity)
```

### Lead Assignment Rules
```
Rule: Web_Lead_Assignment
  Criteria: LeadSource = 'Web' AND State = 'CA'
  Assign To: Queue > West_Coast_SDRs

Rule: Enterprise_Lead
  Criteria: Company_Size__c > 1000
  Assign To: User > enterprise.ae@company.com
```

### Lead Scoring
Options:
- **Einstein Lead Scoring** — ML-based, automatic (requires sufficient data)
- **Custom scoring** — formula field based on firmographic + behavioral data
- **Marketing Cloud scoring** — if using Account Engagement (Pardot)

```
Score =
  IF(AnnualRevenue > 1000000, 20, IF(AnnualRevenue > 100000, 10, 5)) +
  IF(ISPICKVAL(Industry, 'Technology'), 15, 5) +
  IF(NumberOfEmployees > 500, 15, IF(NumberOfEmployees > 50, 10, 5)) +
  IF(NOT(ISBLANK(Email)), 10, 0) +
  IF(LeadSource = 'Referral', 20, IF(LeadSource = 'Web', 10, 5))
```

## Guided Selling

### Einstein Opportunity Scoring
- Predicts likelihood of close based on historical patterns
- Provides key factors (positive and negative)
- Requires: 200+ closed opportunities, 2+ months of history

### Sales Cadences (High Velocity Sales)
- Define outreach sequences: email → call → email → social → call
- Automated email steps with templates
- Manual steps with reminders
- Branch based on prospect response
