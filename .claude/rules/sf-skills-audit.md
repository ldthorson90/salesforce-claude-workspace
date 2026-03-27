# Jaganpro/sf-skills Compatibility Audit

Audited: 2026-03-27
Status: EVALUATE BEFORE INSTALLING — hook conflicts identified

## Overview

33 skills (not 25 as originally reported), 7 agents, MIT license, actively maintained.
Installs to `~/.claude/` (global level), which means it affects ALL projects.

## Hook Conflicts

sf-skills ships 5 hooks that WILL conflict with existing global and workspace hooks:

| sf-skills Hook | Existing Conflict | Risk |
|----------------|-------------------|------|
| PreToolUse: Bash/MCP guardrails (prompt/Haiku) | Global: Ollama reachability check, Docker check | LOW — different matchers |
| PreToolUse: SOQL schema validation (command) | None | SAFE |
| PostToolUse: Write/Edit validator dispatcher | Workspace: Apex anti-pattern detection (PostToolUse Write) | HIGH — duplicate Apex validation |
| PostToolUse: Bash debug log analysis (agent/Haiku) | Global: Bash error logger | MEDIUM — both fire on Bash |
| SessionStart: session directory lifecycle | None | SAFE |

## Key Conflicts

1. **PostToolUse Write/Edit**: sf-skills runs Prettier + LSP + 90-point scorer + PMD on
   every Apex write. Our workspace hook runs simpler regex checks for SOQL-in-loops,
   without-sharing, and hardcoded IDs. If both run, Apex writes get double-validated
   with overlapping checks. The sf-skills version is more comprehensive but slower (10-30s).

2. **PostToolUse Bash**: sf-skills runs a Haiku agent for debug log analysis. Our global
   hook runs a Python error logger. Both fire on every Bash tool use. The Haiku agent
   adds latency and token cost.

3. **Global installation**: sf-skills installs to `~/.claude/skills/` and `~/.claude/hooks/`.
   This means it runs globally on all projects — not just Salesforce workspaces.

## Recommendations

**Do NOT blind-install.** Instead:

1. Cherry-pick individual skills by copying their SKILL.md files to the workspace
   `.claude/skills/` directory (project-level, not global)
2. Skip the hook system entirely — our workspace hooks are lighter and sufficient
3. If you want the validator dispatcher, install it manually and disable the workspace
   PostToolUse hook to avoid duplication

**Worth cherry-picking:**
- `sf-apex` — comprehensive Apex generation rules
- `sf-flow` — Flow best practices (110-point scorer)
- `sf-lwc` — LWC with SLDS scoring (165 points)
- `sf-soql` — Query optimization with Query Plan API
- `sf-testing` — test generation patterns
- `sf-deploy` — deployment best practices

**Skip (not relevant for independent consultant):**
- `sf-datacloud-*` (8 skills) — only if client uses Data Cloud
- `sf-industry-commoncore-*` (6 skills) — only if client uses Industries
- `sf-ai-agentforce-*` (5 skills) — only if client uses Agentforce
- `sf-diagram-nanobananapro` — vendor-specific tool

## ehebert7/salesforce-claude-framework

Lighter framework. The test-class-generator agent produces similar output to our
`/sf-test-gen` skill. The "strategic plan architect" agent is interesting for
requirement analysis but overlaps with consulting governance rules.

**Not worth installing** — our custom skills cover the same ground with better
integration into the existing pipeline. The session persistence layer conflicts
with our memory system.
