# Salesforce Claude Workspace

**24 Claude Code skills, 3 MCP servers, and 4 validation hooks for independent Salesforce consultants.** Drop into any workspace and get production-grade Salesforce development, architecture, and delivery tooling.

---

## Quick Start

```bash
# Clone into your working directory
git clone https://github.com/ldthorson90/salesforce-claude-workspace.git
cd salesforce-claude-workspace

# Open in Claude Code
claude
```

Claude Code automatically loads the skills, hooks, and MCP servers from `.claude/` and `.mcp.json`.

**First steps inside Claude Code:**
```
/sf-org-setup my-sandbox        # Connect a Salesforce org
/sf-new-project acme-billing    # Scaffold an SFDX project
/sf-component lwc InvoiceTable  # Start building
```

---

## Skills (24)

### Setup & Scaffolding

| Skill | What it does |
|---|---|
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
| `/sf-commerce` | **B2C Commerce Architect** — storefront, catalog, checkout, SCAPI, PWA Kit |
| `/sf-cpq` | **CPQ Specialist** — product rules, price rules, quote templates, approvals |
| `/sf-omnistudio` | **OmniStudio Consultant** — OmniScripts, FlexCards, Integration Procedures |

### Operations

| Skill | What it does |
|---|---|
| `/sf-deploy` | Deploy with production blocker, Code Analyzer gate, test run, coverage report, git tag |
| `/sf-org-analyze` | Analyze existing org metadata, automation, Apex health, security (read-only) |
| `/sf-migration` | Data migration: field mapping, ETL patterns, Bulk API, external IDs, validation |
| `/sf-debug` | Debug logs, governor limit diagnosis, SOQL query plans, async job tracking |

### Architecture & Delivery

| Skill | What it does |
|---|---|
| `/sf-decide` | Route architecture questions to Salesforce decision guides (8 domains) |
| `/sf-handoff` | Generate client deliverables: solution design doc, admin guide, deployment runbook |

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
| **Apex anti-pattern detection** | PostToolUse (Write/Edit `.cls`/`.trigger`) | DML-in-loop (including for-each), `without sharing`, hardcoded IDs, empty catch blocks, dynamic SOQL concatenation, fat triggers |
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

**Example query in Claude Code:**
> "What are the governor limits for SOQL queries in a single transaction?"

Claude will query Context7 automatically when the workspace is loaded.

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
│   └── skills/                  # 24 skills
│       ├── sf-org-setup/        ├── sf-sales/
│       ├── sf-new-project/      ├── sf-service/
│       ├── sf-test-gen/         ├── sf-data-cloud/
│       ├── sf-validation/       ├── sf-agentforce/
│       ├── sf-custom-metadata/  ├── sf-commerce/
│       ├── sf-integration/      ├── sf-cpq/
│       ├── sf-flow/             ├── sf-omnistudio/
│       ├── sf-layout/           ├── sf-deploy/
│       ├── sf-security/         ├── sf-org-analyze/
│       ├── sf-report/           ├── sf-migration/
│       ├── sf-experience/       ├── sf-debug/
│       ├── sf-decide/           └── sf-handoff/
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
