---
name: sf-client-switch
description: Fast multi-client context switcher — loads org auth, recent git activity, open Obsidian items, and active workflows for a named client in under 5 seconds.
user-invocable: true
---

# Salesforce Client Context Switch

Instantly load the full context for a client engagement. Surfaces org auth status, recent work, open items from Obsidian notes, and active playbook state — no heavy analysis, just fast context loading.

## Arguments

- `<client-name>` — client or project name (partial match accepted, case-insensitive)

## Workflow

### Step 1: Find Project Directory

Search for a matching SFDX project under the workspace:

```bash
find "/home/luke/work/Salesforce Workspace" -maxdepth 2 -name "CLAUDE.md" | xargs grep -li "<client-name>" 2>/dev/null
```

Also check directory names:
```bash
ls "/home/luke/work/Salesforce Workspace/" | grep -i "<client-name>"
```

If multiple matches found: list them and ask the user to confirm which one.

If no match found: print "No project found matching '<client-name>'. Available projects:" and list all directories containing a CLAUDE.md.

### Step 2: Load Project CLAUDE.md

Read `<project-dir>/CLAUDE.md`. Extract:
- Org alias (look for `org_alias:`, `org-alias:`, or `sf org login` references)
- Engagement type
- Active cloud products
- Any pinned notes or current focus areas

### Step 3: Check Org Auth Status

```bash
sf org display -o <org-alias> --json 2>/dev/null
```

Parse the JSON:
- If successful: extract `orgName`, `instanceUrl`, `isSandbox`, `isScratch`, `expirationDate`
- If auth error: mark as "⚠ needs re-auth"
- If no org alias found in CLAUDE.md: mark as "— no org configured"

Do not attempt login or any org operations. Read-only status check only.

### Step 4: Recent Git Activity

```bash
git -C "<project-dir>" log --oneline --format="%ar: %s" -10 2>/dev/null
```

If not a git repo: note "— not a git repository"

### Step 5: Check for Active Playbook State

Look for state files:
```bash
ls "<project-dir>/.claude/playbook-state/" 2>/dev/null
```

If state files exist, read each one to extract:
- Playbook name
- Current step number and name
- Last updated timestamp

### Step 6: Search Obsidian for Open Items

Search the Obsidian vault for notes related to this client:
```bash
grep -rl "<client-name>" "/home/luke/Documents/Obsidian Vault/" 2>/dev/null | head -10
```

For the most recently modified matching note:
```bash
ls -t $(grep -rl "<client-name>" "/home/luke/Documents/Obsidian Vault/" 2>/dev/null) | head -3
```

Read the top 1-2 most recent notes. Extract:
- Any lines with `- [ ]` (open checkboxes / open items)
- Any lines with `TODO`, `ACTION`, `FOLLOW UP`, `NEXT:`
- The last meeting date (look for date headings)

### Step 7: Output Status Summary

Print the following (keep it concise — this is a quick context load, not a report):

```
## Client: <ClientName>
**Project:** <project-dir>
**Org:** <org-alias> (<Sandbox / Scratch Org / Production>) — <org instance URL>
**Auth:** ✓ valid  |  ⚠ expires <date>  |  ⚠ needs re-auth

---

### Recent Work
<date>: <commit message>
<date>: <commit message>
<date>: <commit message>
(last 5 shown)

---

### Open Items (Obsidian)
- [ ] <item from notes>
- [ ] <item from notes>
Last note: <note title> (<relative date>)
(or: No Obsidian notes found for this client)

---

### Active Workflows
- <playbook-name>: Step <N> — <step description> (updated <relative date>)
(or: No active playbooks)

---

### Quick Actions
▶  Continue work:     cd "<project-dir>"
▶  Open org:          sf org open -o <org-alias>
▶  Re-auth if needed: sf org login web -a <org-alias>
▶  Start playbook:    /sf-playbook <suggested-playbook>
```

**Performance target:** Complete in under 5 seconds. No SOQL queries, no metadata retrieval, no code analysis. Context loading only.

## Output

A concise status summary printed to the console. No files written unless explicitly requested.
