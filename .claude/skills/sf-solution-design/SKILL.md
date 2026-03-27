---
name: sf-solution-design
description: Generate a complete pre-build solution design document — data model, automation, integration, security, UI, testing, and deployment plan.
user-invocable: true
---

# Solution Design Document Generator

Pre-build design artifact for Salesforce consulting engagements. Assembles outputs from upstream skills (/sf-discovery, /sf-gap-analysis, /sf-data-model, /sf-automation-map) into a unified, client-ready solution design document. Distinct from /sf-handoff, which produces post-build deliverables.

## Arguments

- `generate` — create a new solution design from prior skill outputs in the project directory
- `generate <org-alias>` — create with org metadata enrichment for current-state sections
- `section <section-name>` — generate just one section (e.g., `section data-model`, `section security`, `section integration`)
- `template` — output a blank template with section headings and placeholder prompts for manual completion

## Workflow

### Step 1: Gather Inputs

Scan the project directory for existing outputs from upstream skills:

| Source Skill | Expected Location | Provides |
|---|---|---|
| /sf-discovery | `docs/discovery-*.md` | Business problem, requirements, stakeholders, success criteria |
| /sf-gap-analysis | `docs/gap-analysis-*.md` | Gap matrix, effort estimates, risk flags |
| /sf-data-model | `docs/data-model-*.md` | ERD, object manifest, field inventory |
| /sf-automation-map | `docs/automation-map-*.md` | Process-to-tool mapping, conflict analysis, execution order |

For each missing input:
- If the prompt contains explicit scope, requirements, constraints, and data volumes inline, treat this as sufficient discovery input and proceed directly — do not ask the user to run prerequisite skills first
- If the prompt lacks enough context to design a section, generate a simplified version inline and flag it as "DRAFT — not validated by dedicated skill"
- Only ask the user for missing input when critical information (client name, core business problem) is completely absent

Collect from the user:
- Client name
- Project name
- Author name (defaults to user if not provided)

If `<org-alias>` was provided, run `sf org display -o <org-alias>` to verify connectivity and confirm it is NOT a production org (per consulting governance). Query org metadata for current-state context: existing custom objects, installed packages, automation inventory.

### Step 2: Executive Summary

Synthesize from discovery output:
- Business problem statement (1-2 sentences)
- Proposed solution approach (1 paragraph)
- Key assumptions and constraints
- Expected business outcomes tied to success criteria

Target audience: executive stakeholders. No jargon. Keep to 1-2 paragraphs.

### Step 3: Solution Architecture Overview

Generate a C4 Context diagram (Mermaid) showing:
- The Salesforce org as the central system
- External systems (integrations, data sources)
- User personas and their interaction points
- Data flows between systems

This provides the 30,000-foot view before diving into sections.

### Step 4: Data Model

If /sf-data-model output exists, pull directly:
- ERD diagram (Mermaid)
- Object inventory table (Object, Label, Type [Standard/Custom], Record Volume Estimate, Owner)
- Key relationship descriptions
- Data volume estimates and implications for governor limits

If no prior output, design the data model inline:
- Identify required objects from requirements
- Map relationships (lookup vs. master-detail)
- Generate a simplified ERD
- Flag as "DRAFT — run /sf-data-model for full validation"

### Step 5: Automation Design

If /sf-automation-map output exists, pull directly:
- Automation inventory table (Process, Tool [Flow/Trigger/etc.], Object, Timing, Priority)
- Execution sequence diagram (Mermaid)
- Conflict analysis (same-object triggers, recursive risks)

Reference architect decision guides for tool selection rationale:
- Record-Triggered Automation decision guide for trigger vs. flow decisions
- Asynchronous Processing decision guide for batch/queueable/future decisions

Include order of execution awareness for any object with multiple automations.

If no prior output, build a simplified automation inventory from requirements and flag as draft.

### Step 6: Integration Architecture (if applicable)

Skip this section if no integrations are identified. Otherwise:

Generate an integration inventory table:

| Integration | External System | Direction | Frequency | Protocol | Auth Method | Error Handling |
|---|---|---|---|---|---|---|
| Example | ERP System | Bidirectional | Real-time | REST | Named Credential + OAuth 2.0 | Retry queue + email alert |

For each integration:
- Sequence diagram (Mermaid) showing request/response flow
- Error handling and retry strategy
- Governor limit considerations (callout limits, async vs. sync)
- Named Credential configuration

Reference:
- Data Integration decision guide for pattern selection
- Event-Driven Architecture decision guide if using Platform Events

### Step 7: Security Model

Design the security model:

- **OWD Settings**: Table of Object -> Default Internal Access -> Default External Access with rationale
- **Role Hierarchy**: Mermaid diagram showing role tree
- **Permission Set Matrix**: Profile/Permission Set -> Object CRUD -> Field-level access for sensitive fields
- **Sharing Rules**: Criteria-based and owner-based sharing rules with business justification
- **Record Types**: Per-object record type matrix tied to profiles

Reference /sf-security skill patterns. Flag any `without sharing` classes with documented justification requirement.

### Step 8: User Interface

Design the UI approach:

- **Lightning Page Designs**: Per-object page layout strategy (standard vs. Dynamic Forms)
- **Record Type Matrix**: Object -> Record Type -> Page Layout -> Profile assignment
- **Custom Component Inventory**: Any LWC components needed, with purpose and data source
- **Navigation**: App -> Tab -> Page flow for each user persona

Reference /sf-layout skill patterns. Prefer standard components over custom where feasible.

### Step 9: Data Migration (if applicable)

Skip if greenfield with no data migration. Otherwise:

- Source system inventory
- Field mapping table (Source Field -> Target Object.Field -> Transformation)
- ETL approach (Data Loader, Bulk API, third-party tool)
- Migration sequence (dependencies: parent objects before children)
- Data cleansing rules
- Validation approach (record counts, spot checks, automated comparison)

Reference /sf-migration skill patterns.

### Step 10: Testing Strategy

Define the testing approach per consulting governance standards:

| Test Type | Scope | Tool | Coverage Target |
|---|---|---|---|
| Unit (Apex) | All Apex classes and triggers | `sf apex run test` | >= 85% per class, >= 90% overall |
| Bulk | Trigger handlers with 200+ records | Apex test classes | All triggers pass bulk test |
| Negative | Invalid data, missing permissions | Apex test classes | Key error paths covered |
| Integration | Callouts, platform events | Apex test + mock frameworks | All integration points |
| UAT | Business process validation | Manual test scripts | All acceptance criteria |

Include a UAT script outline:
- Test scenario -> steps -> expected result -> pass/fail for each business requirement

### Step 11: Deployment Plan

Define the deployment sequence respecting metadata dependencies:

1. **Pre-deployment**: Backup, feature flag disable, user communication
2. **Metadata sequence**: Custom objects -> fields -> validation rules -> flows -> Apex -> LWC -> permission sets -> layouts -> profiles
3. **Data operations**: Data migration loads, record type assignments
4. **Post-deployment**: Feature flag enable, cache clear, smoke test
5. **Rollback plan**: Per-step rollback procedure, decision criteria for rollback
6. **Verification**: Post-deployment checklist (functional, data integrity, security)

Reference /sf-deploy skill patterns.

### Step 12: Risks and Mitigations

Pull risk flags from /sf-gap-analysis output if available. Check `.claude/consulting-assets/risk-patterns.md` for common consulting risk patterns.

Generate a risk matrix:

| # | Risk | Probability | Impact | Mitigation | Owner |
|---|---|---|---|---|---|
| R1 | Data volume exceeds governor limits | Medium | High | Implement async processing, batch operations | Technical Lead |

Categorize: Technical, Data, Process, Timeline, Dependency.

### Step 13: Output Generation

Assemble all sections into a single markdown document. Output the complete document and stop. Do not prompt the user to save, generate individual sections, or convert to another format — the document is the deliverable.

## Output Format

```markdown
# Solution Design Document

| Field | Value |
|---|---|
| Client | <client-name> |
| Project | <project-name> |
| Version | 1.0 |
| Date | <YYYY-MM-DD> |
| Author | <author-name> |
| Status | DRAFT |

---

## 1. Executive Summary

<business problem, proposed solution, key assumptions, expected outcomes>

## 2. Solution Architecture

<C4 Context diagram — Mermaid>
<narrative description of architecture approach>

## 3. Data Model

<ERD — Mermaid>
<object inventory table>
<data volume estimates>

## 4. Automation Design

<automation inventory table>
<execution sequence diagram — Mermaid>
<conflict analysis>

## 5. Integration Architecture

<integration inventory table>
<per-integration sequence diagrams — Mermaid>

## 6. Security Model

<OWD table>
<role hierarchy diagram — Mermaid>
<permission set matrix>
<sharing rules>

## 7. User Interface

<page design summary>
<record type matrix>
<custom component inventory>

## 8. Data Migration

<source systems, field mapping, ETL approach, validation>

## 9. Testing Strategy

<test type matrix>
<UAT script outline>

## 10. Deployment Plan

<deployment sequence>
<rollback plan>
<post-deployment verification>

## 11. Risks & Mitigations

<risk matrix table>

## 12. Assumptions & Constraints

<numbered list of assumptions>
<numbered list of constraints>

## 13. Appendix: Decision Log

<ADR entries from /sf-decide usage during design>
| Decision | Options Considered | Selected | Rationale | Guide Referenced |
|---|---|---|---|---|
```

## Section Name Reference

Valid values for `section <section-name>`:

| Name | Maps To |
|---|---|
| `executive-summary` | Section 1 |
| `architecture` | Section 2 |
| `data-model` | Section 3 |
| `automation` | Section 4 |
| `integration` | Section 5 |
| `security` | Section 6 |
| `ui` | Section 7 |
| `migration` | Section 8 |
| `testing` | Section 9 |
| `deployment` | Section 10 |
| `risks` | Section 11 |
