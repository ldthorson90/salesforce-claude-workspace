# Salesforce Claude Workspace

**43 Claude Code skills, 7 consulting playbooks, 17 subagents, 3 MCP servers, and 4 validation hooks for Salesforce professionals.** Drop into any workspace and get production-grade Salesforce development, architecture, and delivery tooling.

---

## Production Safety

> **This workspace is built so that AI-generated code cannot reach a client's production org.**

Three independent layers enforce this:

| Layer | Where | What it does |
|---|---|---|
| **Org connection** | `/sf-org-setup` | Detects production orgs at auth time. Refuses to configure write-capable MCP tools. Only read-only access is permitted for production. |
| **Deploy blocker** | `/sf-deploy` | Checks `isSandbox` and `isScratch` before every deploy. Hard stops if either is false. No override flag exists. |
| **Bash hook** | `.claude/settings.json` | Intercepts `sf project deploy` shell commands. Queries `sf org display` live and blocks execution if the target is a production org. |

**What this means in practice:**
- You can authenticate against a client's production org for read-only analysis (org health checks, metadata review)
- You cannot deploy metadata, run destructive operations, or execute DML against production — the workspace will refuse and explain why
- Sandboxes and scratch orgs are unrestricted
- If something looks wrong, run `sf org display -o <alias>` to verify what you're connected to

---

## New Client Engagement

**How to start a new client in 5 minutes:**

### Step 1 — Open the workspace in Claude Code

```bash
cd "/home/luke/Cowork Projects/Salesforce Workspace"
claude
```

Or from Cowork: open a new Claude Code session and select this folder as the project root.

### Step 2 — Scaffold a project for the client

```
/sf-new-project <client-name>
```

Examples: `/sf-new-project acme-manufacturing`, `/sf-new-project pinnacle-financial`

This creates an isolated SFDX project directory for the client with its own CLAUDE.md, context.yaml, git repo, and .gitignore. **Each client lives in its own directory — nothing crosses over.**

### Step 3 — Connect their sandbox (not production)

```
/sf-org-setup <client-name>-dev
```

This runs `sf org login web`, checks the org type, and configures the DX MCP server. If you accidentally point it at a production org, it will stop and tell you.

Good alias conventions: `acme-dev`, `acme-qa`, `pinnacle-sandbox`

### Step 4 — Run discovery or switch to an existing client

**Starting a new engagement from scratch:**
```
/sf-playbook discovery
```

**Returning to an existing client:**
```
/sf-client-switch acme-manufacturing
```

This loads: org auth status, recent git activity, open Obsidian notes, active playbooks — all in under 5 seconds.

### Step 5 — Build, test, deploy (to sandbox)

```
/sf-deploy acme-dev
```

The deploy skill runs Code Analyzer, executes tests, reports coverage, and tags the deployment in git. It will not deploy to production.

---

## Quick Start (First-Time Setup)

```bash
# Clone into your Cowork Projects folder
git clone https://github.com/ldthorson90/salesforce-claude-workspace.git
cd salesforce-claude-workspace

# Open in Claude Code
claude

# Verify everything is configured correctly
/sf-workspace-setup
```

`/sf-workspace-setup` checks: SF CLI, Node.js, Java, MCP server connectivity, Obsidian vault structure, and git hooks. Run it once when you first set up, and any time something feels off.

---

## Skills (43)

### Setup & Scaffolding

| Skill | What it does |
|---|---|
| `/sf-workspace-setup` | Verify and configure the workspace: prerequisites, MCPs, Obsidian vault, git hooks |
| `/sf-org-setup` | Connect a Salesforce org, configure DX MCP, block production deploys |
| `/sf-new-project` | Scaffold SFDX project with CLAUDE.md, context.yaml, git, .gitignore |
| `/sf-component` | Scaffold LWC components, Apex trigger handlers, service classes |

### Development (Code)

| Skill | What it does |
|---|---|
| `/sf-test-gen` | Generate Apex test classes: bulk, boundary, negative, `@TestSetup`, 85%+ coverage |
| `/sf-validation` | Validation rules and formula fields with null safety, cross-object, regex patterns |
| `/sf-custom-metadata` | Custom Metadata Types, Custom Settings, Custom Labels, feature flag patterns |
| `/sf-integration` | Connected Apps, Named Credentials, callouts, Platform Events, CDC |

### Development (Declarative)

| Skill | What it does |
|---|---|
| `/sf-flow` | Record-triggered, screen, scheduled, orchestration Flows. Subflow design, fault handling |
| `/sf-layout` | Lightning Record Pages, Dynamic Forms, record types, compact layouts |
| `/sf-security` | OWD, sharing rules, profiles vs permission sets, FLS, role hierarchy |
| `/sf-report` | Reports, dashboards, custom report types, cross-filters, dynamic dashboards |
| `/sf-experience` | Experience Cloud sites: templates, guest user security, sharing sets, audiences |

### Cloud Products

| Skill | Aligned to cert |
|---|---|
| `/sf-sales` | **Sales Cloud Consultant** — opportunities, forecasting, territories, products, leads |
| `/sf-service` | **Service Cloud Consultant** — case management, entitlements, knowledge, omnichannel |
| `/sf-data-cloud` | **Data Cloud Consultant** — connect, prepare, harmonize, segment, activate |
| `/sf-agentforce` | **AI Specialist** — agent builder, prompt templates, actions, Trust Layer, testing |
| `/sf-dc-agentforce` | Data Cloud → Agentforce pipeline: unified profiles, segment-gated agents, calculated insights |
| `/sf-commerce` | **B2C Commerce Architect** — storefront, catalog, checkout, SCAPI, PWA Kit |
| `/sf-cpq` | **CPQ Specialist** — product rules, price rules, quote templates, approvals |
| `/sf-omnistudio` | **OmniStudio Consultant** — OmniScripts, FlexCards, Integration Procedures |

### Operations

| Skill | What it does |
|---|---|
| `/sf-deploy` | Deploy with production blocker, Code Analyzer gate, test run, coverage report |
| `/sf-org-analyze` | Analyze existing org metadata, automation, Apex health, security (read-only) |
| `/sf-migration` | Data migration: field mapping, ETL patterns, Bulk API, external IDs, validation |
| `/sf-debug` | Debug logs, governor limit diagnosis, SOQL query plans, async job tracking |
| `/sf-health-check` | Org health scorecard: governor limits, metadata hygiene, automation, security, data quality, tech debt |
| `/sf-release-notes` | Filter Salesforce release notes by active features; Ollama-classified impact summary with breaking changes |

### Client-Facing

| Skill | What it does |
|---|---|
| `/sf-demo-script` | Persona-based demo walkthroughs: click-by-click steps, talking points, edge case recovery |
| `/sf-client-switch` | Load full client context in < 5s: org auth, recent git activity, Obsidian open items, active playbooks |

### Consulting Lifecycle

| Skill | What it does |
|---|---|
| `/sf-discovery` | Guided requirements gathering: stakeholder interview, business process mapping |
| `/sf-gap-analysis` | Current vs desired state: classify requirements as met, partial, or gap |
| `/sf-data-model` | Interactive data model designer with ERD generation and platform limit validation |
| `/sf-automation-map` | Map business processes to automation tools with conflict detection |
| `/sf-solution-design` | Complete pre-build solution design document |
| `/sf-compliance-check` | Pre-delivery quality gate: code, architecture, documentation standards |
| `/sf-estimate` | Session-based complexity estimation with T-shirt sizing and risk multipliers |
| `/sf-user-story` | Convert notes and transcripts to formal user stories with acceptance criteria |
| `/sf-sow-builder` | Generate Statement of Work: scope, phases, deliverables, assumptions, RACI |
| `/sf-status-update` | Client-facing status report from git history — business language, no jargon |
| `/sf-change-request` | Scope change impact analysis with effort delta and formal change request |
| `/sf-cert-prep` | Certification study: scenario, walkthrough, deep-dive, exam-sim modes for 9 cert tracks |

### Architecture & Delivery

| Skill | What it does |
|---|---|
| `/sf-decide` | Route architecture questions to Salesforce decision guides (8 domains) |
| `/sf-handoff` | Generate client deliverables: solution design doc, admin guide, deployment runbook |

---

## Consulting Playbooks (7)

Orchestrated multi-step workflows that sequence skills and subagents with checkpoints.

```
/sf-playbook <name>
```

| Playbook | What it does |
|---|---|
| `discovery` | Client intake → org analysis → gap analysis → requirements → estimate |
| `design` | Data model → automation map → integration → security → solution design assembly |
| `build` | Scaffold → TDD tests → implement → compliance check → deploy |
| `deliver` | Compliance gate → deliverables → training materials → knowledge capture |
| `investigate` | Symptoms → diagnosis → impact → solution → change request |
| `enhance` | Intake → impact analysis → design delta → estimate → build handoff |
| `weekly-review` | Per-client status reports → health checks → Obsidian summary |
| `proposal` | Prospect intake → org analysis → capability assessment → estimate → SOW → proposal doc |

---

## Subagents (17)

Specialized consulting agents in `.claude/agents/consulting/`, invoked automatically from playbooks.

| Category | Agents |
|---|---|
| Discovery & Requirements | `discovery-intake`, `gap-analyst`, `requirements-writer`, `stakeholder-mapper` |
| Design | `data-modeler`, `automation-mapper`, `security-designer`, `integration-architect` |
| Delivery | `scope-estimator`, `sow-builder`, `status-reporter`, `demo-scripter`, `change-request-analyst` |
| Investigation | `issue-diagnostician`, `impact-analyzer` |
| Knowledge | `release-notes-analyst`, `cert-prep-coach` |

---

## Scheduled Tasks

Automated recurring tasks (managed via Claude Code scheduled tasks MCP):

| Task | Schedule | Purpose |
|---|---|---|
| `weekly-client-review` | Mon 8:30 AM | Runs `/sf-playbook weekly-review` for all active clients |
| `org-health-checks` | Fri 4:00 PM | Runs `/sf-health-check` per connected org |
| `eval-regression-weekly` | Mon 9:00 AM | Runs full eval suite, flags regressions, updates BACKLOG |
| `engagement-retro-reminder` | Last Fri of month | Runs `/retro` on month's git history |

See `.claude/scheduled-tasks/README.md` for details and management commands.

---

## MCP Servers (3)

Configured in `.mcp.json`. Active when Claude Code starts.

| Server | Tools | Purpose |
|---|---|---|
| **mcp-mermaid** | Diagram generation | ERD, sequence, flowchart, C4, class diagrams |
| **@advanced-communities/salesforce-mcp-server** | 40 Salesforce tools | Org exploration, SOQL, Apex execution, debug logs. `READ_ONLY=true` by default |
| **@tsmztech/mcp-server-salesforce** | 15 Salesforce tools | Custom object/field creation with auto-FLS, aggregate queries, debug logs |

**Per-org MCP server** (configured via `/sf-org-setup`):

| Server | Purpose |
|---|---|
| **@salesforce/mcp** | Official Salesforce DX MCP. 60+ tools across 12 toolsets. Requires `sf org login web` |

---

## Validation Hooks (4)

Defined in `.claude/settings.json`. Run automatically on file edits.

| Hook | Trigger | What it catches |
|---|---|---|
| **Apex advisory** | PreToolUse (Edit/Write `.cls`/`.trigger`) | Reminds: bulkification, trigger-handler, USER_MODE, 4-space indent |
| **Production deploy blocker** | PreToolUse (Bash `sf deploy`) | Hard-blocks deploy to production orgs. Queries `sf org display` to check |
| **Apex anti-pattern detection** | PostToolUse (Write/Edit `.cls`/`.trigger`) | DML-in-loop, `without sharing`, hardcoded IDs, empty catch blocks, fat triggers |
| **LWC + Flow validation** | PostToolUse (Write/Edit `.html`/`.flow-meta.xml`) | Legacy `if:true`/`if:false`, ternary operators in templates, Flow naming convention |

---

## Documentation Access (Context7)

Query Salesforce documentation directly from Claude Code via [Context7](https://context7.com). No local files needed.

| Library | Snippets | Use for |
|---|---|---|
| `/damecek/salesforce-documentation-context` | 44,802 | Full Salesforce developer docs (Apex, SOQL, admin, metadata) |
| `/websites/v1_lightningdesignsystem` | 10,639 | SLDS components, form layouts, utility classes |
| `/trailheadapps/apex-recipes` | 457 | TriggerHandler framework, best practice code patterns |
| `/beyond-the-cloud-dev/soql-lib` | 874 | Fluent SOQL builder patterns |
| `/salesforcecli/cli` | 577 | SF CLI command reference |
| `/websites/architect_salesforce_well-architected` | 10 | Architecture principles, security patterns |
| `/google/flow-lens` | 40 | Flow XML to UML/Mermaid diagram conversion |

---

## Architecture Decision Guides

Use `/sf-decide` to route architecture questions. Guides are fetched dynamically from the official [Salesforce Architect](https://architect.salesforce.com) site:

| Domain | Source |
|---|---|
| Record-Triggered Automation | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/trigger-automation) |
| Data 360 Provisioning | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/data-360-provisioning) |
| Data 360 Interoperability | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/data-360-interoperability) |
| Step-Based Async Framework | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/async-framework) |
| Asynchronous Processing | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/async-processing) |
| Building Forms | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/forms) |
| Event-Driven Architecture | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/event-driven) |
| Data Integration | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/data-integration) |

---

## Project Structure

```
salesforce-claude-workspace/
├── CLAUDE.md                    # Workspace conventions (loaded by Claude Code)
├── .mcp.json                    # MCP server configuration (3 servers)
├── .claude/
│   ├── context.yaml             # Workspace metadata + Context7 library index
│   ├── settings.json            # Permissions + 4 validation hooks
│   ├── rules/
│   │   ├── consulting-governance.md   # Production safety, challenge-first, client isolation
│   │   └── sf-skills-audit.md         # Jaganpro/sf-skills compatibility notes
│   ├── agents/
│   │   └── consulting/          # 17 specialized consulting subagents
│   ├── playbooks/               # 8 orchestrated consulting workflows
│   ├── scheduled-tasks/         # Scheduled task configs and README
│   └── skills/                  # 43 skills
│       ├── sf-workspace-setup/  ├── sf-sales/
│       ├── sf-org-setup/        ├── sf-service/
│       ├── sf-new-project/      ├── sf-data-cloud/
│       ├── sf-component/        ├── sf-agentforce/
│       ├── sf-test-gen/         ├── sf-dc-agentforce/
│       ├── sf-validation/       ├── sf-commerce/
│       ├── sf-custom-metadata/  ├── sf-cpq/
│       ├── sf-integration/      ├── sf-omnistudio/
│       ├── sf-flow/             ├── sf-deploy/
│       ├── sf-layout/           ├── sf-org-analyze/
│       ├── sf-security/         ├── sf-migration/
│       ├── sf-report/           ├── sf-debug/
│       ├── sf-experience/       ├── sf-health-check/
│       ├── sf-decide/           ├── sf-release-notes/
│       ├── sf-handoff/          ├── sf-demo-script/
│       ├── sf-discovery/        ├── sf-client-switch/
│       ├── sf-gap-analysis/     ├── sf-cert-prep/
│       ├── sf-data-model/       ├── sf-sow-builder/
│       ├── sf-automation-map/   ├── sf-status-update/
│       ├── sf-solution-design/  ├── sf-change-request/
│       ├── sf-compliance-check/ ├── sf-playbook/
│       ├── sf-estimate/         └── sf-user-story/
├── .gitignore
├── LICENSE                      # MIT
└── README.md
```

---

## Customization

### Add a per-org MCP server

Run `/sf-org-setup <alias>` after authenticating an org, or manually add to a project's `.claude/settings.json`:

```json
{
  "mcpServers": {
    "salesforce-dx": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp", "--orgs", "my-sandbox", "--toolsets", "orgs,metadata,data,testing,code-analysis,lwc-experts"]
    }
  }
}
```

### Restrict community MCP to specific orgs

Edit `.mcp.json` and set `ALLOWED_ORGS`:

```json
"env": {
  "READ_ONLY": "true",
  "ALLOWED_ORGS": "client-a-dev,client-b-sandbox"
}
```

### Add skills

Create a directory under `.claude/skills/<skill-name>/` with a `SKILL.md` file:

```yaml
---
name: my-skill
description: One-line description for skill matching.
user-invocable: true
---

# Skill instructions here
```

### Cherry-pick from Jaganpro/sf-skills

The workspace includes a compatibility audit at `.claude/rules/sf-skills-audit.md`. To install individual skills from [Jaganpro/sf-skills](https://github.com/Jaganpro/sf-skills) without the full hook system:

```bash
# Clone and copy specific skills
git clone --depth 1 https://github.com/Jaganpro/sf-skills.git /tmp/sf-skills
cp -r /tmp/sf-skills/skills/sf-apex .claude/skills/
cp -r /tmp/sf-skills/skills/sf-lwc .claude/skills/
rm -rf /tmp/sf-skills
```

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI)
- [Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli) (`sf`)
- Node.js 18+ (for MCP servers via npx)
- Java 11+ (for Code Analyzer)

---

## License

MIT
