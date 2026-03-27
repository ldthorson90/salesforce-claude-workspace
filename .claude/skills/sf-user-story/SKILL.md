---
name: sf-user-story
description: Convert unstructured conversation notes and meeting transcripts into formal Salesforce user stories with acceptance criteria, implementation notes, and complexity sizing.
user-invocable: true
---

# Salesforce User Story Generator

Convert unstructured notes, meeting transcripts, and verbal descriptions into formal user stories with acceptance criteria, Salesforce implementation mapping, and complexity sizing.

## Arguments

- `<input-file>` — path to notes/transcript file
- `inline` — paste or type notes directly
- `batch` — process multiple stories from one input
- `refine <story-file>` — improve existing stories with more detail

## Workflow

### Step 1: Input Processing

- Accept unstructured text: meeting notes, email threads, chat transcripts, verbal descriptions
- Extract distinct user stories (one input may contain many stories)
- If `refine` mode: read the existing story file and identify gaps in acceptance criteria, implementation notes, or sizing
- If input is ambiguous or missing critical context, ask clarifying questions:
  - Who are the users affected?
  - What business process does this change?
  - What triggers this need (pain point, new requirement, compliance)?

### Step 2: Persona Extraction

Identify who needs what. Map to standard Salesforce personas:

| Persona Category | Specific Roles |
|-----------------|----------------|
| Sales | Sales Rep, Sales Manager, Sales VP, BDR/SDR |
| Service | Service Agent, Service Manager, Field Technician |
| Admin/Ops | System Administrator, Business Analyst, RevOps |
| Portal | Partner, Customer (community/portal users) |
| System | Integration (system-to-system), Scheduled Process |
| Leadership | Executive, Director (reporting/visibility consumers) |

If the persona is unclear from the input, infer from context or ask. Never default to a generic "User" persona — Salesforce implementations always have identifiable role-based personas.

### Step 3: Story Formulation

For each identified story, generate:

```
As a [Persona]
I want [Action/Capability]
So that [Business Outcome/Value]
```

Rules:
- Action must be specific and testable (not "manage data" but "filter the opportunity list by close date range")
- Outcome must be business value, not technical implementation (not "so that the trigger fires" but "so that I can focus on deals closing this quarter")
- One story per capability — split compound stories ("I want X and Y" becomes two stories)
- Flag anything that reads as a technical task rather than user-facing value and suggest reframing

### Step 4: Acceptance Criteria

Generate Given/When/Then acceptance criteria for each story:

```
Given [precondition]
When [action]
Then [expected result]
```

Requirements:
- Minimum 3 acceptance criteria per story
- Must include: one positive case (happy path), one negative case (error/invalid input), and one edge case (boundary condition, empty state, or permission variation)
- Each criterion must be independently testable
- Use concrete values where possible (not "a large number" but "200+ records")

### Step 5: Salesforce Implementation Notes

For each story, map to Salesforce platform capabilities:

- **Objects affected**: which standard/custom objects are read, written, or created
- **Automation needed**: Flow (record-triggered, screen, scheduled), Apex trigger/handler, batch job, platform event
- **UI changes**: page layout modification, Lightning Record Page, new LWC component, list view, report
- **Security**: new permission set, sharing rule, field-level security change, record type access
- **Dependencies**: other stories this depends on or enables (use story IDs)

These notes are implementation guidance, not prescriptive design. The builder session makes final technical decisions.

### Step 6: Complexity and Sizing

For each story, assess:

- **T-shirt size**: S / M / L / XL (aligned with `/sf-estimate` categories)
- **Category**: Configuration / Declarative Automation / Custom Development / Data Migration / Integration
- **Complexity factors**: data volume, integration touchpoints, custom UI, complex business logic, governor limit concerns
- **Risk flags**: governor limits at scale, data model changes affecting other features, breaking changes to existing automation

### Step 7: Output Generation

Assemble the final user stories document. Apply INVEST validation:
- **I**ndependent: can be delivered without other stories (flag dependencies if not)
- **N**egotiable: describes what, not how
- **V**aluable: delivers business value to the persona
- **E**stimable: sized and categorized
- **S**mall: fits in a single build session (split if XL)
- **T**estable: acceptance criteria are concrete and verifiable

## Output Format

```markdown
# User Stories

## Source: <input description>
## Date: <date>
## Stories Generated: <N>

### Story Summary

| ID | Title | Persona | Size | Category | Dependencies |
|----|-------|---------|------|----------|-------------|

---

### US-001: <Short Title>

**As a** <persona>
**I want** <action>
**So that** <outcome>

#### Acceptance Criteria

1. **Given** <precondition> **When** <action> **Then** <result>
2. **Given** <precondition> **When** <action> **Then** <result>
3. **Given** <precondition> **When** <action> **Then** <result>

#### Salesforce Implementation

- **Objects**: Account, Contact, Custom_Object__c
- **Automation**: Record-Triggered Flow (after insert)
- **UI**: Lightning Record Page modification
- **Security**: New Permission Set "Feature_Access"
- **Dependencies**: US-003

#### Sizing

- **Complexity**: M
- **Category**: Declarative Automation
- **Risk**: Low
- **Notes**: <any sizing rationale or caveats>

---

### US-002: <Short Title>
...
```

If `batch` mode, include the summary table at the top and all stories in a single document.

If `refine` mode, output the improved story with changes annotated (what was added/changed and why).

## Notes

- Stories should be INVEST compliant. Flag any story that violates INVEST and explain which principle is broken.
- Split epic-sized items (XL or larger) into multiple stories. Explain the decomposition rationale.
- Flag stories that are really technical tasks (not user-facing value) and suggest reframing as user-facing stories or reclassifying as technical enablers.
- When processing meeting notes, distinguish between decisions (actionable stories), open questions (need clarification), and background context (informational, not a story).
- Offer to save output to the project directory and/or create GitHub Issues from the stories.
