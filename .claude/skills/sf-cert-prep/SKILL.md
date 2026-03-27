---
name: sf-cert-prep
description: Certification study mode with practice scenarios, exam simulation, and progress tracking.
user-invocable: true
---

# Salesforce Cert Prep

Interactive study sessions for Salesforce certification tracks — scenarios, walkthroughs, deep-dives, and full exam simulation.

## Arguments

- `<cert>` — certification track (see list below)
- `--mode <mode>` — study mode: `scenario` | `walkthrough` | `deep-dive` | `exam-sim` (default: `scenario`)
- `--topic <topic>` — focus on a specific topic within the cert
- `--questions <n>` — number of questions for `exam-sim` mode (default: 20)

## Supported Certifications

1. Administrator
2. Platform App Builder
3. Platform Developer I
4. Platform Developer II
5. Sales Cloud Consultant
6. Service Cloud Consultant
7. Data Cloud Consultant
8. AI Specialist (Agentforce)
9. CPQ Specialist

## Workflow

### Step 1: Cert Selection

If no `<cert>` argument is provided, display the 9 tracks numbered and prompt the user to select one before continuing.

### Step 2: Fetch Exam Guide

Search for the official exam guide:

```
WebSearch: site:trailhead.salesforce.com <cert> exam guide
```

Extract the exam domains and their percentage weights. This drives proportional question distribution in `exam-sim` and focus areas in `walkthrough`.

### Step 3: Run Study Mode

Execute the selected mode (see below).

### Step 4: Progress Log (offer after session)

After the session ends, offer to log progress to Obsidian:

```
~/Documents/Obsidian Vault/Cert Prep/<cert>/<YYYY-MM-DD>-<mode>.md
```

Include: topics covered, score (if `exam-sim`), weak domains, and recommended next focus.

### Step 5: Suggest Next Session

Based on weak domains or uncovered topics, recommend the focus area for the next study session.

---

## Study Modes

### scenario (default)

Use `qwen3:8b` via `ollama_chat` for scenario generation variety.

For each round:

1. Generate a realistic business scenario (3–5 sentences) relevant to the cert and any specified `--topic`
2. Ask: "Which approach would you recommend and why?"
3. Wait for the user's response
4. Provide the correct answer with full rationale
5. Reference authoritative documentation via Context7 (`/damecek/salesforce-documentation-context`) for backing

Repeat until the user ends the session or requests a different mode.

### walkthrough

Walk through a Salesforce architect decision guide or exam domain step by step.

For each decision point or concept:

- Explain the options available
- Cover trade-offs and when to use each
- Tie explicitly to exam domains and their percentage weights from the exam guide

Reference: Context7 with `/damecek/salesforce-documentation-context` for authoritative content.

If `--topic` is specified, scope the walkthrough to that topic. Otherwise, cover the highest-weighted domain first.

### deep-dive

Comprehensive exploration of a specific topic (e.g., "sharing model", "trigger framework", "Flow best practices").

Structure:
1. **What it is** — definition and core concepts
2. **When to use it** — use cases and decision criteria
3. **Limitations** — governor limits, platform constraints, edge cases
4. **Exam gotchas** — common trick questions and misconceptions
5. **Common mistakes** — patterns to avoid
6. **Practice questions** — 3–5 questions on the topic with answers and rationale

Pull authoritative docs via Context7 (`/damecek/salesforce-documentation-context`) throughout.

Requires `--topic` to be meaningful. If no topic is specified, prompt the user before starting.

### exam-sim

Full multiple-choice exam simulation.

1. Use `qwen3:8b` via `ollama_chat` to generate `<n>` questions (default: 20)
2. Questions must be A/B/C/D format, covering the cert's official exam domains proportionally by weight
3. Present one question at a time; accept the user's answer before proceeding
4. After all questions are answered:
   - Display score (X / N correct, percentage)
   - Break down performance by domain
   - Identify weak domains (< 70% correct in that domain)
   - Recommend specific study areas based on weak domains
5. Save results to Obsidian:
   ```
   ~/Documents/Obsidian Vault/Cert Prep/<cert>/<YYYY-MM-DD>-exam-sim.md
   ```
   Include: date, cert, score, per-domain breakdown, weak areas, recommended next focus.

---

## Hard Rules

- Never fabricate exam questions from memory alone — always generate via `qwen3:8b` or ground in fetched exam guide content.
- Always cite the official exam guide domain when explaining why an answer is correct.
- Progress logs go to Obsidian only after user confirmation — never write automatically.
- For `exam-sim`, do not reveal the answer until the user has responded to the question.
