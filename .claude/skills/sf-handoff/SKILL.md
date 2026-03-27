---
name: sf-handoff
description: Generate client deliverables for project close — solution design document, admin guide, deployment runbook, data dictionary, and test results summary.
user-invocable: true
---

# Salesforce Client Handoff

Generate professional deliverables for project delivery and client handoff.

## Arguments

- `design-doc` — solution design document
- `admin-guide` — administrator guide for ongoing maintenance
- `deploy-runbook` — step-by-step deployment instructions
- `data-dictionary <org-alias>` — auto-generate from org metadata
- `test-summary <org-alias>` — test results and coverage report
- `full` — generate all deliverables

## Solution Design Document

### Structure
```markdown
# Solution Design Document
## Client: <name>
## Project: <name>
## Version: <X.Y> | Date: <date> | Author: <name>

### 1. Executive Summary
- Business problem
- Proposed solution (1-2 paragraphs)
- Key assumptions and constraints

### 2. Solution Architecture
- Architecture diagram (Mermaid C4 or integration diagram)
- Component inventory (what's being built/configured)
- Integration points

### 3. Data Model
- ERD diagram (Mermaid)
- Object descriptions table (Object | Purpose | Key Fields | Relationships)
- Data volume estimates

### 4. Automation Design
- Automation inventory (Object | Type | Trigger | Purpose)
- Automation sequence diagram for complex workflows
- Decision rationale (why Flow vs Apex vs other — reference /sf-decide output)

### 5. Security Model
- OWD settings
- Role hierarchy diagram
- Permission set matrix
- Sharing rules

### 6. User Interface
- Lightning page designs
- Record type matrix (Record Type | Page Layout | Profile/Permission Set)
- Custom components inventory

### 7. Integration Design (if applicable)
- Integration architecture diagram
- API contract per integration (endpoint, auth, payload, frequency)
- Error handling strategy

### 8. Testing Strategy
- Test types (unit, integration, UAT)
- Test coverage targets
- UAT script summary

### 9. Deployment Plan
- Deployment sequence
- Rollback plan
- Post-deployment verification steps

### 10. Risks and Mitigations
```

### Generation Approach
- Pull data model from `/sf-org-analyze` if org is connected
- Pull architecture decisions from `/sf-decide` history
- Pull automation inventory from org metadata
- Generate diagrams via Mermaid MCP
- Reference the architect decision guides for rationale

## Admin Guide

### Structure
```markdown
# Administrator Guide
## System: <name>

### 1. System Overview
- What was built and why
- Key objects and their relationships (simplified ERD)

### 2. Daily Operations
- Common tasks and how to perform them
- Queue management
- Assignment rule administration

### 3. User Management
- How to add/remove users
- Permission set assignments by role
- Record type assignments

### 4. Automation Reference
- Table of all automation (name, type, what it does, when it runs)
- How to deactivate/reactivate in emergency
- Known dependencies between automations

### 5. Data Management
- Duplicate rules and matching rules
- Data import procedures
- Mass update procedures

### 6. Troubleshooting
- Common error messages and resolutions
- Debug log instructions
- Escalation path

### 7. Change Management
- How to request changes
- Sandbox refresh procedures
- Deployment process overview
```

## Deployment Runbook

### Structure
```markdown
# Deployment Runbook
## Release: <version>
## Target Org: <alias/type>

### Pre-Deployment Checklist
- [ ] All code passes Code Analyzer (0 Critical/High)
- [ ] All tests pass (XX.X% coverage)
- [ ] Deployment validated in staging sandbox
- [ ] Rollback plan reviewed
- [ ] Client approval received

### Deployment Sequence
1. Deploy metadata (objects, fields, permsets) — `sf project deploy start`
2. Deploy Apex classes and triggers
3. Deploy Flows as DRAFT
4. Activate Flows post-verification
5. Assign permission sets to users
6. Verify page layouts and Lightning pages

### Post-Deployment Verification
- [ ] Run smoke tests (list specific scenarios)
- [ ] Verify automation fires correctly
- [ ] Check sharing rules and access
- [ ] Confirm reports/dashboards render

### Rollback Plan
- Deactivate Flows via Setup
- Revert metadata: `sf project deploy start --metadata-dir rollback/`
- Notify affected users
```

## Data Dictionary (auto-generated)

When an org alias is provided, query metadata to generate:

```markdown
# Data Dictionary

## <ObjectName>
| Field | API Name | Type | Required | Description |
|-------|----------|------|----------|-------------|
```

Query: `sf data query -o <org> -q "SELECT QualifiedApiName, Label, DataType, IsRequired, Description FROM FieldDefinition WHERE EntityDefinition.QualifiedApiName = '<Object>'" --json`

## Output Format

All deliverables are generated as Markdown files in the project directory under `docs/`:
```
docs/
├── solution-design.md
├── admin-guide.md
├── deployment-runbook.md
├── data-dictionary.md
└── test-summary.md
```

The `/docx` or `/pdf` global skills can convert these to client-ready formats.
