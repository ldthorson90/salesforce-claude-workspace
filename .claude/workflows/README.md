# Workflow State Persistence

Playbook execution state is persisted to disk so workflows can resume across sessions.

## State File Location

```
.claude/workflows/.state/<playbook-name>.json
```

One state file per playbook. Overwritten on each checkpoint. Do not commit these to git.

## State Structure

```json
{
  "playbook_name": "discovery-to-design",
  "playbook_version": "1.0.0",
  "started_at": "2026-03-27T10:00:00Z",
  "updated_at": "2026-03-27T14:22:00Z",
  "current_step": "step-03-architecture",
  "status": "in_progress",
  "inputs": {
    "client_name": "Acme Corp",
    "engagement_type": "implementation",
    "org_alias": "acme-dev"
  },
  "completed_steps": ["step-01-discovery", "step-02-stakeholder-review"],
  "skipped_steps": [],
  "step_outputs": {
    "step-01-discovery": {
      "org_analysis": ".claude/workflows/output/discovery-to-design-acme-corp/step-01-org-analysis.md"
    }
  }
}
```

### Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `playbook_name` | string | Kebab-case playbook identifier |
| `playbook_version` | string | Semver from the playbook definition |
| `started_at` | ISO 8601 | When execution began |
| `updated_at` | ISO 8601 | Last checkpoint write |
| `current_step` | string | Step ID currently executing or next to execute |
| `status` | enum | `in_progress`, `paused`, `completed`, `failed` |
| `inputs` | map | Resolved playbook inputs |
| `completed_steps` | list | Step IDs that finished successfully |
| `skipped_steps` | list | Step IDs skipped due to `on_failure: skip` |
| `step_outputs` | map | Accumulated outputs keyed by step ID |

## Resume Behavior

1. `/sf-playbook resume` reads the most recent state file (by `updated_at`)
2. Displays completed steps and their outputs for context
3. Prompts the user to confirm resumption from `current_step`
4. Re-creates TodoWrite entries with completed steps already marked done
5. Continues execution from the paused step with all prior outputs available

If no paused playbook exists, `/sf-playbook resume` reports that and exits.

## Output Directory

Generated files go to:

```
.claude/workflows/output/<playbook-name>-<client-name>/
```

Files are named with the step ID prefix for traceability.
