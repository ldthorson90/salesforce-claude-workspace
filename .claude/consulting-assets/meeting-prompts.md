# Meeting Prompts

Prompt templates for processing Salesforce consulting meeting types.
Each template covers pre-meeting preparation, during-meeting note structure,
and post-meeting extraction.

Ported from ConsultHub's meeting pipeline (steps: transcribe, summarize,
extract action items, extract learnings, classify meeting, extract
intelligence — decisions, scope flags, risks) and adapted for Salesforce
consulting contexts.

---

## Discovery Workshop

Initial requirements gathering session with client stakeholders.

### Pre-Meeting Preparation

```
Prepare a briefing for a Salesforce discovery workshop with [CLIENT_NAME].

Context:
- Industry: [INDUSTRY]
- Engagement type: [new_build | optimization | migration | integration]
- Attendees: [STAKEHOLDER_LIST with roles]
- Known information: [ANY PRIOR CONTEXT]

Generate:
1. A prioritized list of discovery questions tailored to this industry and
   engagement type (reference .claude/consulting-assets/discovery-questions.md)
2. Key risk patterns to probe for during the session
3. Relevant Salesforce architecture considerations for this industry
4. A proposed agenda with time allocations (90-minute session)
```

### During-Meeting Note Structure

```
## Discovery Workshop — [CLIENT_NAME]
**Date**: [DATE]  **Duration**: [DURATION]  **Attendees**: [NAMES]

### Business Context
- Primary business objectives:
- Key KPIs / success metrics:
- Stakeholder priorities (by person):

### Current State
- Existing systems:
- Data sources:
- Pain points (ranked by stakeholder emphasis):

### Requirements Captured
- Must-have (P0):
- Should-have (P1):
- Nice-to-have (P2):
- Explicitly out of scope:

### Constraints Identified
- Timeline:
- Budget:
- Technical:
- Organizational:

### Open Questions
- [Question — Owner — Due date]

### Risks Surfaced
- [Risk — Severity — Notes]
```

### Post-Meeting Summary Prompt

```
Analyze this Salesforce discovery workshop transcript and extract structured
findings. Return JSON with these sections:

{
  "summary": "3-5 sentence overview focusing on business objectives and key findings",
  "decisions": [
    {"decision": "What was agreed", "context": "Why", "owner": "Who"}
  ],
  "requirements": [
    {"title": "Short title", "description": "Detail", "priority": "P0|P1|P2", "category": "data|automation|ui|integration|security|reporting"}
  ],
  "scope_flags": [
    {"signal": "What expanded or shifted", "severity": "low|medium|high", "detail": "Explanation"}
  ],
  "risks": [
    {"risk": "Description", "category": "data|technical|organizational|timeline|scope", "severity": "low|medium|high"}
  ],
  "action_items": ["Action item with owner and due date"],
  "open_questions": ["Unanswered question with owner"],
  "stakeholder_insights": [
    {"name": "Person", "role": "Title", "priorities": "What they care about", "concerns": "What worries them"}
  ]
}
```

---

## Architecture Review

Technical design session covering data model, automation, integration, and security.

### Pre-Meeting Preparation

```
Prepare for a Salesforce architecture review session.

Context:
- Engagement: [ENGAGEMENT_TITLE]
- Review focus: [data_model | automation | integration | security | performance]
- Current design artifacts: [WHAT EXISTS]
- Known technical constraints: [CONSTRAINTS]

Generate:
1. Review checklist for the focus area(s)
2. Common anti-patterns to watch for
3. Salesforce governor limit considerations
4. Questions to validate architectural decisions
5. Reference to relevant architect decision guides
```

### During-Meeting Note Structure

```
## Architecture Review — [ENGAGEMENT_TITLE]
**Date**: [DATE]  **Focus**: [FOCUS_AREA]  **Attendees**: [NAMES]

### Design Decisions Made
- [Decision — Rationale — Alternatives considered]

### Technical Findings
- Governor limit concerns:
- Data model issues:
- Automation conflicts:
- Security gaps:
- Performance risks:

### Recommendations
- Critical (must address before build):
- Important (address during build):
- Advisory (consider for future):

### Open Technical Questions
- [Question — Needs investigation — Owner]
```

### Post-Meeting Summary Prompt

```
Analyze this Salesforce architecture review transcript. Extract:

{
  "summary": "3-5 sentence technical summary",
  "decisions": [
    {"decision": "Technical decision made", "context": "Rationale", "alternatives_considered": ["Alt 1", "Alt 2"], "owner": "Who"}
  ],
  "technical_risks": [
    {"risk": "Description", "category": "governor_limits|data_model|automation|integration|security|performance", "severity": "low|medium|high", "mitigation": "Proposed approach"}
  ],
  "action_items": ["Technical action with owner"],
  "architecture_notes": "Key design patterns, data model decisions, or automation approaches agreed upon",
  "open_questions": ["Unresolved technical question"]
}
```

---

## Sprint Demo

Iteration review showing completed work to client stakeholders.

### Pre-Meeting Preparation

```
Prepare for a sprint demo session.

Context:
- Sprint: [SPRINT_NAME] ([START_DATE] to [END_DATE])
- Completed stories: [LIST]
- Incomplete stories: [LIST with reasons]
- Known issues: [LIST]
- Demo environment: [SANDBOX_NAME]

Generate:
1. Demo script with logical flow and talking points
2. Backup plan if demo environment has issues
3. Anticipated client questions and prepared answers
4. Metrics to present (velocity, coverage, defects)
```

### During-Meeting Note Structure

```
## Sprint Demo — [SPRINT_NAME]
**Date**: [DATE]  **Attendees**: [NAMES]

### Demonstrated
- [Story — Demo outcome — Client reaction]

### Client Feedback
- Positive:
- Concerns:
- Change requests:

### Scope Changes Requested
- [Change — Impact assessment — Decision]

### Next Sprint Preview
- Planned stories:
- Dependencies:
- Risks:
```

### Post-Meeting Summary Prompt

```
Analyze this sprint demo transcript. Extract:

{
  "summary": "2-3 sentence sprint outcome summary",
  "demonstrated_items": [
    {"story": "Title", "outcome": "accepted|needs_revision|deferred", "feedback": "Client reaction"}
  ],
  "scope_flags": [
    {"signal": "New or changed requirement surfaced during demo", "severity": "low|medium|high", "detail": "Context"}
  ],
  "action_items": ["Action with owner and sprint target"],
  "client_satisfaction_signals": "Overall tone and satisfaction indicators",
  "risks": [
    {"risk": "Description", "category": "scope|timeline|technical", "severity": "low|medium|high"}
  ]
}
```

---

## UAT Session

User acceptance testing session with client end-users.

### Pre-Meeting Preparation

```
Prepare for a Salesforce UAT session.

Context:
- Features under test: [LIST]
- Test scripts prepared: [YES/NO — if yes, location]
- Test data seeded: [YES/NO — sandbox name]
- Known issues / workarounds: [LIST]
- UAT participants: [NAMES with roles]

Generate:
1. UAT facilitation guide (how to run the session)
2. Issue logging template for participants
3. Go/no-go criteria for each feature
4. Fallback plan for blocking defects discovered during session
```

### During-Meeting Note Structure

```
## UAT Session — [FEATURE_SET]
**Date**: [DATE]  **Environment**: [SANDBOX]  **Testers**: [NAMES]

### Test Results
- [Test case — Pass/Fail — Notes — Severity if failed]

### Defects Found
- [Defect — Severity — Steps to reproduce — Assigned to]

### User Feedback
- Usability concerns:
- Training needs identified:
- Process gaps (system works but process unclear):

### Sign-off Status
- [Feature — Signed off / Conditional / Blocked — Conditions]
```

### Post-Meeting Summary Prompt

```
Analyze this UAT session transcript. Extract:

{
  "summary": "2-3 sentence UAT outcome summary",
  "test_results": [
    {"feature": "Feature name", "status": "passed|failed|conditional", "issues": "Description if any"}
  ],
  "defects": [
    {"title": "Short description", "severity": "critical|high|medium|low", "steps_to_reproduce": "Brief steps", "assigned_to": "Owner"}
  ],
  "sign_off_status": {
    "features_passed": 0,
    "features_conditional": 0,
    "features_blocked": 0,
    "overall": "go|conditional_go|no_go"
  },
  "action_items": ["Fix or follow-up with owner and date"],
  "training_needs": ["Identified training gap"]
}
```

---

## Steering Committee

Executive status update for project sponsors and leadership.

### Pre-Meeting Preparation

```
Prepare a steering committee briefing.

Context:
- Engagement: [ENGAGEMENT_TITLE]
- Reporting period: [DATE_RANGE]
- Engagement health score: [OVERALL_SCORE from health-scoring.md dimensions]
- Key metrics: [BUDGET_BURN, TIMELINE_STATUS, RISK_COUNT]
- Escalations needed: [LIST]

Generate:
1. Executive summary (3-4 sentences, lead with outcomes)
2. Stoplight status for each health dimension (Green/Yellow/Red)
3. Top 3 accomplishments this period
4. Top 3 risks with recommended actions
5. Decisions needed from steering committee
6. Look-ahead for next period
```

### During-Meeting Note Structure

```
## Steering Committee — [ENGAGEMENT_TITLE]
**Date**: [DATE]  **Attendees**: [NAMES with titles]

### Decisions Made
- [Decision — Who decided — Impact]

### Escalations Raised
- [Issue — Resolution or next step]

### Stakeholder Sentiment
- Executive sponsor:
- Business owner:
- IT leadership:

### Direction Changes
- [Change — Rationale — Impact on scope/timeline/budget]
```

### Post-Meeting Summary Prompt

```
Analyze this steering committee transcript. Extract:

{
  "summary": "2-3 sentence executive summary of outcomes",
  "decisions": [
    {"decision": "What was decided", "context": "Why", "owner": "Who", "impact": "What changes"}
  ],
  "escalations": [
    {"issue": "What was escalated", "resolution": "How it was resolved or next step", "owner": "Who"}
  ],
  "direction_changes": [
    {"change": "What shifted", "impact_scope": "Description", "impact_timeline": "Description", "impact_budget": "Description"}
  ],
  "action_items": ["Action with owner and date"],
  "stakeholder_sentiment": "Overall executive confidence level and any concerns",
  "risks": [
    {"risk": "Description", "severity": "low|medium|high|critical", "mitigation": "Agreed approach"}
  ]
}
```

---

## Go-Live Readiness

Pre-deployment checkpoint to validate readiness for production cutover.

### Pre-Meeting Preparation

```
Prepare a go-live readiness assessment.

Context:
- Planned go-live date: [DATE]
- Deployment target org: [ORG_ALIAS]
- Outstanding defects: [COUNT by severity]
- Test coverage: [PERCENTAGE]
- Data migration status: [STATUS]
- Training completion: [PERCENTAGE]
- Rollback plan: [EXISTS / NEEDS_WORK]

Generate:
1. Go-live readiness checklist (technical, data, people, process)
2. Risk assessment for go-live date
3. Recommended hypercare plan
4. Communication plan for go-live (internal + client-facing)
5. Go/no-go criteria with current status for each
```

### During-Meeting Note Structure

```
## Go-Live Readiness — [ENGAGEMENT_TITLE]
**Date**: [DATE]  **Go-Live Target**: [TARGET_DATE]  **Attendees**: [NAMES]

### Readiness Checklist
- [ ] Code deployed to staging and validated
- [ ] Test coverage >= 85% with meaningful assertions
- [ ] Code Analyzer: zero Critical/High violations
- [ ] Data migration: trial run completed and validated
- [ ] User training: all user groups trained
- [ ] Rollback plan: documented and tested
- [ ] Support handoff: runbook and escalation path defined
- [ ] Monitoring: dashboards and alerts configured
- [ ] Communication: go-live announcement drafted

### Go/No-Go Decision
- Decision: [GO / NO-GO / CONDITIONAL GO]
- Conditions (if conditional):
- Postponement plan (if no-go):

### Open Blockers
- [Blocker — Owner — Resolution plan — Target date]
```

### Post-Meeting Summary Prompt

```
Analyze this go-live readiness meeting transcript. Extract:

{
  "summary": "2-3 sentence readiness assessment",
  "go_no_go_decision": "go|no_go|conditional_go",
  "conditions": ["Condition that must be met before go-live"],
  "blockers": [
    {"blocker": "Description", "owner": "Who", "resolution_plan": "How", "target_date": "When"}
  ],
  "readiness_scores": {
    "technical": "green|yellow|red",
    "data": "green|yellow|red",
    "people": "green|yellow|red",
    "process": "green|yellow|red"
  },
  "risks": [
    {"risk": "Go-live risk", "severity": "low|medium|high|critical", "mitigation": "Plan"}
  ],
  "action_items": ["Pre-go-live action with owner and date"],
  "hypercare_plan": "Summary of post-go-live support arrangements"
}
```
