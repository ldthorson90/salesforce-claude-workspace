---
name: sf-demo-script
description: Generate click-by-click demo scripts for client walkthroughs with persona-based talking points, step-by-step navigation, and edge case callouts.
user-invocable: true
---

# Salesforce Demo Script Generator

Generate a complete, click-by-click demo script for client walkthroughs. Scripts are persona-based with exact navigation steps, talking points that connect features to business value, expected screen behavior, and edge case recovery guidance.

## Arguments

- `<feature>` — the Salesforce feature or capability being demoed (e.g., "opportunity management", "case deflection", "Einstein lead scoring")
- `<persona>` — target audience role (e.g., "sales rep", "service agent", "sales manager", "admin", "executive sponsor")
- `<org-alias>` — org alias to inspect for real config context (optional but recommended)
- `--scenarios <n>` — number of key scenarios to cover (default: 3)

## Workflow

### Step 1: Gather Context

If arguments are not fully provided, ask:

```
What feature are we demoing?
Who is the audience (role/persona)?
Do you have an org alias for live config? (optional)
How many scenarios? (default: 3)
What is the primary business goal this demo should achieve?
```

### Step 2: Inspect Org Configuration (if org-alias provided)

Confirm auth:
```bash
sf org display -o <org-alias> --json
```

If auth fails, warn: "Org auth failed — generating script without live config. Review steps against your actual setup."

Pull relevant config based on the feature being demoed. Examples:

**For Sales features:**
```bash
sf data query -q "SELECT MasterLabel FROM OpportunityStage WHERE IsActive = true ORDER BY SortOrder" -o <org-alias>
sf data query -q "SELECT Name FROM RecordType WHERE SObjectType = 'Opportunity' AND IsActive = true" -o <org-alias>
```

**For Service features:**
```bash
sf data query -q "SELECT MasterLabel FROM CaseStatus WHERE IsDefault = false ORDER BY SortOrder" -o <org-alias>
sf data query -q "SELECT Name FROM RecordType WHERE SObjectType = 'Case' AND IsActive = true" -o <org-alias>
```

**For general:**
```bash
sf data query -q "SELECT Label FROM TabDefinition WHERE IsCustom = false" -o <org-alias>
```

Use real values from org in the generated steps (actual stage names, record type names, etc.).

### Step 3: Generate Pre-Demo Setup Checklist

Output a checklist the presenter must complete before the demo:

```markdown
## Pre-Demo Setup Checklist

### Sample Data
- [ ] Create/confirm demo Account: "<CompanyName>" (Industry: <relevant>)
- [ ] Create/confirm demo Contact: "<Name>" linked to demo account
- [ ] Create/confirm demo <object>: "<RecordName>" in <stage/status>
- [ ] <feature-specific records>

### Configuration Verification
- [ ] Log into org as <persona's profile> (not as Admin)
- [ ] Confirm <key feature> is enabled: Setup > <path>
- [ ] Verify home page layout shows <relevant components>
- [ ] Clear browser cache / open incognito window

### Browser State
- [ ] Bookmark: <org URL>/lightning/page/home
- [ ] Have backup tab open to: <fallback URL>
- [ ] Screen resolution: 1920x1080 recommended
- [ ] Close email/Slack notifications
```

### Step 4: Generate Step-by-Step Walkthrough

For each scenario (1 through `<n>`), produce:

```markdown
## Scenario <N>: <Scenario Title>

**Business Objective:** <What business problem this scenario demonstrates solving>
**Duration:** ~<X> minutes
**Starting Point:** <org URL>/lightning/page/home (logged in as <persona>)

---

### Step 1: <Action Title>
**Click:** App Launcher (waffle icon, top-left) > type "<App Name>" > select "<App Name>"
**Say:** "As a <persona>, your first stop every morning is the <App Name> app. Notice how the home page surfaces exactly what <persona role> needs — <specific component> shows your pipeline at a glance."
**Expected:** Home page loads with <App Name> branding. <Component> is visible in <position> of the page.
**[SCREENSHOT: App Launcher showing <App Name> selected, home page loaded]**

---

### Step 2: <Action Title>
**Click:** <Tab Name> in top navigation > <Button or link>
**Say:** "<Talking point connecting action to business value>"
**Expected:** <Exactly what the screen shows — list name, record count, visible fields>
**[SCREENSHOT: <description>]**

---

### Step N: <Action Title>
...

### Scenario Wrap-Up
**Say:** "In just <X> minutes, <persona> was able to <summarize what was accomplished>. That's <business outcome — time saved, visibility gained, errors prevented>."
```

Generate `<n>` complete scenarios with 4–8 steps each, using real config values pulled from the org.

### Step 5: Edge Cases Reference

```markdown
## Edge Cases & Recovery

| Situation | What You'll See | Recovery |
|-----------|----------------|----------|
| Slow page load | Spinner on <component> | Say "While this loads, let me highlight..." — switch to backup tab |
| Record not found | Empty list view | Apply filter: <field> = <value>; or navigate directly: <URL> |
| Permission error | "Insufficient privileges" toast | Switch back to admin user, explain this is a profile setting |
| Feature not visible | Missing tab/button | Navigate via URL: <direct path>; note it requires <permission/setting> |
| <feature-specific issue> | <symptom> | <recovery> |
```

### Step 6: Closing Talking Points

```markdown
## Closing — Value Summary

**Say:**
"What we just saw covers three things your team needs to <business goal>:
1. **<Capability 1>** — so that <business outcome>
2. **<Capability 2>** — eliminating <pain point>
3. **<Capability 3>** — giving <persona> <benefit>

The next step is <call to action: sandbox access / pilot scope / technical discovery>. I can have a sandbox configured for your team to explore within <timeframe>."

## Questions to Anticipate
- "How long does implementation take?" → "For this scope, typically <range>. We'd confirm in discovery."
- "What does it cost?" → "Licensing depends on your current edition. Let's set up a scoping call."
- "Can we see <other feature>?" → "Absolutely — I can build that into a follow-up demo."
```

### Step 7: Output

Write the complete script to: `demo-script-<feature>-<persona>.md` in the current directory.

Print confirmation:
```
Demo script generated: demo-script-<feature>-<persona>.md
Scenarios: <n>
Estimated demo duration: ~<X> minutes
Setup checklist: <count> items

To convert to Word doc for client delivery, run: /docx demo-script-<feature>-<persona>.md
```

## Output

A structured markdown file with sections:
1. **Pre-Demo Setup Checklist** — data, config, browser prep
2. **Scenario 1..N** — step-by-step with click targets, talking points, expected behavior, screenshot placeholders
3. **Edge Cases Reference** — recovery table for common issues
4. **Closing** — value summary, call to action, anticipated questions

Optionally convert to `.docx` using the `/docx` skill on the output file.
