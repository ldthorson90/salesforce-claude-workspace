---
name: sf-estimate
description: Complexity-based scope estimation for Salesforce projects — T-shirt sizing by work category with session budgets, risk multipliers, and scope comparison.
user-invocable: true
---

# Salesforce Scope Estimator

Generate complexity-based scope estimates for Salesforce projects. No time or calendar estimates — complexity and session budgets only.

## Arguments

- `<requirements-file>` — path to requirements document (from `/sf-discovery` or `/sf-gap-analysis`)
- `inline` — estimate from conversation (describe requirements interactively)
- `compare` — generate minimal / recommended / full scope comparison

## Workflow

### Step 1: Load Requirements

- If file provided: parse the requirements document (markdown, YAML, or plain text)
- If `/sf-gap-analysis` output exists in the project directory: use the gap matrix with effort estimates
- If `inline`: ask the user to describe scope via conversation — prompt for: project type, objects involved, integrations, data migration needs, user personas, and key business processes
- Extract discrete work items from whatever input is provided

### Step 2: Categorize Work Items

Classify each requirement or gap into Salesforce work categories:

| Category | Description | Examples |
|----------|-------------|---------|
| Configuration | Point-and-click setup | Page layouts, fields, record types, validation rules, reports, dashboards |
| Declarative Automation | Low-code automation | Flows, approval processes, assignment rules, email alerts, escalation rules |
| Custom Development | Apex/LWC code | Triggers, handlers, batch jobs, LWC components, Visualforce pages |
| Data Migration | Data import/transform | Data mapping, ETL scripts, dedup rules, validation, bulk loading |
| Integration | External system connection | API callouts, middleware config, platform events, CDC, Named Credentials |
| Training & Adoption | Change management | User training materials, admin training, documentation, UAT coordination |

If a work item spans multiple categories, split it. Each item belongs to exactly one category.

### Step 3: T-Shirt Sizing

Size each work item within its category using these calibration anchors:

| Size | Configuration | Declarative | Custom Dev | Data Migration | Integration |
|------|--------------|-------------|------------|----------------|-------------|
| S | 1-2 objects, simple fields | Single flow, simple logic | <100 lines Apex, single class | <1K records, simple mapping | Single endpoint, no transform |
| M | 3-5 objects, picklists, record types | Multi-step flow, subflows | 100-500 lines, handler pattern | 1K-50K records, field mapping | 2-3 endpoints, basic transform |
| L | 5-10 objects, complex relationships | Complex flow with error handling | 500-1500 lines, service layer | 50K-500K records, dedup logic | Multiple systems, middleware |
| XL | 10+ objects, data model redesign | Multiple interacting flows | 1500+ lines, async processing | 500K+ records, complex transform | Real-time, bidirectional sync |

### Step 4: Session Budget Estimation

Map T-shirt sizes to Claude session budgets:

| Size | Estimated Sessions | Token Budget |
|------|-------------------|--------------|
| S | 0.5-1 session | ~50K tokens |
| M | 1-2 sessions | ~100K tokens |
| L | 2-4 sessions | ~200K tokens |
| XL | 4-8 sessions | ~400K tokens |

One session = one focused Claude Code build session working a single concern. Sessions include implementation, testing, and iteration.

### Step 5: Risk Multipliers

Evaluate each applicable risk factor and compound them multiplicatively against the base estimate:

| Risk Factor | Multiplier | When to Apply |
|-------------|-----------|---------------|
| Integration complexity | x1.5 per external system | Any non-Salesforce system involved |
| Data quality | x1.3 | Source data has known quality issues, missing fields, inconsistent formats |
| Scope ambiguity | x1.5 | Requirements are vague, stakeholders disagree, no sign-off yet |
| Technical debt | x1.3 | Building on a legacy org with existing automation, naming conflicts, or undocumented customizations |
| Compliance | x1.3 | Regulated industry (healthcare, finance, government) requiring audit trails or data handling constraints |
| First-time patterns | x1.2 | Using Salesforce features the team has not implemented before |

Apply only factors that are relevant. State which factors were applied and why.

### Step 6: Scope Comparison (if `compare` argument provided)

Generate three scope tiers side by side:

- **Minimal**: Must-have requirements only, simplest viable implementation. Cuts corners on UX polish, reporting, and edge case handling. Ships core functionality.
- **Recommended**: Must-have + should-have requirements, production-ready implementation. Proper error handling, reasonable test coverage, admin documentation.
- **Full**: All requirements including nice-to-haves, optimized implementation with future-proofing. Includes advanced reporting, comprehensive training materials, and extensibility considerations.

For each tier, show what is included and what is explicitly excluded.

### Step 7: Output Generation

Assemble the final estimate document.

## Output Format

```markdown
# Scope Estimate

## Project: <name>
## Date: <date>

### Estimate Summary

| Category | Items | S | M | L | XL | Est. Sessions |
|----------|-------|---|---|---|----|--------------:|
| Configuration | | | | | | |
| Declarative Automation | | | | | | |
| Custom Development | | | | | | |
| Data Migration | | | | | | |
| Integration | | | | | | |
| Training & Adoption | | | | | | |
| **Total** | | | | | | |

### Total Estimate

- **Base estimate**: <N> sessions
- **Risk multipliers applied**: <list each factor and its multiplier>
- **Adjusted estimate**: <N> sessions
- **Confidence**: Low / Medium / High
- **Confidence rationale**: <why this confidence level>

### Work Item Detail

| ID | Requirement | Category | Size | Sessions | Risk Factors | Notes |
|----|-------------|----------|------|----------|--------------|-------|
| E-001 | ... | ... | ... | ... | ... | ... |

### Scope Comparison (if requested)

| Tier | Items | Sessions | What's Included | What's Excluded |
|------|-------|----------|-----------------|-----------------|
| Minimal | | | | |
| Recommended | | | | |
| Full | | | | |

### Assumptions

- <assumption 1>
- <assumption 2>

List every assumption that affects the estimate. If an assumption is wrong, note which items would change.

### Cost Drivers

- **Largest items by session count**: top 3 items consuming the most sessions
- **Highest risk multipliers**: items with the most compounded risk
- **Dependencies that could cascade**: items where a delay or scope change ripples to others

### Recommendations

- **Quick wins**: items to deliver first for early stakeholder value
- **Deferral candidates**: items that could move to a Phase 2 without blocking core value
- **Clarification needed**: items that cannot be reliably estimated without more information
```

## Notes

- Never produce time-based estimates (days, weeks, hours). Sessions and token budgets only.
- If requirements are too vague to estimate, say so and list what information is missing.
- When estimating from `/sf-gap-analysis` output, preserve the gap IDs for traceability.
- Offer to save the estimate to the project directory when complete.
