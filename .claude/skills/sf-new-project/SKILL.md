---
name: sf-new-project
description: Scaffold a new SFDX project for a client engagement with CLAUDE.md, context.yaml, git, and org connection.
user-invocable: true
---

# Salesforce New Project

Scaffold a complete SFDX project for a client engagement.

## Arguments

Required:
- `<project-name>` — kebab-case project name (e.g., `acme-billing-enhancement`)

Optional:
- `<org-alias>` — connect to an existing authorized org
- `<template>` — `standard` (default), `empty`, or `analytics`

## Workflow

### Step 1: Create the SFDX Project

```bash
cd "<your-salesforce-workspace-directory>"
sf project generate --name <project-name> --template standard --manifest
cd <project-name>
```

### Step 2: Initialize Git

```bash
git init
git add -A
git commit -m "chore: scaffold SFDX project"
```

### Step 3: Create Project CLAUDE.md

Write a `CLAUDE.md` in the project root:

```markdown
# <Project Name>

SFDX project for <client/purpose — ask the user>.

## Org Connection

- Alias: <org-alias or "not yet connected — run /sf-org-setup">
- Type: <sandbox/scratch/production>
- API Version: <from sfdx-project.json>

## Test Commands

```bash
# Run all local tests
sf apex run test --test-level RunLocalTests --result-format human -o <org-alias>

# Run specific test class
sf apex run test --class-names <TestClassName> --result-format human -o <org-alias>

# Run Code Analyzer
sf scanner run --target force-app --format table
```

## Conventions

Inherits from workspace CLAUDE.md. Project-specific overrides:
- <to be filled as project evolves>
```

### Step 4: Create Project context.yaml

Write `.claude/context.yaml`:

```yaml
project:
  name: <project-name>
  type: sfdx
  client: <ask user or "TBD">
  org_alias: <org-alias or "not connected">

structure:
  apex: force-app/main/default/classes/
  lwc: force-app/main/default/lwc/
  triggers: force-app/main/default/triggers/
  flows: force-app/main/default/flows/
  objects: force-app/main/default/objects/

test_commands:
  all: "sf apex run test --test-level RunLocalTests --result-format human -o <org-alias>"
  specific: "sf apex run test --class-names {class} --result-format human -o <org-alias>"
  analyze: "sf scanner run --target force-app --format table"
```

### Step 5: Create .claude Directory

```bash
mkdir -p .claude/skills .claude/rules
```

Create `.claude/settings.json` if an org alias was provided:

```json
{
  "mcpServers": {
    "salesforce-dx": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp",
        "--orgs", "<ORG_ALIAS>",
        "--toolsets", "orgs,metadata,data,users,testing,code-analysis,lwc-experts"]
    }
  }
}
```

### Step 6: Set Up .gitignore

Replace the project `.gitignore` with a comprehensive SFDX + Claude Code gitignore:

```
# Salesforce auth and local config (CRITICAL - contains credentials)
.sfdx/
.sf/
.env
.env.*

# Claude Code local config
.claude/settings.local.json

# Node
node_modules/

# Logs
*.log
deploy-results/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db
```

### Step 7: Commit and Summary

```bash
git add -A
git commit -m "chore: add Claude Code config and project conventions"
```

Print summary:
```
Project created: <project-name>
Location: ./<project-name>
Org: <connected or "run /sf-org-setup <alias> to connect">
Git: initialized with 2 commits

Directory structure:
  force-app/main/default/   — metadata source
  .claude/                   — Claude Code config
  CLAUDE.md                  — project conventions

Next steps:
- Run /sf-org-setup <alias> to connect a Salesforce org (if not already)
- Start building: /sf-component lwc <ComponentName>
```

## Notes

- Each client project is a separate git repo inside the Salesforce Workspace
- Never share org credentials or metadata across client projects
- The workspace CLAUDE.md provides baseline conventions; project CLAUDE.md adds specifics
