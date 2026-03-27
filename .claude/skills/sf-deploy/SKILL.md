---
name: sf-deploy
description: Deploy source to a Salesforce org with pre-flight safety checks, code analysis gate, test execution, and deployment report.
user-invocable: true
---

# Salesforce Deploy

Deploy source to a Salesforce org with mandatory safety checks.

## Arguments

- `<target-org>` — org alias to deploy to
- Optional: `--validate-only` (check-only deploy, no actual changes)
- Optional: `--test-level <level>` (RunLocalTests, RunSpecifiedTests, RunAllTestsInOrg)

## Workflow

### Step 1: Pre-flight — Production Blocker

```bash
sf org display -o <target-org> --json
```

Parse the JSON result:
- If `isSandbox` is false AND `isScratch` is false: **BLOCK the deploy entirely**
  - Print: "BLOCKED: Production org detected. Per consulting governance, AI-generated code must not deploy to production. Deploy to a sandbox first, then promote through your change management process."
  - **Stop. Do not continue.**
- If the command fails (auth error), tell the user to run `/sf-org-setup` first

### Step 2: Code Analysis Gate

```bash
sf scanner run --target force-app --format json --normalize-severity
```

Parse the JSON output. Count violations by severity.

- **If ANY Critical or High violations exist:**
  - Print a table: File | Rule | Severity | Message
  - Print: "Code Analysis found X Critical and Y High violations. Fix these before deploying."
  - **Stop. Do not continue.**
- **If only Medium/Low:** Print summary count and continue
- **If Code Analyzer not installed:** Warn but continue (don't block on missing tooling)

### Step 3: Deploy Source

If `--validate-only`:
```bash
sf project deploy start -o <target-org> --dry-run --wait 30 --json
```

Otherwise:
```bash
sf project deploy start -o <target-org> --wait 30 --json
```

Parse the result:
- If deploy fails: print component failures (component name, problem type, message)
- If deploy succeeds: continue to test step

### Step 4: Run Tests

```bash
sf apex run test -o <target-org> --test-level RunLocalTests --result-format json --wait 20
```

Parse results:
- Total tests, passing, failing
- Code coverage percentage (from `summary.testRunCoverage`)
- If any tests fail: print failure details (test name, method, message, stack trace)
- Coverage warnings:
  - < 75%: "CRITICAL: Below Salesforce minimum deployment threshold (75%)"
  - 75-84%: "WARNING: Below consultant delivery standard (85%)"
  - >= 85%: "Coverage meets delivery standard"

### Step 5: Git Tag (success only, not validate-only)

If deploy succeeded AND not `--validate-only` AND in a git repo:
```bash
git tag -a "deploy/<target-org>/$(date +%Y%m%d-%H%M%S)" -m "Deployed to <target-org>: X components, XX.X% coverage"
```

### Step 6: Deployment Report

```
=== Deployment Report ===
Org:         <target-org> (<sandbox/scratch>)
Mode:        Deploy / Validate-only
Status:      SUCCESS / FAILED
Code Analysis: X violations (0 Critical, 0 High, X Medium, Y Low)
Components:  X deployed
Tests:       X passed, Y failed
Coverage:    XX.X%
Git Tag:     deploy/<target-org>/YYYYMMDD-HHMMSS
Duration:    Xm Xs

Next steps:
- <if failed: list what to fix>
- <if success: "Ready for UAT" or "Promote to next environment">
```

## Hard Rules

- Production deploy is a HARD BLOCK. No override, no `--force`, no exceptions.
- Code Analysis gate is mandatory. Zero Critical/High tolerance.
- If `--validate-only`, skip git tag and note "validation only" in report.
- Always run tests unless the user explicitly specifies `--test-level NoTestRun` (and warn if they do).
