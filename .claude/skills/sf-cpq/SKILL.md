---
name: sf-cpq
description: Design and implement Salesforce CPQ — product rules, price rules, quote templates, approvals, guided selling, and advanced pricing. Aligned to CPQ Specialist certification.
user-invocable: true
---

# Salesforce CPQ

Configure, Price, Quote on the Salesforce platform.

## Arguments

- `design <requirements>` — design a CPQ solution
- `product-rule <name>` — create a product rule (validation, alert, selection, filter)
- `price-rule <name>` — create a price rule (discount, markup, override)
- `quote-template <name>` — design a quote document template
- `approval <name>` — configure an approval chain
- `guided-selling <name>` — create a guided selling flow

## CPQ Architecture

```
Quote (SBQQ__Quote__c)
  ├── Quote Lines (SBQQ__QuoteLine__c)
  │   ├── Product (Product2)
  │   │   ├── Product Options (SBQQ__ProductOption__c)
  │   │   ├── Product Features (SBQQ__ProductFeature__c)
  │   │   └── Configuration Attributes
  │   ├── Pricing
  │   │   ├── List Price
  │   │   ├── Discount Schedule
  │   │   ├── Price Rules (calculated adjustments)
  │   │   └── Net Price
  │   └── Subscription Terms (if subscription)
  ├── Quote Template → PDF Document
  └── Approval Process
       └── Approved → Contract / Order
```

## Product Configuration

### Product Types

| Type | Use Case | Example |
|---|---|---|
| Standalone | Single product, no options | "Support Plan" |
| Bundle | Parent + required/optional components | "CRM Suite" = Base + Modules |
| Multi-Dimensional | Pricing varies by dimensions (quantity × term) | Volume licensing |

### Product Options (Bundle Components)
```
Bundle: Enterprise CRM Suite
  Feature: Core Modules (required)
    ├── Option: CRM Base (required, included, quantity 1)
    └── Option: User License (required, quantity = # users)
  Feature: Add-On Modules (optional)
    ├── Option: Marketing Module ($500/mo)
    ├── Option: Service Module ($400/mo)
    └── Option: Analytics Module ($300/mo)
  Feature: Implementation
    └── Option: Onboarding Package (one-time, $5000)
```

### Configuration Attributes
Custom fields on the quote line that affect pricing/behavior:
- Number of users → drives license quantity
- Deployment type (cloud/on-prem) → affects base price
- Support tier (standard/premium) → adds to line price

## Product Rules

### Rule Types

| Type | Purpose | When |
|---|---|---|
| Validation | Block invalid configurations | Save/Calculate |
| Alert | Warn but don't block | Save/Calculate |
| Selection | Auto-add/remove products | Add/Remove |
| Filter | Restrict product search | Product Lookup |

### Examples

**Validation Rule:** Prevent incompatible products
```
Rule: Prevent_Legacy_With_Modern
  Type: Validation
  Scope: Quote
  Conditions: Quote contains "Legacy Module" AND "Modern Module"
  Error Message: "Legacy and Modern modules cannot be combined. Choose one."
```

**Selection Rule:** Auto-add required components
```
Rule: Auto_Add_Support
  Type: Selection
  Scope: Product
  Conditions: Bundle = "Enterprise Suite" AND Support not present
  Action: Add "Standard Support" with quantity 1
```

**Filter Rule:** Restrict products by account type
```
Rule: Government_Products_Only
  Type: Filter
  Conditions: Account.Type = 'Government'
  Filter: Product.Available_For_Government__c = true
```

## Price Rules

### Pricing Waterfall
```
List Price
  → Volume Discount (Discount Schedule)
  → Partner Discount (Price Rule)
  → Promotional Discount (Price Rule)
  → Manual Discount (rep-entered, capped by approval)
  = Customer Price
  → Additional Discount (approval required if > threshold)
  = Net Price
```

### Price Rule Example
```
Rule: Volume_Discount_Override
  Target Object: Quote Line
  Target Field: SBQQ__Discount__c
  Conditions:
    - Product Family = 'Licenses'
    - Quantity >= 100
  Price Action:
    - Set Discount = 15%
```

### Discount Schedules
```
Discount Schedule: Volume_Licensing
  Type: Slab (each tier priced independently)
  Tiers:
    1-49 units:    $100/unit (0% discount)
    50-99 units:   $90/unit  (10% discount)
    100-499 units: $80/unit  (20% discount)
    500+ units:    $70/unit  (30% discount)
```

**Slab vs Range:**
- **Slab:** Each tier has its own price (most common for software)
- **Range:** All units priced at the tier where total quantity falls

## Quote Templates

### Document Design
```
Quote Template: Standard_Proposal
  Sections:
    1. Cover Page (company logo, client name, date, quote #)
    2. Executive Summary (custom rich text field)
    3. Solution Overview (grouped by product feature)
    4. Pricing Table
       - Line items grouped by category
       - Columns: Product, Description, Quantity, Unit Price, Discount, Net Price
       - Subtotals per group
       - Grand total
    5. Terms and Conditions (standard T&C template)
    6. Signature Block (electronic signature integration)
```

### Template Configuration
- Use template sections for conditional content (show/hide by product)
- Dynamic line columns: show relevant columns only
- Conditional page breaks between sections
- Multi-language templates for international deals
- PDF generation via CPQ Document Generation

## Approval Process

### Approval Chain Design
```
Approval: Discount_Over_20_Percent
  Entry Criteria: Max Discount on any line > 20%
  Approval Steps:
    1. Sales Manager (if discount <= 30%)
    2. VP Sales (if discount <= 40%)
    3. CRO (if discount > 40%)

Approval: Non_Standard_Terms
  Entry Criteria: Custom_Terms__c = true
  Approval Steps:
    1. Legal Review
```

### Advanced Approvals (CPQ native)
- Multi-tier approval chains
- Smart approvals: skip levels if pre-approved
- Recall and resubmit
- Approval delegation
- Email notifications with quote summary

## Guided Selling

### Flow Design
```
Guided Selling: Find_Right_Product
  Step 1: "What are you looking for?"
    Options: [New Purchase, Add-On, Renewal, Replacement]

  Step 2: "How many users?"
    Input: Number field

  Step 3: "Which capabilities do you need?"
    Multi-select: [CRM, Marketing, Service, Analytics, Commerce]

  Step 4: "What's your deployment preference?"
    Options: [Cloud, Hybrid, On-Premise]

  Result: Auto-add matching bundle with correct options and quantities
```

### Implementation
- Use CPQ Guided Selling or a Screen Flow that populates quote lines
- Map answers to product selection logic
- Pre-populate quantities based on user count
- Set default configuration attributes

## Subscription & Renewal

### Subscription Pricing
- **Subscription Term:** length of contract (12, 24, 36 months)
- **Proration:** partial period billing (daily, monthly)
- **Uplift:** automatic price increase at renewal (e.g., 3% annual)
- **Co-termination:** align new subscriptions to existing contract end date

### Renewal Process
```
Contract End - 90 days → Auto-create Renewal Opportunity
Contract End - 60 days → Auto-create Renewal Quote (copy lines, apply uplift)
Sales rep reviews → Adjusts pricing/products → Sends to customer
Customer accepts → Quote → New Contract
```

## Performance Considerations

- **Calculator plugin:** Custom Apex for complex pricing (use sparingly, affects save time)
- **Quote line limit:** Performance degrades past 200 lines (paginate if needed)
- **Twin fields:** CPQ syncs Quote fields ↔ Opportunity fields (understand the mapping)
- **Package deployment:** CPQ has specific deployment order requirements (deploy CPQ package first)
