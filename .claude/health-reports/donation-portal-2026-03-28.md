# Org Health Report: donation-portal
**Org:** ldthorson90.3ad205bb04f5@agentforce.com | https://orgfarm-c11b456be5-dev-ed.develop.my.salesforce.com
**Date:** 2026-03-28
**Type:** Developer Edition (OrgFarm)
**API Version:** 66.0

---

## Overall Score: 94/100 🟢

| Category          | Status | Score | Issues |
|-------------------|--------|-------|--------|
| Governor Limits   | 🟢     | 100   | 0      |
| Metadata Hygiene  | 🟡     | 75    | 1      |
| Automation Health | 🟢     | 100   | 0      |
| Security Posture  | 🟢     | 100   | 0      |
| Data Quality      | 🟢     | 100   | 0      |
| Technical Debt    | 🟢     | 90    | 1      |

---

## 🔴 Critical Issues (fix before next delivery)

None.

---

## 🟡 Warnings (address within sprint)

- **[Metadata Hygiene]** 25 unassigned custom permission sets found. Many appear to be managed-package sets (SubscriptionManagement, Commerce_Shopper, Enablement, Copilot, Agentforce). Review and either assign to users/groups or document as intentionally unassigned managed package dependencies.
  - **Recommended action:** Run `SELECT Name, Label FROM PermissionSet WHERE IsCustom = true AND Id NOT IN (SELECT PermissionSetId FROM PermissionSetAssignment) AND NamespacePrefix = null` to isolate org-custom (non-managed) unassigned sets. Delete or assign as appropriate.

- **[Technical Debt]** No Apex test coverage data available — `ApexCodeCoverageAggregate` returned 0 records. This indicates tests have not been run in this org since last deployment.
  - **Recommended action:** Run `sf apex run test --test-level RunLocalTests -o donation-portal` to generate coverage data before next delivery. Verify all classes meet 75% threshold.

---

## Recommendations (Priority Order)

1. **Run Apex test suite** — Coverage data is absent. With 4 active Apex classes (DeveloperEditionUtils, PostInstallScript, and their test classes), a single test run will establish the baseline. Required before any deployment to production.

2. **Audit unassigned permission sets** — Filter out managed-package sets (those with a NamespacePrefix). For the remaining org-custom sets, either assign to the appropriate users or remove if stale.

3. **Monitor data storage** — Org is on a 5 MB data storage limit (Developer Edition). Currently at 0% usage, but this will become a constraint as donation data grows. Plan for a sandbox or scratch org strategy before scaling.

---

## Data Snapshot

| Metric | Value | Status |
|--------|-------|--------|
| Daily API Requests used | 1 / 15,000 (0.01%) | 🟢 |
| Data Storage | 0 / 5 MB (0%) | 🟢 |
| File Storage | 0 / 20 MB (0%) | 🟢 |
| Async Job Failures (7d) | 0 | 🟢 |
| Obsolete Flows | 0 | 🟢 |
| Unassigned Permission Sets | 25 | 🟡 |
| Active Triggers | 0 | 🟢 |
| Active Flows | 0 | 🟢 |
| Active Workflow Rules | 0 | 🟢 |
| Profiles with Modify All Data | 1 (System Administrator only) | 🟢 |
| Guest Profiles | 0 | 🟢 |
| OWD Public Read/Write Objects | 0 | 🟢 |
| Active Duplicate Rules | 3 (Account, Contact, Lead) | 🟢 |
| Leads missing Email (90d) | 0 / 22 (0%) | 🟢 |
| Contacts missing Account | 0 / 20 (0%) | 🟢 |
| Active Apex Classes | 4 (all API v64) | 🟢 |
| Classes on API < v50 | 0 | 🟢 |
| Apex Coverage Data | Not available | 🟡 |

---

## Trend
No prior report found — this is the baseline.
