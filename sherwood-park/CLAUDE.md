# RVA Sherwood Park — SFDX Project

Read `~/.claude/brain.yaml` and `~/work/cliffrose/salesforce/.claude/context.yaml` for workspace context.
Read `~/work/sherwood-park/.claude/context.yaml` for contract and contact details.
Read `~/work/sherwood-park/CLAUDE.md` for engagement rules.

## Project State

- **Org:** Not yet connected. Run `sf org login web -a sherwood-dev` when credentials are available.
- **sfdx-project.json:** Not yet initialized. Run `sf project generate -n sherwood-park` after org login.
- **Sandbox:** `sherwood-dev` — to be created after production org confirmed.

## Org Rules

- Org type is likely NPSP nonprofit — confirm via `sf org display -o sherwood-dev` before any deploy.
- Never push metadata to production. All work targets `sherwood-dev`.
- Run `sf scanner:run --target force-app --format table` before any deployment.

## SF CLI Pattern

```bash
source ~/.nvm/nvm.sh
sf data query -q "SELECT Id, Name FROM Account LIMIT 5" -o sherwood-dev
# Tooling API (Flows, Apex, Triggers):
sf data query -q "SELECT Id, MasterLabel FROM FlowDefinition" -o sherwood-dev -t
```

## Engagement Reference

- Consulting engagement: `~/work/sherwood-park/`
- Active tasks: `~/work/sherwood-park/tasks.md`
- Worklog: `~/work/sherwood-park/worklog.md`
