---
name: sf-flow
description: Design and build Salesforce Flows — record-triggered, screen, scheduled, autolaunched, and platform event. Covers subflow design, bulkification, fault handling, and Flow testing.
user-invocable: true
---

# Salesforce Flow Builder

Design and build Flows with proper architecture, bulkification, and error handling.

## Arguments

- `<type> <ObjectName> <description>` — create a flow
  - Types: `record-triggered`, `screen`, `scheduled`, `autolaunched`, `platform-event`, `orchestration`
- `subflow <name>` — create a reusable subflow
- `migrate <process-builder-name>` — migrate a Process Builder to Flow

## Flow Type Selection

| Scenario | Flow Type | Trigger |
|---|---|---|
| Before-save field updates (no DML) | Record-Triggered (Before Save) | Record create/update |
| After-save with DML/callouts | Record-Triggered (After Save) | Record create/update/delete |
| User-facing form/wizard | Screen Flow | Button, Quick Action, Lightning Page |
| Background automation | Autolaunched | Invocable, Process, Apex |
| Time-based follow-ups | Scheduled / Scheduled Paths | Schedule or record change + delay |
| React to platform events | Platform Event-Triggered | Platform event publish |
| Multi-step approval/process | Orchestration | Complex business processes |

### Before Save vs After Save

**Before Save (preferred when possible):**
- 10x faster (no extra DML)
- Can set fields on triggering record directly
- Cannot: create/update OTHER records, send emails, make callouts, post to Chatter

**After Save:**
- Can perform DML on other objects
- Can make callouts (via async path)
- Can send emails, create tasks, post to Chatter
- Runs in its own transaction

## Architecture Rules

### Naming Convention
`<Object>_<Action>_<Context>`
Examples:
- `Account_Update_AfterInsert`
- `Case_Escalation_ScheduledPath`
- `Lead_Conversion_ScreenFlow`

### One Flow Per Object Per Context
Do NOT create multiple record-triggered flows on the same object + trigger context. Consolidate logic into one flow with Decision elements. Multiple flows on the same trigger have **non-deterministic execution order**.

### Subflow Design
- Extract reusable logic into subflows (DRY principle)
- Subflows should have clear input/output variables
- Name: `Sub_<Purpose>` (e.g., `Sub_Send_Notification`, `Sub_Create_Task`)
- Subflows run in the CALLER's transaction (not their own)

### Bulkification

Flows are auto-bulkified by the runtime, BUT you must avoid:
- **Get Records inside loops** — query BEFORE the loop, filter in the loop
- **Create/Update/Delete inside loops** — collect records in a collection variable, DML AFTER the loop
- **Decision elements that query** — pre-load data into collection variables

**Pattern: Collection-based processing**
```
1. Get Records → Collection variable
2. Loop over collection
3. Inside loop: Assignment to build output collection
4. After loop: Create/Update/Delete the output collection (single DML)
```

### Fault Handling

**Every DML and callout element MUST have a fault path:**
- Connect the fault connector to a fault-handling subflow or screen
- Log the fault: `{!$Flow.FaultMessage}` and `{!$Flow.InterviewGuid}`
- Notify admins via email or platform event
- For screen flows: display user-friendly error message
- For record-triggered: consider creating an error log record

### Entry Conditions
- Use entry conditions to filter BEFORE the flow runs (more efficient than Decision elements)
- Use `$Record` and `$Record__Prior` for field change detection
- Formula conditions: `{!$Record.Status__c} != {!$Record__Prior.Status__c}`

## Screen Flow Patterns

### Multi-Step Wizard
1. Use screen elements for each step
2. Navigation: use `Previous` and `Next` with validation between screens
3. Progress indicator: use `<lightning-progress-indicator>` or custom LWC
4. Final screen: summary + confirmation before DML

### Dynamic Choice Sets
- Use `Get Records` to populate picklists dynamically
- Filter choices based on previous screen inputs
- Use `Choice` resources for static options

### Input Validation
- Use `Validate` component on screen fields
- Custom validation: Decision element after screen, loop back if invalid
- Regex validation via formula

## Flow Testing

```bash
# Create flow test
sf flow test create --flow-name <FlowApiName> -o <org-alias>

# Run flow tests
sf flow test run -o <org-alias> --result-format human
```

Test scenarios:
- Single record trigger
- Bulk trigger (200+ records)
- Fault path execution
- Entry condition filtering (records that SHOULD NOT trigger)
- Null/empty field handling

## AI Decision Element (new)

For flows using AI Decision elements:
- NEVER place inside loops (token cost scales linearly)
- Must include merge field references for context
- Place after data collection, before action branching
- Deploy as Draft first, test, then activate

## Metadata Location

- Flow definitions: `force-app/main/default/flows/`
- Flow tests: `force-app/main/default/flowTests/`

## Context7 References

- `/damecek/salesforce-documentation-context` — "Flow bulkification fault handling"
- `/google/flow-lens` — Flow XML to UML diagram conversion
