# Salesforce Workspace

Multi-project workspace for Salesforce development, architecture, and consulting work.
Not a single git repository — contains SFDX projects, reference materials, and decision guides.

## Workspace Structure

```
Salesforce Workspace/
├── salesforce-architect-decision-guides/   # Architecture reference (read-only)
├── <sfdx-project>/                         # Individual SFDX projects (each is its own git repo)
└── .claude/                                # Workspace-level Claude config
```

## Session Context

This workspace is used for:
- Salesforce architecture and design work (data models, integration patterns, automation)
- SFDX project development (Apex, LWC, Flows)
- Consulting deliverables (architecture diagrams, decision documents)
- Salesforce exam prep and reference

## Salesforce Development Conventions

### Apex
- 4-space indentation, K&R brace style
- Trigger + handler pattern (thin trigger delegates to handler class)
- Bulkify everything: no SOQL/DML in loops, use collections
- `USER_MODE` for CRUD/FLS enforcement
- Explicit access modifiers on all members
- Static recursion guards in trigger handlers
- Test classes: `@TestSetup`, bulk tests (200+ records), positive/negative/boundary cases

### LWC
- 2-space indentation
- Wire service for data access; minimize imperative Apex
- `lwc:if` / `lwc:elseif` / `lwc:else` (not legacy `if:true` / `if:false`)
- No ternary operators in templates (unsupported)
- SLDS utility classes for styling
- Proper `@api`, `@wire`, `@track` decorators
- Error handling via `ShowToastEvent`

### Flows
- Naming: `<Object>_<Action>_<Context>` (e.g., `Account_Update_AfterInsert`)
- Prefer record-triggered flows over process builders
- Use subflows for reusable logic
- Fault paths on every DML/callout element

### SOQL
- Always use bind variables (never string concatenation for dynamic SOQL)
- Selective queries: indexed fields in WHERE clauses
- Limit large result sets; use FOR loops for bulk processing

## Skills

### Setup & Scaffolding
- **`/sf-org-setup <ORG_ALIAS>`** — Connect org, configure DX MCP, production safety check
- **`/sf-new-project <project-name>`** — Scaffold SFDX project with full config and .gitignore
- **`/sf-component <type> <name>`** — (Global) Scaffold LWC, Apex trigger handlers, or service classes

### Development — Code
- **`/sf-test-gen <ClassName>`** — Generate Apex test classes (bulk/boundary/negative scenarios)
- **`/sf-validation rule|formula <Object>`** — Validation rules and formula fields with null safety
- **`/sf-custom-metadata <type>`** — Custom Metadata Types, Custom Settings, feature flag patterns
- **`/sf-integration design|callout|platform-event`** — Connected Apps, Named Credentials, callouts, events

### Development — Declarative
- **`/sf-flow <type> <Object>`** — Flows: record-triggered, screen, scheduled, subflows, fault handling
- **`/sf-layout page|dynamic-form <Object>`** — Lightning pages, Dynamic Forms, record types
- **`/sf-security design|audit|permset`** — OWD, sharing rules, profiles, permission sets, FLS
- **`/sf-report report|dashboard|report-type`** — Reports, dashboards, custom report types
- **`/sf-experience design|security|theme`** — Experience Cloud sites, guest user security

### Operations
- **`/sf-deploy <target-org>`** — Deploy with production blocker, Code Analyzer gate, test run, report
- **`/sf-org-analyze <org-alias>`** — Analyze existing org metadata, automation, tech debt (read-only)
- **`/sf-migration plan|mapping|template`** — Data migration: field mapping, ETL, Bulk API, validation
- **`/sf-debug logs|governor|query|async`** — Troubleshoot: debug logs, governor limits, SOQL performance

### Cloud Products
- **`/sf-sales`** — Sales Cloud: opportunities, forecasting, territories, products, lead management
- **`/sf-service`** — Service Cloud: case management, entitlements, knowledge, omnichannel
- **`/sf-data-cloud`** — Data Cloud: connect, prepare, harmonize, segment, activate, calculated insights
- **`/sf-agentforce`** — Agentforce: AI agents, prompt templates, actions, Trust Layer, testing
- **`/sf-commerce`** — B2C Commerce: storefront, catalog, checkout, SCAPI, cartridges, PWA Kit
- **`/sf-cpq`** — CPQ: product rules, price rules, quote templates, approvals, guided selling
- **`/sf-omnistudio`** — OmniStudio: OmniScripts, FlexCards, Integration Procedures, DataRaptors

### Consulting Lifecycle
- **`/sf-discovery <engagement-type>`** — Guided requirements gathering with stakeholder mapping
- **`/sf-gap-analysis <org-alias>`** — Current vs desired state analysis with gap classification
- **`/sf-data-model design|review|extend`** — Interactive data model designer with platform limit validation
- **`/sf-automation-map design|audit|conflict-check`** — Process-to-automation mapping with decision guides
- **`/sf-solution-design generate|template`** — Complete pre-build solution design document
- **`/sf-compliance-check <org-alias|source-path>`** — Pre-delivery quality gate (code, architecture, docs)
- **`/sf-estimate <requirements-file>`** — Complexity-based scope estimation (sessions, not hours)
- **`/sf-user-story <input-file>`** — Convert notes/transcripts to formal user stories
- **`/sf-sow-builder`** — Generate SOW from requirements + estimates (overview, scope, phases, RACI, change mgmt)
- **`/sf-status-update`** — Client-facing status report from git history (business language, no jargon)
- **`/sf-change-request`** — Scope change impact analysis with effort delta and recommendation

### Workflow Orchestration
- **`/sf-playbook <playbook-name>`** — Run orchestrated consulting playbooks with gates and checkpoints

### Playbooks
- **`/sf-playbook discovery`** — Client discovery: intake interview, org analysis, gap analysis, requirements, estimation
- **`/sf-playbook design`** — Solution design: data model, automation map, integration, security, assembly
- **`/sf-playbook build`** — Implementation sprint: scaffold, TDD tests, implement, compliance, deploy
- **`/sf-playbook deliver`** — Handoff & go-live: compliance gate, deliverables, training, knowledge capture
- **`/sf-playbook investigate`** — Issue investigation: symptoms, diagnosis, impact, solution, change request
- **`/sf-playbook enhance`** — Enhancement request: intake, impact, design delta, estimate, build handoff
- **`/sf-playbook weekly-review`** — Multi-client weekly status: per-client reports, health checks, Obsidian summary

### Architecture & Delivery
- **`/sf-decide`** — Route architecture questions to the right decision guide (8 frameworks)
- **`/sf-handoff design-doc|admin-guide|deploy-runbook`** — Client deliverables for project close

## Agent Library

17 specialized consulting subagents in `.claude/agents/consulting/`, orchestrated by
`orchestrator.md` and invoked from playbooks.

| Category | Agents |
|----------|--------|
| Discovery & Requirements | `discovery-intake`, `gap-analyst`, `requirements-writer`, `stakeholder-mapper` |
| Design | `data-modeler`, `automation-mapper`, `security-designer`, `integration-architect` |
| Delivery | `scope-estimator`, `sow-builder`, `status-reporter`, `demo-scripter`, `change-request-analyst` |
| Investigation | `issue-diagnostician`, `impact-analyzer` |
| Knowledge | `release-notes-analyst`, `cert-prep-coach` |

Agent contracts are defined in `.claude/agents/consulting/agent-contract.yaml`.

## MCP Servers

### Salesforce DX MCP (per-org, configured via /sf-org-setup)
```
npx -y @salesforce/mcp --orgs <ORG_ALIAS> --toolsets orgs,metadata,data,users,testing,code-analysis,lwc-experts
```
Requires pre-authorized org: `sf org login web -a <ALIAS>`

### Toolsets (load selectively to save context)
- `orgs` - Org info, auth, scratch orgs
- `data` - SOQL queries, record operations
- `metadata` - Deploy/retrieve
- `users` - User and permission management
- `testing` - Apex test execution, coverage
- `code-analysis` - PMD, ESLint, CPD, ApexGuru via Code Analyzer v5
- `lwc-experts` - LWC development assistance
- `aura-experts` - Aura-to-LWC migration blueprints
- `devops` - DevOps Center integration
- `enrich_metadata` - Metadata enrichment
- `mobile` / `mobile-core` - Mobile development
- `scale-products` - Scale products tooling

### Mermaid (architecture diagrams)
Configured in `.mcp.json`. Generates ERD, sequence, flowchart, C4, and class diagrams.

### Advanced Communities SF MCP (40 tools, READ_ONLY mode)
Configured in `.mcp.json`. Org exploration, SOQL, Apex execution, debug logs. `READ_ONLY=true` by default.
Set `ALLOWED_ORGS` env var to restrict which orgs Claude can access (multi-client safety).

### tsmztech SF MCP (schema creation, debug logs)
Configured in `.mcp.json`. Custom object/field creation with auto-FLS, aggregate queries,
debug log management. Uses SF CLI auth.

### Context7 (documentation lookup, global)
Query Salesforce docs directly. Key libraries (see context.yaml for full list):
- `/damecek/salesforce-documentation-context` — 44K snippets, full SF developer docs
- `/websites/v1_lightningdesignsystem` — 10K snippets, SLDS components
- `/trailheadapps/apex-recipes` — TriggerHandler framework, best practice patterns

## Consulting Governance

- AI-generated metadata targets Developer Sandbox or Scratch Org ONLY, never production
- Challenge requirements before implementing — check architect decision guides first
- Each client engagement gets its own SFDX project directory (never cross-reference)
- Code must pass Code Analyzer with zero Critical/High violations before delivery
- See `.claude/rules/consulting-governance.md` for full rules

## Architecture Reference

Use `/sf-decide` to route architecture questions. Guides are fetched dynamically via Context7
from the official Salesforce Architect site. Topics covered:
1. Record-Triggered Automation — [source](https://architect.salesforce.com/decision-guides/trigger-automation)
2. Data 360 Provisioning — [source](https://architect.salesforce.com/decision-guides/data-360-provisioning)
3. Data 360 Interoperability — [source](https://architect.salesforce.com/decision-guides/data-360-interoperability)
4. Step-Based Async Framework — [source](https://architect.salesforce.com/decision-guides/async-framework)
5. Asynchronous Processing — [source](https://architect.salesforce.com/decision-guides/async-processing)
6. Building Forms — [source](https://architect.salesforce.com/decision-guides/forms)
7. Event-Driven Architecture — [source](https://architect.salesforce.com/decision-guides/event-driven)
8. Data Integration — [source](https://architect.salesforce.com/decision-guides/data-integration)

Optionally download guides locally to `salesforce-architect-decision-guides/` for offline use.

## Code Analysis

Run Salesforce Code Analyzer before deployment:
- `sf scanner:run --target <path> --format table`
- ApexGuru for Apex-specific anti-pattern detection (via MCP)
- ESLint with `@lwc/eslint-plugin-lwc` for LWC

## Test Commands

Per-project. Each SFDX project should define in its own CLAUDE.md:
```bash
sf apex run test --test-level RunLocalTests --result-format human
```
