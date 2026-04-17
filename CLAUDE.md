# Salesforce Workspace

Multi-project workspace for Salesforce development, architecture, and consulting.
Not a single git repo; contains SFDX projects, reference materials, and decision guides.

## Structure

```
salesforce/
├── salesforce-architect-decision-guides/   # Architecture reference (read-only)
├── <sfdx-project>/                         # Individual SFDX projects (own git repos)
└── .claude/
    ├── skills/          # 43 skills: setup, dev, ops, consulting lifecycle, cloud products
    ├── agents/          # 17 consulting subagents, orchestrated by orchestrator.md
    ├── rules/           # Dev standards (Apex, LWC, Flow), governance, skills audit
    └── scheduled-tasks/ # Automated recurring tasks
```

## Session Context

Used for Salesforce architecture/design, SFDX development (Apex, LWC, Flows),
consulting deliverables, and exam prep.

## Development Standards

Loaded automatically via `.claude/rules/` with glob-scoped frontmatter:

- **`rules/apex-standards.md`** — Apex style, bulkification, SOQL, code analysis gate, testing
- **`rules/lwc-standards.md`** — LWC templates, wire service, SLDS, linting
- **`rules/flow-standards.md`** — Flow naming, subflows, fault paths
- **`rules/consulting-governance.md`** — Org safety, challenge pattern, engagement hygiene, code review posture
- **`rules/sf-skills-audit.md`** — Jaganpro/sf-skills compatibility analysis and cherry-pick recommendations

## Architecture Reference

Use `/sf-decide` to route architecture questions. Guides fetched via Context7:

1. [Record-Triggered Automation](https://architect.salesforce.com/decision-guides/trigger-automation)
2. [Data 360 Provisioning](https://architect.salesforce.com/decision-guides/data-360-provisioning)
3. [Data 360 Interoperability](https://architect.salesforce.com/decision-guides/data-360-interoperability)
4. [Step-Based Async Framework](https://architect.salesforce.com/decision-guides/async-framework)
5. [Asynchronous Processing](https://architect.salesforce.com/decision-guides/async-processing)
6. [Building Forms](https://architect.salesforce.com/decision-guides/forms)
7. [Event-Driven Architecture](https://architect.salesforce.com/decision-guides/event-driven)
8. [Data Integration](https://architect.salesforce.com/decision-guides/data-integration)

Optionally download to `salesforce-architect-decision-guides/` for offline use.

## MCP Servers

All configured in `.mcp.json`. Load toolsets selectively to save context.

- **Salesforce DX MCP** — `npx -y @salesforce/mcp --orgs <ALIAS>`. Toolsets: orgs, data, metadata, users, testing, code-analysis, lwc-experts, aura-experts, devops, enrich_metadata, mobile, scale-products
- **Advanced Communities SF MCP** — 40 tools, `READ_ONLY=true` default. Set `ALLOWED_ORGS` for multi-client safety
- **tsmztech SF MCP** — Custom object/field creation with auto-FLS, aggregate queries, debug logs
- **Context7** — Salesforce docs lookup. Key libraries: `/damecek/salesforce-documentation-context` (44K snippets), `/websites/v1_lightningdesignsystem` (10K), `/trailheadapps/apex-recipes`
- **Mermaid** — Architecture diagrams (ERD, sequence, flowchart, C4, class)

## Scheduled Tasks

See `.claude/scheduled-tasks/README.md` for details.

| Task | Schedule | Purpose |
|------|----------|---------|
| `weekly-client-review` | Mon 8:30 AM | `/sf-playbook weekly-review` for all active clients |
| `org-health-checks` | Fri 4:00 PM | `/sf-health-check` per connected org |
| `engagement-retro-reminder` | Last Fri/month | `/retro` on month's git history |

Manual (not scheduled): `/sf-release-notes` — run 3x/year around Salesforce releases.
