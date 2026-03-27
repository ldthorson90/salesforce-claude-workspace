---
name: sf-org-analyze
description: Analyze an existing Salesforce org's metadata structure, automation, technical debt, and dependencies. For client intake and handoff documentation.
user-invocable: true
---

# Salesforce Org Analyzer

Understand an existing org's structure for client intake, assessment, or handoff.

## Arguments

- `<org-alias>` — authorized org to analyze
- Optional: `--focus <area>` — narrow analysis: `objects`, `automation`, `apex`, `security`, `all` (default)

## Workflow

### Step 1: Org Overview

```bash
sf org display -o <org-alias> --json
```

Extract: org type, API version, instance URL, org ID.

```bash
sf org list metadata-types -o <org-alias> --json
```

Count metadata by type. Print summary:
- Custom Objects, Custom Fields, Apex Classes, Apex Triggers
- Flows, Lightning Pages, LWC Components, Aura Components
- Permission Sets, Profiles, Connected Apps

### Step 2: Data Model (--focus objects or all)

Query custom objects:
```bash
sf data query -o <org-alias> -q "SELECT QualifiedApiName, Label, Description FROM EntityDefinition WHERE IsCustomizable = true AND IsCustomSetting = false ORDER BY QualifiedApiName" --json
```

For each custom object, query relationships:
```bash
sf data query -o <org-alias> -q "SELECT QualifiedApiName, DataType, ReferenceTo, RelationshipName FROM FieldDefinition WHERE EntityDefinition.QualifiedApiName = '<Object>' AND DataType IN ('Lookup', 'MasterDetail')" --json
```

Output:
- Object inventory table (Object | Label | Fields | Relationships)
- Relationship map (Parent -> Child via field)
- If Mermaid MCP is available, generate an ERD:
  ```mermaid
  erDiagram
    Account ||--o{ Contact : "has"
    Account ||--o{ Opportunity : "has"
  ```

### Step 3: Automation Inventory (--focus automation or all)

Query active flows:
```bash
sf data query -o <org-alias> -q "SELECT Definition.DeveloperName, ProcessType, TriggerType, TriggerObjectOrEvent.QualifiedApiName FROM FlowVersionView WHERE Status = 'Active'" --json
```

Query triggers (via Tooling API or metadata retrieve):
```bash
sf data query -o <org-alias> -q "SELECT Name, TableEnumOrId, Status FROM ApexTrigger WHERE NamespacePrefix = null" --json --use-tooling-api
```

Output:
- Automation map table (Object | Type | Name | Trigger Context | Status)
- Flag objects with multiple automation layers (trigger + flow = conflict risk)
- Flag deprecated automation types (Workflow Rules, Process Builders)

### Step 4: Apex Health (--focus apex or all)

```bash
sf data query -o <org-alias> -q "SELECT Name, ApiVersion, LengthWithoutComments, Status FROM ApexClass WHERE NamespacePrefix = null ORDER BY Name" --json
```

Coverage data:
```bash
sf data query -o <org-alias> -q "SELECT ApexClassOrTrigger.Name, NumLinesCovered, NumLinesUncovered FROM ApexCodeCoverageAggregate WHERE ApexClassOrTrigger.NamespacePrefix = null" --json
```

Analysis:
- Flag classes with API version < 50.0 (stale, may use deprecated APIs)
- Flag classes > 500 lines (complexity risk)
- Flag classes/triggers with 0% coverage
- Calculate overall org coverage percentage
- If source is retrievable, run `sf scanner run` for anti-pattern detection

### Step 5: Security Posture (--focus security or all)

```bash
sf data query -o <org-alias> -q "SELECT Name, (SELECT Id FROM ObjectPermissions WHERE PermissionsModifyAllRecords = true) FROM PermissionSet WHERE IsOwnedByProfile = false" --json
```

- Count permission sets with elevated access
- Flag any "Modify All" or "View All" on sensitive objects
- Count profiles vs permission sets (heavy profile usage = tech debt)

### Step 6: Generate Report

Output a structured Markdown report:

```markdown
# Org Analysis: <org-alias>

## Overview
| Metric | Count |
|--------|-------|
| Custom Objects | XX |
| Apex Classes | XX |
| Apex Triggers | XX |
| Active Flows | XX |
| LWC Components | XX |
| Permission Sets | XX |

## Data Model
<object inventory table>
<Mermaid ERD diagram>

## Automation Map
<automation table by object>
<conflict warnings>

## Apex Health
- API Version Distribution: vXX (N classes), vYY (N classes)
- Coverage: XX.X% overall
- Large classes (>500 lines): <list>
- Zero coverage: <list>

## Security Findings
- Permission Sets with elevated access: <list>
- Profile-heavy configuration: yes/no

## Technical Debt Summary
1. <prioritized finding>
2. <prioritized finding>
3. <prioritized finding>

## Recommendations
- <actionable recommendation>
```

## Notes

- This skill is READ-ONLY. It never modifies the org.
- Safe to run against production orgs (queries only).
- Large orgs: use SOQL LIMIT clauses and process in batches.
- The Mermaid MCP should be used for ERD generation when available.
