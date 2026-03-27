# Playbook Definitions

Playbooks are YAML files that define multi-step consulting workflows. They live in this
directory and are executed via `/sf-playbook <name>`.

## Schema Reference

See `.claude/workflow-schema.yaml` for the full schema definition, field types, and
validation rules.

## Creating a New Playbook

1. Copy the template below into a new file: `<playbook-name>.yaml`
2. Define inputs the playbook needs to start
3. Add steps in execution order
4. Mark steps with `checkpoint: true` at natural pause points
5. Define final outputs with `source_step` references

## Minimal Template

```yaml
name: my-playbook
description: What this playbook accomplishes in one line.
version: "1.0.0"

inputs:
  client_name:
    type: string
    description: Client or project name
    required: true
    prompt: "Client name?"

steps:
  - id: step-01
    name: Gather Requirements
    description: Collect and structure requirements from available inputs.
    agent: skill
    skill: sf-org-analyze
    inputs:
      org_alias: playbook.inputs.org_alias
    outputs:
      analysis:
        type: file
        description: Analysis output
    checkpoint: true
    on_failure: halt

  - id: step-02
    name: Review with Stakeholder
    description: Present findings for validation.
    agent: user
    inputs:
      analysis: steps.step-01.outputs.analysis
    outputs:
      approved:
        type: boolean
        description: Whether the analysis is approved
    gate: client-approved
    on_failure: halt

  - id: step-03
    name: Generate Deliverable
    description: Produce the final deliverable from approved inputs.
    agent: skill
    skill: sf-handoff
    inputs:
      source: steps.step-01.outputs.analysis
    outputs:
      deliverable:
        type: file
        description: Final deliverable document
    on_failure: halt

outputs:
  deliverable:
    type: file
    description: The produced deliverable
    source_step: step-03
```

## Conventions

- **Naming**: kebab-case filenames matching the `name` field (e.g., `discovery-to-design.yaml`)
- **Step IDs**: `step-NN` format, zero-padded, sequential (step-01, step-02, ...)
- **Checkpoints**: place after steps that involve significant work or user interaction.
  Checkpoints save state so the playbook can resume if the session ends.
- **Gates**: use for steps that require explicit approval. Common gate values:
  - `client-approved` — user confirms deliverable meets expectations
  - `user-confirmed` — user acknowledges information
  - `code-analyzer-clean` — no Critical/High violations
  - `tests-passing` — all Apex tests pass
- **Agent types**:
  - `user` — pause for human input
  - `skill` — invoke a `/sf-*` skill (specify in `skill` field)
  - `<agent-name>` — invoke a subagent (must have a contract in `agent-contract.yaml`)
