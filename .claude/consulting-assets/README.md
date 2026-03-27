# Consulting Assets

Reusable consulting knowledge ported from ConsultHub and adapted for
Salesforce consulting engagements. These assets are referenced by skills
and used during client-facing work.

## Origin

These assets were extracted from the ConsultHub application's backend
services and domain models, then expanded with Salesforce platform-specific
knowledge. ConsultHub source modules:

- `backend/services/discovery_questions.py` — question generation by industry/engagement type
- `backend/models/discovery.py` — question bank schema (category, industry, effectiveness scoring)
- `backend/services/risk_tracker.py` — rule-based risk detection (timeline, communication, scope, resource)
- `backend/models/risk_pattern.py` — risk pattern schema (type, severity, source meetings)
- `backend/services/meeting_steps.py` — meeting pipeline (transcribe, summarize, extract actions/learnings, classify)
- `backend/services/intelligence_steps.py` — transcript intelligence extraction (decisions, scope flags, risks)
- `backend/services/health_scorer.py` — engagement health scoring (momentum, action completion, financial, scope, risk)
- `backend/services/weekly_digest.py` — periodic engagement digest (meetings, actions, time, risks)
- `backend/services/prospect_pipeline.py` — lead scoring factors
- `backend/services/sow_generator.py` — SOW assembly from engagement context
- `backend/services/context_manager.py` — context assembly pattern for LLM prompts

## Files

### `discovery-questions.md`
Structured interview questions organized by engagement type: New Build,
Optimization, Migration, Integration, and Support. Each section covers
business context, current state, requirements, constraints, and success
criteria. Use during initial client conversations and discovery workshops.

**Referenced by**: `/sf-discovery`, `/sf-handoff`, meeting prep prompts

### `risk-patterns.md`
20 common Salesforce consulting risk patterns with trigger signals, severity
classification, impact analysis, mitigation strategies, and platform-specific
notes. Categories: Technical, Organizational, Data, Timeline, Scope.

**Referenced by**: `/sf-org-analyze`, health scoring, meeting intelligence
extraction, steering committee prep

### `meeting-prompts.md`
Prompt templates for six consulting meeting types: Discovery Workshop,
Architecture Review, Sprint Demo, UAT Session, Steering Committee, and
Go-Live Readiness. Each template includes pre-meeting preparation,
during-meeting note structure, and post-meeting extraction prompts.

**Referenced by**: Meeting processing pipeline, `/sf-handoff`, calendar
integration workflows

### `health-scoring.md`
Six-dimension engagement health scoring framework: Scope, Timeline,
Technical, Relationship, Budget, and Risk. Each dimension includes
scoring criteria (0.0-1.0), data sources, early warning signals, and
recommended actions per Green/Yellow/Red tier. Includes a weekly health
check process and report template.

**Referenced by**: `/sf-playbook`, weekly digest, steering committee prep,
engagement dashboard

## How Skills Reference These Assets

Skills reference these assets via relative path from the workspace root:

```
.claude/consulting-assets/discovery-questions.md
.claude/consulting-assets/risk-patterns.md
.claude/consulting-assets/meeting-prompts.md
.claude/consulting-assets/health-scoring.md
```

Skills should read these files when they need consulting domain knowledge
rather than embedding that knowledge directly in the skill SKILL.md. This
keeps the knowledge centralized and maintainable.

## When to Use Each Asset

| Scenario | Asset |
|----------|-------|
| First call with a new client | discovery-questions.md |
| Scoping a new engagement | discovery-questions.md + risk-patterns.md |
| Preparing for any client meeting | meeting-prompts.md |
| Processing meeting notes/transcripts | meeting-prompts.md (post-meeting prompts) |
| Weekly engagement check-in | health-scoring.md |
| Steering committee preparation | health-scoring.md + meeting-prompts.md |
| Identifying project risks | risk-patterns.md |
| Go-live readiness assessment | meeting-prompts.md + risk-patterns.md |
| Client handoff documentation | All assets inform deliverable quality |
