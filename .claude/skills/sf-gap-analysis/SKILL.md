---
name: sf-gap-analysis
description: Analyze gaps between current Salesforce org state and desired requirements — classifies each requirement as met, partially met, or gap with implementation type.
user-invocable: true
---

# Salesforce Gap Analysis

Compare current org capabilities against requirements to identify gaps, classify implementation types, and estimate effort. Works with or without a live org connection.

## Arguments

- `<org-alias>` — Salesforce org to analyze (optional — works without org access using manual input)
- `<requirements-file>` — path to a requirements document, typically output from `/sf-discovery` (optional — can ask inline)

## Workflow

### Step 1: Load Requirements

Attempt to load requirements in this order:

1. **If `<requirements-file>` is provided:** parse the file. Expect markdown with a requirements table matching the `/sf-discovery` output format (columns: ID, Requirement, Priority, Acceptance Criteria, SF Feature, Complexity).
2. **If no file but `/sf-discovery` was run recently:** search the project directory for `discovery-report-*.md` files. If found, offer to load the most recent one.
3. **If neither:** ask the user to describe requirements via conversational input. Capture each as:
   - ID (assign REQ-001, REQ-002, ...)
   - Description
   - Priority (Must Have / Should Have / Could Have / Won't Have)

Confirm the loaded requirements with the user before proceeding. Display a count and summary.

### Step 2: Assess Current State

**If `<org-alias>` is provided:**

Run `/sf-org-analyze <org-alias>` or use MCP tools directly to gather:

| Metadata Category | What to Check |
|-------------------|---------------|
| Custom Objects | Object names, field counts, record types |
| Custom Fields | Fields on standard and custom objects |
| Automations | Flows (record-triggered, screen, scheduled), Apex triggers, Process Builders |
| Validation Rules | Active validation rules per object |
| Permission Sets | Permission sets, permission set groups, profiles |
| Installed Packages | AppExchange packages and managed packages |
| Integrations | Connected Apps, Named Credentials, External Services |
| Reports & Dashboards | Report types, dashboard count, scheduled reports |
| Lightning Pages | FlexiPages, Dynamic Forms usage |
| Approval Processes | Active approval processes |

Store this inventory for comparison in Step 3.

**If no org alias:**

Ask the user to describe current capabilities. Walk through each requirement and ask:
- Does the org already handle this today?
- If partially, what works and what doesn't?
- If not at all, is there a workaround in place?

### Step 3: Gap Classification

For each requirement, compare against the current state and classify:

| Status | Definition | Action |
|--------|-----------|--------|
| **Already Met** | Org fully supports this requirement today | Document where (object, flow, field, etc.) |
| **Partially Met** | Capability exists but needs enhancement | Document what's missing and what exists |
| **Gap: Configuration** | Achievable with clicks — page layouts, record types, list views, validation rules, formula fields | Estimate: typically S or M |
| **Gap: Declarative** | Needs Flows, approval processes, duplicate rules, assignment rules, or other low-code tools | Estimate: typically M |
| **Gap: Custom Development** | Requires Apex, LWC, or Visualforce | Estimate: typically L or XL |
| **Gap: Integration** | Needs external system connection — API, middleware, platform events | Estimate: typically L or XL |
| **Gap: Data Migration** | Needs data import, transformation, deduplication, or enrichment | Estimate: varies by volume |
| **Gap: Third-Party** | Needs AppExchange package or external tool | Note: adds license cost and dependency |

When classifying, apply consulting governance rules:
- Challenge the user if a requirement is being over-engineered (e.g., Apex for something a Flow handles)
- Reference `/sf-decide` for architecture decisions on borderline cases
- Flag governor limit risks for high-volume requirements
- Note when a "Gap: Configuration" could become "Gap: Declarative" at scale

### Step 4: Effort Estimation

T-shirt size each gap:

| Size | Definition | Typical Duration |
|------|-----------|------------------|
| **S** | Single configuration change, no dependencies | Hours |
| **M** | Multiple related changes, minor testing | Days |
| **L** | Complex feature, cross-object, integration, or significant testing | 1-2 weeks |
| **XL** | Major feature, multiple integrations, data migration, or architectural change | 2+ weeks |

For each gap, also capture:
- **Dependencies** — does this gap depend on another gap being resolved first?
- **Risk** — Low / Medium / High / Critical
- **Risk factors** — governor limits, data volume, external system dependency, complexity, skill gap

Flag any gaps with circular dependencies or dependency chains longer than 3 items.

### Step 5: Output Generation

Generate the gap analysis report. Ask where to save (same options as `/sf-discovery`: project directory, Obsidian vault, or clipboard).

If saving to the project directory, name the file `gap-analysis-<date>.md`.

## Output Format

```markdown
# Gap Analysis Report

## Client: <name>
## Date: <YYYY-MM-DD>
## Org: <alias or "Manual Assessment">
## Requirements Source: <file path or "Inline">

---

### Executive Summary

- **Requirements Analyzed:** <N>
- **Already Met:** <N> (<percent>%)
- **Partially Met:** <N> (<percent>%)
- **Gaps Identified:** <N> (<percent>%)

**Gap Breakdown by Type:**
| Gap Type | Count | Avg Effort |
|----------|-------|------------|
| Configuration | <N> | <S/M/L/XL> |
| Declarative | <N> | <S/M/L/XL> |
| Custom Development | <N> | <S/M/L/XL> |
| Integration | <N> | <S/M/L/XL> |
| Data Migration | <N> | <S/M/L/XL> |
| Third-Party | <N> | <S/M/L/XL> |

### Gap Analysis Matrix

| Req ID | Requirement | Priority | Status | Gap Type | Effort | Risk | Dependencies |
|--------|-------------|----------|--------|----------|--------|------|--------------|

### Gap Detail

#### GAP-001: <short name>
- **Requirement:** REQ-XXX — <description>
- **Current State:** <what exists today>
- **Gap:** <what is missing>
- **Recommended Approach:** <how to close the gap>
- **Effort:** S / M / L / XL
- **Dependencies:** <list or "None">
- **Risks:** <specific risks for this gap>

*(Repeat for each gap)*

### Recommendations

#### Quick Wins (Configuration gaps, S/M effort)
- <item> — <why it's a quick win>

#### Phase 1 Priorities (Must Have gaps)
- <item> — <effort> — <dependencies>

#### Phase 2 Candidates (Should Have gaps)
- <item> — <effort> — <dependencies>

#### Deferred Items (Could Have / Won't Have, or XL effort)
- <item> — <reason for deferral>

### Dependency Map

List gaps in implementation order, accounting for dependencies:
1. <GAP-XXX> — no dependencies (start here)
2. <GAP-YYY> — depends on GAP-XXX
3. <GAP-ZZZ> — depends on GAP-YYY

### Next Steps

- [ ] Review gap analysis with stakeholders
- [ ] Confirm priority and phasing
- [ ] Begin solution design for Phase 1 gaps
- [ ] Run `/sf-decide` for architecture decisions on complex gaps
```

## Important

- This skill analyzes and recommends. It does not generate code or metadata.
- When an org is connected, rely on actual metadata — do not guess at capabilities.
- When no org is connected, clearly label the assessment as based on user-reported information.
- Challenge over-engineering: if a Flow can do it, do not recommend Apex.
- Reference architect decision guides via `/sf-decide` for any gap that involves an architecture choice.
- Flag gaps that cross governor limit thresholds (50K query rows, 150 DML statements, 6MB heap, 120s timeout).
- If the requirements document is vague, ask clarifying questions before classifying — do not assume.
- Pair naturally with `/sf-discovery` (upstream) and `/sf-handoff` (downstream).
