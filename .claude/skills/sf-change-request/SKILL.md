---
name: sf-change-request
description: Document scope changes with impact analysis across data model, automation, and integration. Estimates effort delta and produces formal change request with recommendation.
model: sonnet
user_invocable: true
---

# /sf-change-request — Scope Change Impact Analysis

Document and analyze scope changes with impact assessment and effort estimation.

## Trigger

User invokes `/sf-change-request` or asks to document a scope change / change request.

## Inputs

1. **Change Description** — what the client is requesting
2. **Original Scope Reference** — path to SOW or requirements document
3. **Project Path** — path to SFDX project (for existing implementation analysis)
4. **Org Alias** — (optional) for live org impact analysis

## Execution

### Step 1: Document the Change

Create a structured change request with:
- **Change ID**: Auto-generated (CR-YYYY-MM-DD-NNN)
- **Requested By**: Client contact name
- **Date Requested**: Current date
- **Description**: Detailed description of the change
- **Business Justification**: Why this change is needed
- **Priority**: Critical / High / Medium / Low

### Step 2: Impact Analysis

Analyze impact across all dimensions:

#### Data Model Impact
- New objects or fields required
- Changes to existing objects/fields
- Relationship changes
- Data migration implications

#### Automation Impact
- New triggers, flows, or automation needed
- Changes to existing automation
- Potential conflicts with current automation map
- Governor limit implications

#### Integration Impact
- New or modified integration points
- API changes or new callouts
- Data flow changes
- External system notifications needed

#### Security Impact
- Sharing model changes
- New permission sets or profiles
- FLS changes
- Record access implications

#### UI Impact
- Page layout changes
- New or modified LWC components
- Experience Cloud changes (if applicable)

### Step 3: Effort Estimation

Estimate additional effort using session-based units:
- Per-dimension effort (data model, automation, integration, UI, testing)
- Total effort delta
- Complexity rating (Low / Medium / High / Very High)
- Risk multiplier (based on risk patterns from `.claude/consulting-assets/risk-patterns.md`)

### Step 4: Recommendation

Provide one of:
- **Approve**: Change is low-risk and within engagement capacity
- **Approve with conditions**: Change requires timeline/budget adjustment
- **Defer**: Change should be moved to a future phase
- **Reject**: Change conflicts with architecture or introduces unacceptable risk

Include rationale for the recommendation.

### Step 5: Output

Write `change_request_<id>.md` with all sections.

## Quality Criteria

- All five impact dimensions are assessed (even if "no impact")
- Effort estimate is broken down, not a single number
- Recommendation includes clear rationale
- Original scope reference is cited for comparison
- Risk patterns are checked against change
- Change is traceable (has unique ID, date, requestor)

## Output Files

| File | Description |
|------|-------------|
| `change_request_<id>.md` | Complete change request document |
