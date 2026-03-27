---
name: sf-playbook
description: Run an orchestrated consulting playbook — sequences skills and agents through a defined workflow with gates, checkpoints, and state persistence.
user-invocable: true
---

# Salesforce Consulting Playbook Runner

Execute multi-step consulting workflows defined as YAML playbooks. Playbooks sequence
skills, subagents, and human review gates into repeatable engagement patterns with
state persistence across sessions.

## Arguments

- `<playbook-name>` — run a named playbook (e.g., `sf-playbook discovery-to-design`)
- `list` — list all available playbooks in `.claude/playbooks/`
- `resume` — resume a paused playbook from its last checkpoint
- `status` — show the current state of an in-progress or paused playbook

## Workflow

### When argument is `list`

1. Scan `.claude/playbooks/` for `*.yaml` files (exclude README)
2. Parse each file and extract `name`, `description`, `version`, and step count
3. Print a table:
   ```
   Playbook                  Version  Steps  Description
   discovery-to-design       1.0.0    5      Run a full consulting discovery...
   migration-assessment      1.0.0    4      Assess an existing org for migration...
   ```
4. Done.

### When argument is `status`

1. Scan `.claude/workflows/.state/` for `*.json` files
2. If none exist, print "No active playbooks." and exit
3. For each state file, print:
   ```
   Playbook: discovery-to-design (v1.0.0)
   Status: paused
   Started: 2026-03-27T10:00:00Z
   Current step: step-03-architecture (Solution Architecture)
   Completed: 2/5 steps
   ```
4. Done.

### When argument is `resume`

1. Scan `.claude/workflows/.state/` for state files with `status: paused`
2. If none, print "No paused playbooks to resume." and exit
3. If multiple paused playbooks, ask the user which one to resume
4. Load the state file and the corresponding playbook YAML
5. Display completed steps and their outputs for context
6. Ask: "Resume from `<current_step>` (<step name>)?"
7. If confirmed, hand off to the orchestrator agent to continue execution

### When argument is `<playbook-name>`

1. Look for `.claude/playbooks/<playbook-name>.yaml`
   - If not found, print available playbooks and exit
2. Parse the playbook YAML
3. Validate against the schema in `.claude/workflow-schema.yaml`:
   - All required top-level fields present
   - Each step has `id`, `name`, `description`, `agent`
   - Steps with `agent: skill` have a `skill` field
   - Input references (`playbook.inputs.*`, `steps.*.outputs.*`) point to valid targets
   - If validation fails, print errors and exit
4. Check for existing state in `.claude/workflows/.state/<playbook-name>.json`
   - If in-progress or paused state exists, ask: "A previous run exists. Resume or start fresh?"
   - If "resume", follow the resume flow above
   - If "start fresh", delete the old state file
5. Collect required inputs:
   - For each input in the playbook's `inputs` block:
     - If `required: true` and no `default`, prompt the user using the `prompt` field
     - If `required: false`, ask only if no default is set
     - Validate input types (e.g., `org-alias` should be a valid sf CLI alias)
6. Initialize state file:
   ```json
   {
     "playbook_name": "<name>",
     "playbook_version": "<version>",
     "started_at": "<now>",
     "updated_at": "<now>",
     "current_step": "<first-step-id>",
     "status": "in_progress",
     "inputs": { ... },
     "completed_steps": [],
     "skipped_steps": [],
     "step_outputs": {}
   }
   ```
7. Create TodoWrite entries for all steps (all `pending`)
8. Hand off to the orchestrator agent at `.claude/agents/consulting/orchestrator.md`
   to execute steps sequentially
9. On completion, present a summary of all outputs and their file locations

## Output Format

### List output
Table of available playbooks with name, version, step count, and description.

### Status output
Current state of active/paused playbooks including progress percentage.

### Run/Resume output
Step-by-step progress updates via TodoWrite, with a final summary:

```
Playbook: discovery-to-design (v1.0.0)
Status: completed
Duration: ~45 minutes across 2 sessions

Deliverables:
  - Architecture Document: .claude/workflows/output/discovery-to-design-acme-corp/step-03-architecture-doc.md
  - Decision Log: .claude/workflows/output/discovery-to-design-acme-corp/step-03-decision-log.md
  - Effort Estimate: .claude/workflows/output/discovery-to-design-acme-corp/step-04-effort-estimate.md
  - Approved Requirements: .claude/workflows/output/discovery-to-design-acme-corp/step-02-approved-requirements.md
```

## Notes

- Playbooks never deploy to production. Any step involving deployment is gated by
  the consulting governance production blocker.
- State files are workspace-local. They are not committed to git.
- Each client engagement should use a separate playbook run. Do not reuse state
  across clients.
- The orchestrator reads agent contracts from `.claude/agents/consulting/agent-contract.yaml`
  to validate subagent invocations.
