# Scheduled Tasks

Automated tasks managed via the Claude Code scheduled tasks MCP server.
Run `mcp__scheduled-tasks__list_scheduled_tasks` to see current status.

## Active Tasks

### weekly-client-review
- **Schedule:** Every Monday at 8:30 AM
- **Purpose:** Multi-client weekly status — runs /sf-playbook weekly-review
- **Output:** Status summaries saved to Obsidian vault

### org-health-checks
- **Schedule:** Every Friday at 4:00 PM
- **Purpose:** Org health monitoring — runs /sf-health-check per connected org
- **Output:** Reports saved to .claude/health-reports/

### engagement-retro-reminder
- **Schedule:** Last Friday of each month at 2:00 PM
- **Purpose:** Monthly retrospective — runs /retro on month's git history
- **Output:** Updates to CLAUDE.md and MEMORY.md

### eval-regression-weekly
- **Schedule:** Every Monday at 9:00 AM
- **Purpose:** Run full eval suite, flag regressions vs previous run, update BACKLOG if new failures found
- **Output:** Pass/fail summary in chat; results JSON saved to `.claude/evals/results/`

## Manual Tasks (Not Scheduled)

### Release Notes Analysis
- **Frequency:** 3x per year (Spring, Summer, Winter releases)
- **Command:** `/sf-release-notes <release> --client <name>`
- **Reason:** Manual — requires human review of breaking changes before distributing to clients

## Notes

- Scheduled tasks run in the background via Claude Code daemon
- If a task fails, check Claude Code session logs
- To pause a task: use `mcp__scheduled-tasks__update_scheduled_task` with enabled=false
- Health check reports accumulate in `.claude/health-reports/` — clean up quarterly
