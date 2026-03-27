---
name: sf-compliance-check
description: Pre-delivery quality gate — automated checks for Salesforce consulting standards including code quality, security, architecture patterns, and documentation.
user-invocable: true
---

# Compliance Check

Pre-delivery quality gate for Salesforce consulting engagements. Runs automated checks against code quality, architecture patterns, and documentation completeness before client handoff. References consulting governance rules in `.claude/rules/consulting-governance.md`.

## Arguments

- `<org-alias>` — run checks against a connected org (queries metadata remotely)
- `<source-path>` — run checks against local SFDX source directory
- `full` — run all checks (code + architecture + docs)
- `code` — code quality checks only
- `architecture` — architecture pattern checks only
- `docs` — documentation completeness checks only

## Workflow

### Step 1: Determine Scope

Identify the check target:
- If an org alias is provided, verify connectivity with `sf org display -o <org-alias>` and confirm it is not a production org
- If a source path is provided, verify the directory exists and contains SFDX source (look for `sfdx-project.json` or `force-app/`)
- If neither is provided, check the current project directory for SFDX source; if not found, ask the user which to check
- If `full`, `code`, `architecture`, or `docs` is specified, filter to those check categories; default is `full`

Initialize the results tracker:
```
results = { passed: [], failed: [], warnings: [] }
```

### Step 2: Code Quality Checks

Run when scope includes `code` or `full`.

| Check | Tool | Pass Criteria | Severity |
|---|---|---|---|
| No SOQL in loops | Code Analyzer (`mcp__sf-advanced__run_code_analyzer`) / regex fallback | Zero violations | CRITICAL |
| No DML in loops | Code Analyzer / regex fallback | Zero violations | CRITICAL |
| No hardcoded IDs | Regex scan for `[a-zA-Z0-9]{15}` and `[a-zA-Z0-9]{18}` patterns matching Salesforce ID format in `.cls` and `.js` files | Zero 15/18-char ID patterns in source code | HIGH |
| Test coverage | `sf apex run test --test-level RunLocalTests --code-coverage` (org only) | >= 85% per class, >= 90% overall | HIGH |
| No empty catch blocks | Regex: `catch\s*\([^)]*\)\s*\{\s*\}` in `.cls` files | Zero violations | HIGH |
| Bind variables in SOQL | Regex: detect string concatenation in SOQL queries (e.g., `'SELECT.*' +` patterns) | No string concatenation in queries | CRITICAL |
| No deprecated APIs | Code Analyzer | Zero violations | MEDIUM |
| Code Analyzer clean | `mcp__sf-advanced__run_code_analyzer` with ruleSelector `["all"]` | Zero Critical, Zero High | CRITICAL |

For Code Analyzer checks:
1. Try `mcp__sf-advanced__run_code_analyzer` first with `ruleSelector: ["all"]` and `target` set to the source path
2. If MCP tool is unavailable, fall back to `sf scanner:run --target <path> --format json`
3. If neither is available, run regex-based checks and note reduced coverage in the report

For each violation found, record:
- File path and line number (when available)
- Rule name / check name
- Severity (CRITICAL / HIGH / MEDIUM)
- Remediation guidance

### Step 3: Architecture Pattern Checks

Run when scope includes `architecture` or `full`.

| Check | Method | Pass Criteria | Severity |
|---|---|---|---|
| Trigger handler pattern | Scan `.trigger` files: body should be <= 5 lines and delegate to a handler class; verify handler class exists | All triggers follow handler pattern | HIGH |
| Wire service preference | Scan LWC `.js` files for imperative Apex calls (`import <method> from '@salesforce/apex/'` used with direct invocation rather than `@wire`) | Document any imperative calls; flag if no justification comment | MEDIUM |
| Sharing model declared | Scan `.cls` files for `with sharing` or `without sharing` keyword | All Apex classes have explicit sharing declaration | HIGH |
| Flow fault paths | Query FlowDefinition metadata for active flows; check XML for DML/callout elements without fault connectors | All DML/callout elements have fault connectors | HIGH |
| Recursion guards | Scan trigger handler classes for static boolean pattern (e.g., `static Boolean hasRun`) | All trigger handlers have recursion protection | HIGH |
| USER_MODE enforcement | Scan SOQL statements for `WITH USER_MODE` or `WITH SYSTEM_MODE`; flag any without explicit mode | All data access uses USER_MODE or has justified SYSTEM_MODE | MEDIUM |

For trigger handler pattern check:
1. Find all `.trigger` files
2. Count non-comment, non-blank lines in trigger body
3. If > 5 lines, flag as violation
4. Check that the trigger body references a handler class
5. Verify the handler class file exists

For sharing model check:
1. Scan all `.cls` files
2. Skip test classes (`@IsTest` annotation)
3. Check class declaration line for `with sharing` or `without sharing`
4. Flag any class missing explicit sharing declaration
5. Flag `without sharing` as a warning requiring documented justification

### Step 4: Documentation Checks

Run when scope includes `docs` or `full`.

| Check | Method | Pass Criteria | Severity |
|---|---|---|---|
| Custom field descriptions | Query `FieldDefinition` for custom fields where `Description` is null (org only) | All custom fields have Description populated | MEDIUM |
| Solution design exists | Check `docs/` directory for `solution-design*.md` | File present | HIGH |
| Admin guide exists | Check `docs/` directory for `admin-guide*.md` | File present | HIGH |
| Deployment runbook exists | Check `docs/` directory for `deployment-runbook*.md` or `deploy-runbook*.md` | File present | HIGH |
| CLAUDE.md current | Check project root for `CLAUDE.md` with test commands section | CLAUDE.md exists and contains test command documentation | MEDIUM |
| Data dictionary exists | Check `docs/` directory for `data-dictionary*.md` or `data-model*.md` | File present | MEDIUM |

For org-based checks (custom field descriptions):
- Use `mcp__sf-tsmz__salesforce_query_records` to query FieldDefinition
- Query: `SELECT EntityDefinition.QualifiedApiName, QualifiedApiName, Description FROM FieldDefinition WHERE EntityDefinition.QualifiedApiName IN (<custom-objects>) AND Description = null`
- Group results by object for readability

For file-based checks:
- Use glob patterns to find matching files
- Check file is non-empty (> 0 bytes)
- For CLAUDE.md, verify it contains a test commands section (search for "test" in headings)

### Step 5: Output Generation

Compile all results into the compliance report.

**Severity classification:**
- **CRITICAL**: Must fix before delivery. Blocks compliance.
- **HIGH**: Should fix before delivery. Strongly recommended.
- **MEDIUM**: Should address. Non-blocking but noted in report.
- **WARNING**: Informational. No action required.

**Overall status logic:**
- PASS: Zero CRITICAL, zero HIGH issues
- FAIL: Any CRITICAL or HIGH issues present
- Include warning count in status line

Save the report to `docs/compliance-report-<YYYY-MM-DD>.md` and display in chat.

## Output Format

```markdown
# Compliance Check Report

| Field | Value |
|---|---|
| Project | <project-name> |
| Date | <YYYY-MM-DD> |
| Scope | <org-alias or source-path> |
| Check Categories | <code, architecture, docs, or all> |

---

### Overall Status: PASS / FAIL (with <N> warnings)

---

### Code Quality

| # | Check | Status | Details |
|---|-------|--------|---------|
| CQ-1 | No SOQL in loops | PASS/FAIL | <file:line if failed> |
| CQ-2 | No DML in loops | PASS/FAIL | <file:line if failed> |
| CQ-3 | No hardcoded IDs | PASS/FAIL | <count and locations if failed> |
| CQ-4 | Test coverage | PASS/FAIL | <overall %, per-class failures> |
| CQ-5 | No empty catch blocks | PASS/FAIL | <file:line if failed> |
| CQ-6 | Bind variables in SOQL | PASS/FAIL | <file:line if failed> |
| CQ-7 | No deprecated APIs | PASS/FAIL | <details if failed> |
| CQ-8 | Code Analyzer clean | PASS/FAIL | <Critical: N, High: N> |

### Architecture Patterns

| # | Check | Status | Details |
|---|-------|--------|---------|
| AP-1 | Trigger handler pattern | PASS/FAIL | <trigger names if failed> |
| AP-2 | Wire service preference | PASS/WARN | <components with imperative calls> |
| AP-3 | Sharing model declared | PASS/FAIL | <classes missing declaration> |
| AP-4 | Flow fault paths | PASS/FAIL | <flow names if failed> |
| AP-5 | Recursion guards | PASS/FAIL | <handler classes missing guards> |
| AP-6 | USER_MODE enforcement | PASS/WARN | <classes missing explicit mode> |

### Documentation

| # | Check | Status | Details |
|---|-------|--------|---------|
| DC-1 | Custom field descriptions | PASS/FAIL | <N fields missing descriptions> |
| DC-2 | Solution design | PASS/FAIL | <path or "missing"> |
| DC-3 | Admin guide | PASS/FAIL | <path or "missing"> |
| DC-4 | Deployment runbook | PASS/FAIL | <path or "missing"> |
| DC-5 | CLAUDE.md current | PASS/FAIL | <details> |
| DC-6 | Data dictionary | PASS/WARN | <path or "missing"> |

---

### Remediation Required

Items that must be addressed before delivery, ordered by severity:

1. **[CRITICAL]** <CQ-N> <description> — <specific fix instructions>
2. **[HIGH]** <AP-N> <description> — <specific fix instructions>
...

### Warnings (non-blocking)

1. <AP-2> <description> — <recommendation>
...

---

### Compliance Summary

| Metric | Value |
|---|---|
| Checks Passed | <N> / <total> |
| Critical Issues | <N> |
| High Issues | <N> |
| Medium Issues | <N> |
| Warnings | <N> |
| **Recommendation** | **READY FOR DELIVERY** / **NEEDS REMEDIATION** |
```
