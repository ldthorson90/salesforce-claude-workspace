---
name: sf-automation-map
description: Map business processes to Salesforce automation tools using architect decision guides — with conflict detection and decision rationale.
user-invocable: true
---

# Salesforce Automation Mapper

Map business processes to the right Salesforce automation tools, detect conflicts across automations on the same object, and document decision rationale grounded in architect decision guides.

## Arguments

- `design` — map processes to automation from requirements
- `audit <org-alias>` — analyze existing automation in an org
- `conflict-check <org-alias>` — detect automation conflicts on an object

## Workflow

### Step 1: Process Inventory

Ask the user to list business processes or import from `/sf-discovery` output.

For each process capture:
- **Trigger event**: record create, update, delete, user action, time-based, external event
- **Objects involved**: which objects are read, written, or related
- **Complexity**: simple field update, multi-object DML, conditional branching, external callout
- **Data volume**: how many records per transaction (1, 10, 200, 10K+)
- **Timing requirement**: synchronous (user waits) vs. asynchronous (background)
- **Error handling**: what happens on failure (retry, notify, rollback)

For `audit` mode:
- Query the org for existing automation using `salesforce_read_apex_trigger` (list all triggers)
- Use `list_metadata` with metadataType `Flow` to list all flows
- Use `salesforce_query_records` on `FlowDefinitionView` for active flow details
- Present the full automation inventory before analysis

### Step 2: Decision Guide Application

For each process, apply the Salesforce Record-Triggered Automation decision framework.

**Fetch guidance:**
- Read `salesforce-architect-decision-guides/01-Record-Triggered-Automation.md` if the local file exists
- Query Context7 for current guidance:
```
mcp__context7__query-docs with libraryId="/damecek/salesforce-documentation-context"
query: "record triggered automation decision guide flow vs apex trigger before after save"
```

**Decision criteria (evaluate in order):**

1. Is it record-triggered or user-initiated?
   - Record-triggered: Flow or Apex Trigger
   - User-initiated: Screen Flow, LWC, or Quick Action

2. Does it need before-save or after-save?
   - Before-save: field updates on the triggering record (most efficient, no DML cost)
   - After-save: creating/updating other records, sending notifications

3. Does it involve DML on other objects?
   - No: Before-Save Flow (fastest, no DML limits consumed)
   - Yes: After-Save Flow or Apex Trigger

4. Does it need external callouts?
   - Yes: Async path required (After-Save Flow with async action, or Queueable Apex)

5. Is the logic complex (deep branching, loops, error recovery)?
   - Simple/moderate: Flow
   - Complex: Apex Trigger + Handler (better testability, version control, debugging)

6. Data volume considerations?
   - Low (< 200 records/txn): Flow or Trigger
   - High (200+ records/txn): Apex with explicit bulkification (Batch for very large sets)

**Tool selection matrix:**

| Tool | When to Use |
|------|-------------|
| Before-Save Flow | Field updates on same record, simple validation, no DML on other objects |
| After-Save Flow | Create/update related records, send email/notifications, simple cross-object logic |
| Record-Triggered Flow (async) | External callouts, heavy processing that can run in background |
| Screen Flow | User-initiated, multi-step forms, guided processes |
| Apex Trigger + Handler | Complex branching, bulkification needs, custom rollback, version-controlled logic |
| Queueable Apex | Chained async processing, callouts, moderate data volume |
| Batch Apex | Large data volume (10K+ records), scheduled processing |
| Platform Event | Decoupled processing, cross-system communication, guaranteed delivery |
| Scheduled Flow / Batch | Time-based periodic processing (nightly sync, weekly cleanup) |
| Change Data Capture | External system sync, audit logging of field changes |

### Step 3: Conflict Detection

For each object that has multiple automations, analyze execution order and conflicts.

**Salesforce execution order (record save):**

1. System validation rules
2. Before-save record-triggered flows
3. Before triggers (Apex)
4. System validation + custom validation rules
5. Duplicate rules
6. After triggers (Apex)
7. Assignment rules
8. Auto-response rules
9. Workflow rules (legacy)
10. After-save record-triggered flows
11. Escalation rules
12. Roll-up summary calculations (+ parent re-evaluation)

Reference: query Context7 for current execution order:
```
mcp__context7__query-docs with libraryId="/damecek/salesforce-documentation-context"
query: "salesforce triggers and order of execution automation flow trigger sequence"
```

**Conflict types to detect:**

| Conflict Type | Description | Risk |
|---------------|-------------|------|
| Re-entry / recursion | Automation A updates record, fires Automation B, which updates record, fires Automation A again | HIGH |
| Field collision | Two automations update the same field with different values | HIGH |
| Governor accumulation | Multiple automations each consuming SOQL/DML, pushing totals toward limits | MEDIUM |
| Order dependency | Automation B depends on a field value set by Automation A, but execution order is wrong | MEDIUM |
| Redundant execution | Two automations doing the same thing (e.g., Flow and Trigger both sending email on same event) | LOW |

**Mitigation strategies:**
- Recursion: static boolean guard in Apex handlers, "Run Once" entry criteria in Flows
- Field collision: consolidate into a single automation
- Governor accumulation: combine DML operations, use collections
- Order dependency: consolidate or use before-save for prerequisite field updates

### Step 4: Async Decision

For processes identified as needing async processing:

**Fetch guidance:**
- Read `salesforce-architect-decision-guides/05-Asynchronous-Processing.md` if available
- Query Context7:
```
mcp__context7__query-docs with libraryId="/damecek/salesforce-documentation-context"
query: "queueable batch schedulable future method comparison async processing salesforce"
```

**Async tool selection:**

| Tool | Max Records | Chaining | Callouts | Scheduling | Use Case |
|------|-------------|----------|----------|------------|----------|
| Future Method | 200/txn | No | Yes (1) | No | Simple one-off callouts |
| Queueable | 200/txn | Yes (depth 5) | Yes | No | Chained processing, complex callouts |
| Batch Apex | 50M | Via finish() | Yes (per execute) | Via Schedulable | Large data volumes, bulk processing |
| Schedulable | N/A | Launches other async | No | Yes (cron) | Periodic jobs |
| Platform Event | Per event limits | N/A | Via subscriber | N/A | Decoupled, event-driven |

### Step 5: Output Generation

Compile the full automation map document.

## Output Format

```markdown
# Automation Map
## Project: <name>
## Date: <date>

### 1. Process-to-Automation Matrix
| Process | Object | Trigger Event | Tool | Timing | Complexity | Decision Guide Reference |
|---------|--------|---------------|------|--------|------------|--------------------------|

### 2. Object Automation Inventory
#### <Object Name>
| Exec Order | Automation Name | Type | Trigger Event | What It Does |
|------------|-----------------|------|---------------|--------------|
- **Conflict Risk**: Low / Medium / High
- **Notes**: ...

### 3. Conflict Analysis
| Object | Conflict Type | Automations Involved | Risk | Mitigation |
|--------|---------------|----------------------|------|------------|

### 4. Decision Rationale
#### <Process Name>
- **Chosen Tool**: ...
- **Why**: (reference decision guide criteria)
- **Alternatives Considered**: ...
- **Trade-offs**: ...
- **Decision Guide**: [Record-Triggered Automation](https://architect.salesforce.com/decision-guides/trigger-automation) or [Async Processing](https://architect.salesforce.com/decision-guides/async-processing)

### 5. Implementation Sequence
1. Deploy <first automation> (no dependencies)
2. Deploy <second automation> (depends on #1)
...
(Parent automations and shared utilities before dependents)

### 6. Governor Limit Risk Assessment
| Object | # Automations | Est. SOQL Queries | Est. DML Statements | Risk | Notes |
|--------|---------------|-------------------|---------------------|------|-------|
```

## Important

- The architect decision guides are authoritative. Always fetch and reference them rather than relying on general knowledge.
- Challenge "just add a flow" thinking. Every automation on a shared object increases conflict risk and governor limit pressure.
- Before-save flows are free (no DML cost). Prefer them for same-record field updates over after-save flows or triggers.
- Consolidation is a valid recommendation. Multiple small automations on one object are often better merged into one.
- For `audit` mode: present findings without judgment first, then recommend improvements. Clients built what they built for a reason.
- Always check for legacy automation (Workflow Rules, Process Builders) and recommend migration to Flows or Apex where appropriate.
- Recursion guards are non-negotiable in Apex trigger handlers. Flag any handler missing a static boolean guard.
