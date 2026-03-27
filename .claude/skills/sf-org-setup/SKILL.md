---
name: sf-org-setup
description: Connect a Salesforce org and configure the DX MCP server for Claude Code. Handles auth, toolset selection, and settings.json updates.
user-invocable: true
---

# Salesforce Org Setup

Connect a Salesforce org to this workspace and configure the official Salesforce DX MCP server.

## Arguments

The user provides:
- `<ORG_ALIAS>` — a short name for the org (e.g., `client-dev`, `acme-sandbox`)
- Optionally: toolset selection (defaults to consultant-recommended set)

## Workflow

### Step 1: Check Prerequisites

Verify the SF CLI is available:
```bash
sf --version
```

If not installed:
```bash
source ~/.nvm/nvm.sh && npm install -g @salesforce/cli
```

Verify Java 11+ is available (required for Code Analyzer):
```bash
java -version
```

### Step 2: Authenticate the Org

Run the web login flow:
```bash
sf org login web -a <ORG_ALIAS>
```

This opens a browser for OAuth. Wait for the user to complete authentication.

After auth, verify and check org type:
```bash
sf org display -o <ORG_ALIAS> --json
```

**CRITICAL**: Check the `instanceUrl` and `orgType` in the response.
- If `orgType` contains "Production" or the instance is a production URL, warn the user:
  "This is a PRODUCTION org. Per consulting governance, AI-generated code should only target Developer Sandboxes or Scratch Orgs. Proceed with read-only operations only?"
- If sandbox/scratch, proceed normally.

### Step 3: Select Toolsets

Present the user with toolset options. For an independent consultant, recommend:

**Recommended (consulting default):**
```
--toolsets orgs,metadata,data,users,testing,code-analysis,lwc-experts
```

**Full development (if building heavily in this org):**
```
--toolsets orgs,metadata,data,users,testing,code-analysis,lwc-experts,aura-experts,devops
```

**Minimal (read-only exploration):**
```
--toolsets orgs,data
```

### Step 4: Update Project Settings

If a project-level `.claude/settings.json` exists in the current SFDX project directory, update its `mcpServers` section. Otherwise update the workspace-level settings.

Add this MCP server config:
```json
{
  "mcpServers": {
    "salesforce-dx": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp",
        "--orgs", "<ORG_ALIAS>",
        "--toolsets", "<SELECTED_TOOLSETS>"]
    }
  }
}
```

### Step 5: Verify Connection

Tell the user to restart Claude Code (or the MCP server) so the new config takes effect.

After restart, the Salesforce DX MCP tools will be available as `mcp__salesforce-dx__*`.

### Step 6: Output Summary

Print a summary:
```
Org connected: <ORG_ALIAS>
Org type: <sandbox/scratch/production>
Toolsets: <selected toolsets>
Config written to: <path to settings.json>

Next steps:
- Restart Claude Code to load the MCP server
- Use `sf org display -o <ORG_ALIAS>` to verify anytime
- Run `/sf-new-project <project-name>` to scaffold an SFDX project for this org
```

## Multi-Org Support

If the user has multiple orgs (e.g., dev sandbox + QA sandbox), each gets its own MCP server entry:
```json
{
  "mcpServers": {
    "sf-client-dev": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp", "--orgs", "client-dev", "--toolsets", "orgs,metadata,data,users,testing,code-analysis,lwc-experts"]
    },
    "sf-client-qa": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp", "--orgs", "client-qa", "--toolsets", "orgs,data,testing"]
    }
  }
}
```
