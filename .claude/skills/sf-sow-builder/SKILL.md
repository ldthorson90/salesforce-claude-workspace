---
name: sf-sow-builder
description: Generate a Statement of Work from requirements and estimates. Produces SOW with overview, scope, phases, deliverables, assumptions, change management, RACI, and acceptance criteria.
model: sonnet
user_invocable: true
---

# /sf-sow-builder — Statement of Work Generator

Generate a comprehensive Statement of Work (SOW) document from project requirements and effort estimates.

## Trigger

User invokes `/sf-sow-builder` or asks to create/generate a SOW.

## Inputs

Gather these from the user or prior playbook outputs:

1. **Client Name** — for document header and naming
2. **Project Name** — engagement title
3. **Requirements Document** — path to requirements file (from /sf-discovery or standalone)
4. **Effort Estimate** — path to estimate file (from /sf-estimate or standalone)
5. **Engagement Type** — implementation, optimization, migration, integration
6. **Start Date** — projected engagement start
7. **SOW Template** — (optional) path to custom template

## Execution

### Step 1: Read Inputs

Read the requirements document and effort estimate. Extract:
- Functional requirements grouped by workstream
- Non-functional requirements (performance, security, compliance)
- Effort estimates per workstream with complexity ratings
- Identified risks and assumptions

### Step 2: Generate SOW Sections

Build the following sections:

#### 1. Executive Overview
- Project background and business drivers
- Engagement objectives (3-5 measurable outcomes)
- High-level scope summary

#### 2. Scope of Work
- In-scope items (grouped by workstream)
- Explicitly out-of-scope items
- Assumptions that bound the scope

#### 3. Project Phases & Timeline
- Phase breakdown with milestones (Discovery, Design, Build, Test, Deploy, Hypercare)
- Session-based duration per phase (not calendar dates — those are client-dependent)
- Dependencies between phases

#### 4. Deliverables
- Per-phase deliverable list with acceptance criteria
- Format specification (document, configured org, trained users, etc.)

#### 5. Assumptions & Dependencies
- Technical assumptions (licenses, sandbox availability, data quality)
- Organizational assumptions (client resource availability, decision-making timeline)
- Reference risk patterns from `.claude/consulting-assets/risk-patterns.md`

#### 6. Change Management
- Change request process (how scope changes are handled)
- Impact assessment requirements for changes
- Approval workflow

#### 7. RACI Matrix
- Roles: Consultant, Client PM, Client Admin, Executive Sponsor, End Users
- Activities: Requirements, Design, Build, Test, Deploy, Training, Go-Live

#### 8. Acceptance Criteria
- Per-deliverable acceptance criteria
- UAT process and timeline
- Sign-off requirements

#### 9. Commercial Terms (Template Only)
- Placeholder for pricing model (T&M vs fixed-price)
- Payment milestone structure
- Note: "Populate with actual commercial terms before client delivery"

### Step 3: Output

1. Write `sow_draft.md` to the engagement output directory
2. Generate `.docx` version using the /docx skill
3. Present summary to user with section overview

## Quality Criteria

- All requirements from input document are reflected in scope
- Effort estimates align between estimate file and SOW phases
- No hardcoded dates (session-based timing only)
- Risk patterns from consulting assets are incorporated into assumptions
- RACI matrix covers all deliverables
- Change management section exists and is non-trivial
- Out-of-scope section is explicit (prevents scope creep)

## Output Files

| File | Description |
|------|-------------|
| `sow_draft.md` | Complete SOW in markdown |
| `sow_draft.docx` | SOW as Word document |
