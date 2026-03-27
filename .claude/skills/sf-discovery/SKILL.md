---
name: sf-discovery
description: Guided requirements gathering for Salesforce consulting engagements — structured stakeholder interviews, business process mapping, and requirements documentation.
user-invocable: true
---

# Salesforce Discovery & Requirements Gathering

Structured requirements gathering for consulting engagements. Walks through stakeholder mapping, business problem capture, current state documentation, and prioritized requirements output.

## Arguments

- `<engagement-type>` — one of: `new-build`, `optimization`, `migration`, `integration`, `support` (optional — will ask if not provided)
- `resume` — resume a paused discovery session from the last saved state in the project directory

## Workflow

### Step 1: Engagement Setup

If engagement type was not provided, ask:

> What type of engagement is this? (new-build, optimization, migration, integration, support)

Set the interview template based on engagement type:

| Type | Focus Areas |
|------|-------------|
| new-build | Full requirements: business process, data model, automation, UI, integrations, security |
| optimization | Current pain points, performance bottlenecks, technical debt, quick wins |
| migration | Source system inventory, data mapping, feature parity, cutover plan |
| integration | Systems landscape, data flow, API inventory, error handling, SLA requirements |
| support | SLA tiers, escalation paths, known issues backlog, monitoring gaps |

Check for `.claude/consulting-assets/discovery-questions.md` in the workspace. If it exists, load domain-specific question templates from it. If not, use the inline fallback questions in each step below.

### Step 2: Stakeholder Mapping

Ask the user to identify stakeholders. For each person, capture:

| Field | Description |
|-------|-------------|
| Name | Full name |
| Role | Job title |
| Department | Business unit or team |
| Influence | High / Medium / Low |
| Decision Authority | Final approver, Recommender, Informed, Consulted |

Classify each stakeholder into one of:
- **Executive Sponsor** — owns budget and strategic alignment
- **Business Owner** — owns the process being automated/changed
- **End User** — daily user of the system
- **Technical** — IT, developer, or architect stakeholder
- **Admin** — Salesforce administrator

If fewer than 2 stakeholders are identified, flag this as a risk: "Limited stakeholder input increases the chance of missed requirements."

### Step 3: Business Problem Capture

Ask these questions one at a time using conversational flow. Do not dump all questions at once.

**Core questions (all engagement types):**
1. What business problem are we solving? What triggered this project?
2. What is the current process? (Walk me through it step by step.)
3. What are the top 3 pain points with the current process?
4. What does success look like? How will you know this project worked?
5. What KPIs or metrics matter? (e.g., case resolution time, lead conversion rate, data accuracy)

**Additional questions by engagement type:**

For **new-build**:
- What systems exist today that touch this process?
- Are there any compliance or regulatory requirements?
- What is the expected data volume (records per day/month)?

For **optimization**:
- What is working well that we should preserve?
- Where do users work around the system instead of using it as designed?
- What reports or dashboards are missing?

For **migration**:
- What is the source system? What version?
- What data must be migrated vs. archived vs. abandoned?
- What is the cutover timeline and downtime tolerance?

For **integration**:
- What systems need to exchange data with Salesforce?
- What direction is the data flow? (inbound, outbound, bidirectional)
- What is the frequency? (real-time, near-real-time, batch, on-demand)
- What happens when the integration fails? What is the retry/error strategy?

For **support**:
- What is the current SLA structure?
- What are the top 5 recurring support issues?
- Who handles Salesforce administration today?

### Step 4: Current State Documentation

**If an org is connected:**
- Suggest running `/sf-org-analyze` to get a metadata inventory of the current org
- Summarize: custom objects, fields, automations (flows/triggers), integrations, installed packages
- Note any technical debt or anti-patterns identified

**If no org is connected:**
- Ask the user to describe:
  - Current systems in use (CRM, ERP, spreadsheets, manual processes)
  - Current data sources and where data lives
  - Existing integrations between systems
  - Known customizations or tribal knowledge

Document the current state as a narrative, not just a list. Capture what works, what doesn't, and why.

### Step 5: Requirements Gathering

Iterate through domain-specific questions based on engagement type. For each requirement captured:

1. **ID** — sequential (REQ-001, REQ-002, ...)
2. **Description** — clear, testable statement of what the system should do
3. **Priority** — use MoSCoW: Must Have / Should Have / Could Have / Won't Have
4. **Acceptance Criteria** — how do we know this requirement is met?
5. **Salesforce Feature Mapping** — tag with the likely SF feature:
   - Validation Rule, Formula Field, Flow, Apex Trigger, LWC, Report/Dashboard
   - Approval Process, Permission Set, Sharing Rule, Integration, Custom Metadata
   - Experience Cloud, Knowledge, Omni-Channel, CPQ, Data Cloud, etc.
6. **Complexity** — Low / Medium / High / Very High

Ask clarifying questions when requirements are vague. Push back on "the system should be easy to use" — ask what "easy" means in measurable terms.

Flag requirements that conflict with each other or with Salesforce governor limits.

### Step 6: Constraint & Risk Identification

Capture constraints across these dimensions:

| Dimension | Questions |
|-----------|-----------|
| Timeline | Go-live date? Hard deadline or flexible? Phased rollout? |
| Budget | Fixed budget? Per-phase budget? License budget vs. implementation budget? |
| Compliance | HIPAA, SOX, GDPR, FedRAMP, PCI-DSS, industry-specific? |
| Resources | Internal team availability? Admin capacity post-go-live? |
| Technical | Data volume limits? API call limits? Existing technical debt? |
| Change Management | User adoption concerns? Training requirements? |

Check for `.claude/consulting-assets/risk-patterns.md` in the workspace. If it exists, cross-reference identified constraints against common risk patterns. If not, use these common risks as a checklist:

- Scope creep due to unclear requirements
- Data quality issues blocking migration
- Integration complexity underestimated
- Insufficient test data or test environment
- Key stakeholder unavailable during critical phases
- Governor limit violations at scale
- User adoption failure due to inadequate training
- Go-live timeline compressed by upstream delays

Assign each identified risk a severity: **Critical / High / Medium / Low**.

### Step 7: Output Generation

Generate the discovery report in markdown format (see Output Format below).

Ask the user where to save:
1. **Project directory** — save as `discovery-report-<date>.md` in the current project
2. **Obsidian vault** — save to the Obsidian vault under a client folder
3. **Clipboard only** — just display the output

If saving, confirm the file path before writing.

## Output Format

```markdown
# Discovery Report

## Client: <name>
## Engagement Type: <type>
## Date: <YYYY-MM-DD>
## Consultant: <name if provided>

---

### 1. Stakeholder Map

| Name | Role | Department | Type | Influence | Decision Authority |
|------|------|------------|------|-----------|-------------------|

### 2. Business Problem Statement

<Narrative summary of the business problem, current pain points, and desired outcomes.>

**Success Criteria:**
- <measurable outcome 1>
- <measurable outcome 2>

**Key Metrics:**
- <KPI 1>
- <KPI 2>

### 3. Current State

<Narrative description of current systems, processes, and known issues.>

**Systems Inventory:**
| System | Purpose | Data Owned | Integration |
|--------|---------|------------|-------------|

**Current Automations:**
- <list of existing automations, workflows, or manual processes>

### 4. Requirements

| ID | Requirement | Priority | Acceptance Criteria | SF Feature | Complexity |
|----|-------------|----------|---------------------|------------|------------|

### 5. Constraints

| Dimension | Constraint | Impact |
|-----------|-----------|--------|

### 6. Risks

| ID | Risk | Severity | Likelihood | Mitigation |
|----|------|----------|------------|------------|

### 7. Open Questions

- <question 1> — Owner: <who needs to answer>
- <question 2> — Owner: <who needs to answer>

### 8. Next Steps

- [ ] <action item 1> — Owner: <name> — Due: <date>
- [ ] <action item 2> — Owner: <name> — Due: <date>
```

## Important

- Ask questions conversationally, not as a wall of text. One topic at a time.
- Push back on vague requirements. "Make it user-friendly" is not a requirement.
- Flag conflicts between requirements early (e.g., "real-time sync" vs. "minimal API usage").
- Reference the architect decision guides via `/sf-decide` when a requirement implies an architecture choice.
- This skill gathers information only. It does not generate code, metadata, or deployable artifacts.
- If the user says `resume`, look for the most recent partial discovery report in the project directory and continue from the last completed section.
