---
name: sf-workspace-setup
description: Onboard a new user to the Salesforce Workspace from scratch. Verifies prerequisites, MCP configuration, Obsidian vault structure, and git hooks. Prints a status report and suggested first steps.
user-invocable: true
model: sonnet
---

# Salesforce Workspace Setup

Verify and configure a new installation of the Salesforce Workspace. Safe to re-run at any time — existing configuration is never overwritten.

## Arguments

- Optional: `--check-only` — report status without making any changes (directories, symlinks, config blocks)

## Workflow

### Step 1: Verify Prerequisites

Run each check and collect results into a status table. Do not abort early — complete all checks first.

#### SF CLI
```bash
sf --version
```
- Pass: record version string
- Fail: note that install command is `source ~/.nvm/nvm.sh && npm install -g @salesforce/cli`

#### Java 11+
```bash
java -version 2>&1
```
Parse the version number from the output (look for `version "11`, `version "17`, `version "21`, etc.).
- Pass: record version
- Fail: note "Java 11+ required for Code Analyzer and Apex compilation. Install via `sudo apt install openjdk-17-jdk` (run in your terminal — sudo is not available in Claude sessions)."

#### Node via nvm
```bash
source ~/.nvm/nvm.sh 2>/dev/null && node --version
```
- Pass: record version; flag as warning if major version < 24
- Fail: note "nvm not found or Node not installed. Run `nvm install 24` in your terminal."

#### Git GPG signing
```bash
git config --global commit.gpgsign
git config --global gpg.format
git config --global user.signingkey
```
- Pass: `commit.gpgsign` is `true`
- Warning: `commit.gpgsign` is `false` or unset — note "GPG signing is required per workspace conventions. Set with: `git config --global commit.gpgsign true`"
- Note: do not attempt to configure this automatically; signing key setup is user-specific

---

### Step 2: Verify Context7 MCP (Global)

```bash
cat ~/.claude/settings.json 2>/dev/null || echo "NOT_FOUND"
```

Parse the JSON. Check whether `mcpServers` contains a key matching `context7` (or a key whose `args` array includes `@upstash/context7-mcp`).

**If found:** record as configured.

**If not found:** print the config block to add manually:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

Note: "Add this to `~/.claude/settings.json` under `mcpServers`. Context7 provides live Salesforce docs, SLDS, and Apex Recipes to all Claude sessions."

Do **not** write to `~/.claude/settings.json` — global settings require manual user action.

---

### Step 3: Salesforce DX MCP (Per-Org)

This MCP is not configured workspace-wide. It is set up per-org via `/sf-org-setup`.

Print:
```
Salesforce DX MCP: per-org (not checked here)
  → Run /sf-org-setup <ORG_ALIAS> to connect your first org.
  → Each org gets its own MCP entry in the project-level .claude/settings.json.
```

If `sf org list --json 2>/dev/null` returns any authenticated orgs, list them as:
```
Authenticated orgs found: <alias> (<username>) [<sandbox|production|scratch>]
```
and note: "Run `/sf-org-setup <alias>` for any org that isn't yet wired to the DX MCP."

---

### Step 4: Verify Workspace MCPs (.mcp.json)

```bash
test -f "/home/luke/work/salesforce/.mcp.json" && echo "EXISTS" || echo "MISSING"
```

**If EXISTS:** Read it and check for both `advanced-communities` and `sf-tsmz` keys in `mcpServers`.
- Both present: record as configured
- Partially configured: list which keys are missing

**If MISSING or incomplete:** Print the template below and note "Create `.mcp.json` in the workspace root with this content (fill in any org-specific env vars)."

```json
{
  "mcpServers": {
    "advanced-communities": {
      "command": "npx",
      "args": ["-y", "@advanced-communities/salesforce-mcp-server"],
      "env": {
        "READ_ONLY": "true",
        "ALLOWED_ORGS": ""
      }
    },
    "sf-tsmz": {
      "command": "npx",
      "args": ["-y", "@tsmztech/mcp-server-salesforce"],
      "env": {
        "SF_ORG_ALIAS": ""
      }
    }
  }
}
```

In `--check-only` mode: report status only, do not create the file.

Otherwise: if `.mcp.json` is missing AND the user has not passed `--check-only`, offer to create it with the template above, but ask for confirmation before writing.

---

### Step 5: Verify Obsidian Vault

```bash
test -d "$HOME/Documents/Obsidian Vault" && echo "EXISTS" || echo "MISSING"
```

**If MISSING:** Warn "Obsidian vault not found at ~/Documents/Obsidian Vault. Check your vault location in Obsidian > Preferences > About." Do not create it.

**If EXISTS:** Check the consulting folder structure:
```bash
test -d "$HOME/Documents/Obsidian Vault/Clients" && echo "ok" || echo "missing"
test -d "$HOME/Documents/Obsidian Vault/Engagements" && echo "ok" || echo "missing"
test -d "$HOME/Documents/Obsidian Vault/Cert Prep" && echo "ok" || echo "missing"
```

- If all present: record as configured
- If any missing AND not `--check-only`: create the missing directories
  ```bash
  mkdir -p "$HOME/Documents/Obsidian Vault/Clients"
  mkdir -p "$HOME/Documents/Obsidian Vault/Engagements"
  mkdir -p "$HOME/Documents/Obsidian Vault/Cert Prep"
  ```
  Report which directories were created.
- If any missing AND `--check-only`: list what would be created

---

### Step 6: Verify Git Pre-Commit Hook

```bash
test -L "/home/luke/work/salesforce/.git/hooks/pre-commit" && echo "SYMLINK" \
  || test -f "/home/luke/work/salesforce/.git/hooks/pre-commit" && echo "FILE" \
  || echo "MISSING"
```

Note: the workspace root is a git repo. Individual SFDX project directories are separate repos and must be checked independently.

**If SYMLINK:** Verify the target resolves:
```bash
readlink -f "/home/luke/work/salesforce/.git/hooks/pre-commit"
```
- If resolves to `~/.claude/hooks/git-pre-commit`: record as correctly linked
- If resolves elsewhere: warn with the actual target path

**If FILE (not symlink):** Warn "pre-commit hook exists but is not symlinked from ~/.claude/hooks/git-pre-commit. It may be outdated or project-specific."

**If MISSING:** If not `--check-only`:
```bash
ln -sf ~/.claude/hooks/git-pre-commit \
  "/home/luke/work/salesforce/.git/hooks/pre-commit"
```
Report the symlink was created.

If `--check-only`: note the symlink is missing and show the command to fix it.

---

### Step 7: Validate Workspace Context File

```bash
python3 -c "
import yaml, sys
with open('/home/luke/work/salesforce/.claude/context.yaml') as f:
    data = yaml.safe_load(f)
print('valid')
print('workspace:', data.get('workspace', {}).get('name', 'unknown'))
print('release:', data.get('release', {}).get('current', 'unknown'))
print('skills:', data.get('release', {}).get('skills', 'unknown'))
" 2>&1
```

- If output starts with `valid`: record workspace name, release version, and skill count
- If `FileNotFoundError`: flag as missing — "context.yaml not found. This file is the authoritative workspace source. Check git status."
- If YAML parse error: flag as invalid — print the error and note "Fix the YAML syntax error in .claude/context.yaml before starting a session."

---

### Step 8: Quick-Start Summary

Print a structured report:

```
=== Salesforce Workspace Setup Report ===
Mode: <Full setup / Check only>
Date: <YYYY-MM-DD>

PREREQUISITES
  SF CLI:          <version | MISSING — install: source ~/.nvm/nvm.sh && npm install -g @salesforce/cli>
  Java:            <version | MISSING — see note>
  Node (nvm):      <version | MISSING — nvm install 24>
  GPG signing:     <Enabled | WARNING: disabled>

MCP SERVERS
  Context7:        <Configured in ~/.claude/settings.json | MISSING — see config block above>
  Salesforce DX:   Per-org (use /sf-org-setup)
  Authenticated orgs: <list or "none">
  .mcp.json:       <Configured (both servers) | Partial: missing <keys> | MISSING — see template above>

OBSIDIAN
  Vault:           <Found at ~/Documents/Obsidian Vault | NOT FOUND>
  Clients/:        <Exists | Created | Would create (--check-only)>
  Engagements/:    <Exists | Created | Would create (--check-only)>
  Cert Prep/:      <Exists | Created | Would create (--check-only)>

GIT HOOKS
  pre-commit:      <Symlinked to ~/.claude/hooks/git-pre-commit | FILE (not symlink) | MISSING — created | MISSING — run: ln -sf ...>

WORKSPACE CONTEXT
  context.yaml:    <Valid — workspace: Salesforce Workspace, release: 1.3.0, 41 skills | MISSING | INVALID: <error>>

=== STATUS ===
  OK:        X checks passed
  WARNINGS:  X items need attention (listed above)
  ERRORS:    X blockers (listed above)

SUGGESTED FIRST STEPS
  <Dynamically generated based on what failed — see logic below>
```

#### First Steps Logic

Generate the "Suggested First Steps" list dynamically:

1. If SF CLI missing → "Install SF CLI: `source ~/.nvm/nvm.sh && npm install -g @salesforce/cli`"
2. If Java missing → "Install Java 11+: run `sudo apt install openjdk-17-jdk` in your terminal"
3. If Node/nvm missing → "Install Node 24 via nvm: `nvm install 24 && nvm use 24`"
4. If GPG signing disabled → "Enable GPG commit signing: `git config --global commit.gpgsign true`"
5. If Context7 not configured → "Add Context7 MCP to ~/.claude/settings.json (config block above), then restart Claude Code"
6. If .mcp.json missing or incomplete → "Create/update .mcp.json in workspace root (template above)"
7. If no authenticated orgs → "Connect your first org: `/sf-org-setup <ORG_ALIAS>`"
8. If context.yaml invalid → "Fix YAML syntax in .claude/context.yaml before proceeding"
9. If all OK → "Workspace is ready. Connect an org with `/sf-org-setup <alias>` or scaffold a project with `/sf-new-project <name>`."

## Flags

- `--check-only`: Run all checks, print the report, but make no changes. Directories are not created, symlinks are not created, no files are written. All "action" items in the report show as "Would do X" rather than "Did X".

## Hard Rules

- Never write to `~/.claude/settings.json`. This file is managed by the user. Only print config blocks for manual insertion.
- Never create or modify SFDX project directories. This skill only manages workspace-level infrastructure.
- Never connect, authenticate, or configure orgs directly. Delegate to `/sf-org-setup`.
- Obsidian vault itself is never created — only the subfolder structure inside a confirmed vault.
- GPG signing keys are never configured automatically — key management is user-specific.
