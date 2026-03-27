---
name: sf-release-notes
description: Fetch Salesforce release notes, filter by client's active features using Ollama classification, and produce a targeted impact summary with breaking changes, new capabilities, and action items.
user-invocable: true
---

# Salesforce Release Notes Analyzer

Fetch and filter Salesforce release notes for a specific release. Uses Ollama (llama3.2) to classify notes by relevance to the client's active clouds and features. Produces a concise impact summary with breaking changes called out prominently.

**Cadence:** Run 3× per year around Salesforce releases (Spring, Summer, Winter). This is a manual skill — not scheduled.

## Arguments

- `<release>` — Salesforce release name (e.g., `"Spring 25"`, `"Summer 25"`, `"Winter 26"`)
- `<org-alias>` — org to check for active features (optional but recommended)
- `--client <name>` — filter for a specific client profile (reads their project CLAUDE.md for active clouds)

## Workflow

### Step 1: Fetch Release Notes

Search for the official release notes:

```
WebSearch: "Salesforce <release> release notes site:help.salesforce.com"
WebSearch: "Salesforce <release> release notes developer site:developer.salesforce.com"
```

Also check Context7:
```
context7 query: "Salesforce <release> release notes breaking changes deprecations"
library: /damecek/salesforce-documentation-context
```

Extract the full release notes content. If multiple pages exist (Admin, Developer, etc.), fetch all relevant sections.

### Step 2: Detect Active Features

**If `--client <name>` provided:**
- Find the client's project directory: search workspace for matching CLAUDE.md
- Read their CLAUDE.md to extract active cloud products and key features
- Note: use this as the primary filter

**If `<org-alias>` provided:**
```bash
sf package installed list -o <org-alias> --json
sf org display -o <org-alias> --json
```
Extract installed packages and org type to infer active features.

**Default feature set** (if neither provided):
- Sales Cloud, Service Cloud, Flow Builder, Apex, LWC, Reports & Dashboards, Sharing & Security

Build a feature list string: `"Sales Cloud, Service Cloud, Flow Builder, Apex triggers, LWC, SOQL, REST API"`

### Step 3: Classify with Ollama

Send the release notes content to llama3.2 for classification. Because content may be large, chunk into sections of ~2000 tokens each and classify per chunk.

For each chunk, use `ollama_chat` with `llama3.2`:

```
System: You are a Salesforce release notes analyst. Classify each release note item based on relevance to the client's active features.

User: Active features for this client: <feature-list>

For each item in the release notes below, classify as:
- RELEVANT: directly impacts one of the active features
- WATCH: might become relevant, worth monitoring
- IGNORE: unrelated product/feature not in active use
- BREAKING: deprecated, removed, behavior changed, or API version impacted — regardless of relevance, always flag these

Format your response as:
BREAKING: <item title> | <brief description of change and impact>
RELEVANT: <item title> | <brief description of why it matters>
WATCH: <item title> | <brief description>
IGNORE: <item title>

Release notes:
<chunk text>
```

Aggregate all classified items across chunks. De-duplicate.

### Step 4: Identify Breaking Changes and Deprecations

Regardless of Ollama classification, scan the raw text for these keywords and flag as BREAKING:
- `deprecated`, `deprecating`, `retiring`, `end of life`, `end-of-life`
- `removed`, `no longer supported`, `will not be supported`
- `replaced by`, `migration required`, `action required`
- `API version`, `minimum API version`
- `Summer '2X enforcement`, `Spring '2X enforcement` (delayed enforcement items)

Extract the surrounding context (± 2 sentences) for each match.

### Step 5: Generate Impact Summary

```markdown
# Salesforce <Release> — Impact Summary
**Client:** <name or "General">
**Active Features:** <comma-separated list>
**Generated:** <date>
**Release Notes Source:** <URL>

---

## ⚠️ Breaking Changes & Deprecations (Action Required)

> These items require review before the release reaches your org.

### <Item Title>
**Change:** <what is changing>
**Impact:** <how this affects the client's org/code>
**Action:** <what to do> by <deadline if known>
**Salesforce Docs:** <link if available>

---

## ✅ Relevant New Capabilities

### <Item Title>
**What's new:** <description>
**Opportunity:** <how the client could benefit or use this>
**Effort to adopt:** Low / Medium / High

---

## 👀 Worth Watching

- **<Item>:** <one-line description of why to monitor>

---

## Filtered Out
<N> items classified as IGNORE (unrelated products/features).

---

## Recommended Actions

| Priority | Action | Owner | Deadline |
|----------|--------|-------|----------|
| High | <breaking change action> | Dev/Admin | Before <release> goes live |
| Medium | <new feature evaluation> | Architect | <sprint> |
| Low | <watch item> | — | Monitor |

---

## Summary for Client Communication

> **<Release> brings X notable changes for your org.** [N breaking changes require attention before the release. N new capabilities are available immediately. N items are worth monitoring for future sprints.]
```

### Step 6: Save Output

If `--client <name>`:
- Write to `.claude/release-notes/<client>-<release>.md`
- Print: "Impact summary saved to .claude/release-notes/<client>-<release>.md"

Otherwise: print to console only.

Offer to draft a client-facing email summary using the output.

## Output

A structured impact summary with breaking changes, relevant new capabilities, and prioritized action items. Saved per-client if `--client` flag is used.

## Notes

- **Cadence:** Run 3× per year (Spring in Jan/Feb, Summer in May/Jun, Winter in Sep/Oct — before each release hits production orgs)
- **Not scheduled** — requires human judgment on breaking change actions before sharing with clients
- **Ollama dependency:** Requires `llama3.2` model running locally. If Ollama is unavailable, classify manually using the keyword scan in Step 4 only and note reduced accuracy
