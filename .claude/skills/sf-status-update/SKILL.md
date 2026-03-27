---
name: sf-status-update
description: Generate client-facing status update from git history. Translates commits to business language, groups by completed/in-progress/upcoming, and identifies blockers.
model: sonnet
user_invocable: true
---

# /sf-status-update — Client-Facing Status Report

Generate a business-language status update from git history and project state.

## Trigger

User invokes `/sf-status-update` or asks for a status update / progress report.

## Inputs

1. **Project Path** — path to the SFDX project directory (default: current directory)
2. **Client Name** — for report header
3. **Reporting Period** — date range or "last week" / "last sprint" (default: last 7 days)
4. **Output Target** — file, clipboard, or Obsidian (default: file)

## Execution

### Step 1: Gather Raw Data

```bash
# Get commits in reporting period
git log --since="<start_date>" --until="<end_date>" --oneline --no-merges

# Get current branch state
git branch -a

# Get recent tags
git tag --sort=-creatordate | head -5
```

### Step 2: Classify Commits

Map each commit to a business category using conventional commit prefixes:
- `feat:` → **Completed Features** — new capabilities delivered
- `fix:` → **Issues Resolved** — bugs fixed or issues addressed
- `refactor:` → **Technical Improvements** — platform stability/performance
- `test:` → **Quality Assurance** — test coverage and validation
- `docs:` → **Documentation** — guides and reference materials
- `chore:` → (omit from client report — internal housekeeping)

### Step 3: Identify Status Categories

**Completed** — merged to main or release branch during period
**In Progress** — on active branches, not yet merged
**Upcoming** — referenced in issues/milestones but not started
**Blockers** — stalled branches, failed deployments, open questions

### Step 4: Generate Report

Write in business language (no technical jargon):
- Lead with achievements and value delivered
- Frame technical work in business terms ("improved data processing reliability" not "refactored batch Apex")
- Flag blockers with what's needed to unblock
- Include a 1-sentence outlook for next period

### Step 5: Output

Based on output target:
- **file**: Write `status_update_<date>.md` to project directory
- **clipboard**: Copy to clipboard (not supported in all environments)
- **obsidian**: Create note in Obsidian vault under `Consulting/<client>/Status Updates/`

## Quality Criteria

- No technical jargon visible to client (no commit hashes, branch names, file paths)
- Every completed item maps to a business outcome
- Blockers include clear next steps
- Report is concise (1 page or less)
- Dates are absolute, not relative

## Output Files

| File | Description |
|------|-------------|
| `status_update_<date>.md` | Client-facing status report |
