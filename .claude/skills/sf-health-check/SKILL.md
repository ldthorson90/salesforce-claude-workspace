---
name: sf-health-check
description: Proactive org health monitoring — checks governor limits, metadata hygiene, automation health, security posture, data quality, and technical debt. Outputs a scored Green/Yellow/Red scorecard with prioritized recommendations.
user-invocable: true
---

# Salesforce Org Health Check

Run a multi-dimensional health audit on a Salesforce org. Produces a scored scorecard with Green/Yellow/Red status per category, a prioritized issues list, and recommendations. Supports trend tracking across runs.

## Arguments

- `<org-alias>` — target org to analyze
- `--category <name>` — run only one category: `governor` | `metadata` | `automation` | `security` | `data` | `debt`
- `--save` — save scorecard to `.claude/health-reports/<org-alias>-<YYYY-MM-DD>.md` for trend tracking

## Workflow

### Step 1: Pre-flight

```bash
sf org display -o <org-alias> --json
```

Parse result:
- If auth fails: print "Org auth failed. Run `sf org login web -a <org-alias>` first." and stop.
- If `isSandbox: false` and `isScratch: false`: print "⚠ Production org detected — running in read-only analysis mode. No changes will be made." Continue.
- Record: `orgName`, `instanceUrl`, `orgType`

---

### Step 2: Governor Limits

```bash
sf limits api display -o <org-alias> --json
```

Also query async job queue:
```bash
sf data query -q "SELECT Status, COUNT(Id) ct FROM AsyncApexJob WHERE Status IN ('Queued','Holding','Failed') AND CreatedDate = LAST_N_DAYS:7 GROUP BY Status" -o <org-alias>
```

**Thresholds:**
| Limit | Yellow | Red |
|-------|--------|-----|
| API calls (daily) | > 70% used | > 90% used |
| Data storage | > 70% used | > 85% used |
| File storage | > 70% used | > 85% used |
| Async queue (Failed jobs, 7d) | > 10 | > 50 |

Score: start at 100, subtract 10 per Yellow issue, 25 per Red issue.

---

### Step 3: Metadata Hygiene

```bash
sf data query -q "SELECT Label, Status FROM Flow WHERE Status = 'Obsolete' LIMIT 50" -o <org-alias>
sf data query -q "SELECT Name FROM PermissionSet WHERE IsCustom = true AND Id NOT IN (SELECT PermissionSetId FROM PermissionSetAssignment)" -o <org-alias>
sf data query -q "SELECT EntityDefinition.QualifiedApiName, DeveloperName, DataType FROM FieldDefinition WHERE EntityDefinition.IsCustomizable = true AND LastModifiedDate < LAST_N_YEARS:2 LIMIT 100" -o <org-alias>
```

**Thresholds:**
| Issue | Yellow | Red |
|-------|--------|-----|
| Obsolete flows | > 5 | > 20 |
| Unassigned custom permission sets | > 3 | > 10 |
| Stale custom fields (2+ years, unused) | > 20 | > 50 |

Score: start at 100, subtract per threshold.

---

### Step 4: Automation Health

```bash
sf data query -q "SELECT TableEnumOrId, COUNT(Id) ct FROM ApexTrigger WHERE Status = 'Active' GROUP BY TableEnumOrId HAVING COUNT(Id) > 0" -o <org-alias>
sf data query -q "SELECT TriggerType, ProcessType, Object, Label FROM Flow WHERE Status = 'Active' AND ProcessType IN ('AutoLaunchedFlow','Workflow') LIMIT 100" -o <org-alias>
sf data query -q "SELECT Label, NextFireTime FROM FlowSchedule WHERE IsDeleted = false" -o <org-alias>
```

Cross-reference objects with both active Apex triggers AND active record-triggered flows — flag as Yellow (potential conflict).

For trigger handler pattern check — scan trigger files for recursion guard:
```bash
sf data query -q "SELECT Name, Body FROM ApexTrigger WHERE Status = 'Active'" -o <org-alias>
```
Flag triggers whose Body does not delegate to a handler class (contains DML/SOQL directly) as Red.

**Thresholds:**
| Issue | Yellow | Red |
|-------|--------|-----|
| Objects with trigger + flow on same event | any | > 3 objects |
| Triggers without handler delegation | > 0 | > 3 |
| Scheduled flows with no NextFireTime | > 2 | — |

---

### Step 5: Security Posture

```bash
sf data query -q "SELECT QualifiedApiName, ExternalSharingModel, InternalSharingModel FROM EntityDefinition WHERE IsCustomizable = true AND InternalSharingModel = 'ReadWrite' LIMIT 50" -o <org-alias>
sf data query -q "SELECT Profile.Name FROM PermissionSet WHERE IsOwnedByProfile = true AND PermissionsModifyAllData = true" -o <org-alias>
sf data query -q "SELECT Name, UserType FROM Profile WHERE UserType = 'Guest'" -o <org-alias>
```

**Thresholds:**
| Issue | Yellow | Red |
|-------|--------|-----|
| Objects with OWD = Public Read/Write | > 2 | > 5 |
| Profiles with Modify All Data | > 2 (excl. System Admin) | — |
| Guest user profile exists | exists | has object permissions |

---

### Step 6: Data Quality

```bash
sf data query -q "SELECT COUNT(Id) total, COUNT(Email) with_email FROM Lead WHERE IsConverted = false AND CreatedDate = LAST_N_DAYS:90" -o <org-alias>
sf data query -q "SELECT COUNT(Id) total, COUNT(AccountId) with_account FROM Contact" -o <org-alias>
sf data query -q "SELECT Name FROM DuplicateRule WHERE IsActive = true" -o <org-alias>
```

**Thresholds:**
| Issue | Yellow | Red |
|-------|--------|-----|
| Leads missing Email (%) | > 20% | > 50% |
| Contacts missing Account | > 15% | > 40% |
| No active duplicate rules for Account/Contact/Lead | warning | — |

---

### Step 7: Technical Debt

```bash
sf data query -q "SELECT ApexClassOrTrigger.Name, NumLinesCovered, NumLinesUncovered FROM ApexCodeCoverageAggregate ORDER BY NumLinesUncovered DESC LIMIT 50" -o <org-alias>
sf data query -q "SELECT Name, ApiVersion FROM ApexClass WHERE ApiVersion < 50.0 AND Status = 'Active' LIMIT 50" -o <org-alias>
sf data query -q "SELECT Name, IsActive FROM WorkflowRule WHERE IsActive = true" -o <org-alias>
sf data query -q "SELECT Name FROM Flow WHERE Status = 'Active' AND ProcessType = 'Workflow'" -o <org-alias>
```

Calculate coverage %: `NumLinesCovered / (NumLinesCovered + NumLinesUncovered) * 100`

**Thresholds:**
| Issue | Yellow | Red |
|-------|--------|-----|
| Apex classes < 75% coverage | > 0 | any class < 50% |
| Apex on API version < 50.0 (Spring '20) | > 5 classes | > 20 classes |
| Active Workflow Rules (deprecated) | > 0 | > 10 |
| Active Process Builders (deprecated) | > 0 | > 5 |

---

### Step 8: Output Scorecard

```
# Org Health Report: <org-alias>
**Org:** <orgName> | <instanceUrl>
**Date:** <timestamp>
**Type:** <Sandbox / Scratch / Production>

## Overall Score: <avg>/100

| Category          | Status | Score | Issues |
|-------------------|--------|-------|--------|
| Governor Limits   | 🟢/🟡/🔴 | XX   | N      |
| Metadata Hygiene  | 🟢/🟡/🔴 | XX   | N      |
| Automation Health | 🟢/🟡/🔴 | XX   | N      |
| Security Posture  | 🟢/🟡/🔴 | XX   | N      |
| Data Quality      | 🟢/🟡/🔴 | XX   | N      |
| Technical Debt    | 🟢/🟡/🔴 | XX   | N      |

## 🔴 Critical Issues (fix before next delivery)
- **[Category]** <description> — Recommended action: <action>

## 🟡 Warnings (address within sprint)
- **[Category]** <description> — Recommended action: <action>

## Recommendations (Priority Order)
1. <Highest impact item>
2. ...

## Trend
<If --save and prior report exists: "vs <prior date>: Score improved/degraded by X points. New issues: N. Resolved: N.">
<If first run: "No prior report found — this is the baseline.">
```

**Status thresholds:** Score ≥ 80 = 🟢, 60–79 = 🟡, < 60 = 🔴

If `--save`: write report to `.claude/health-reports/<org-alias>-<YYYY-MM-DD>.md`. Check for previous reports in that directory and include trend comparison.

## Output

Scorecard printed to console. With `--save`, also written to `.claude/health-reports/` for trend tracking across runs.
