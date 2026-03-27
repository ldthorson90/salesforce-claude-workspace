# Engagement Health Scoring

Health scoring dimensions for Salesforce consulting engagements. Each
dimension uses a 0.0-1.0 scale (0.0 = critical, 1.0 = healthy) with
Green/Yellow/Red thresholds.

Ported from ConsultHub's health scorer (dimensions: momentum, action
completion, financial, scope, risk; weighted composite scoring with
signal-level detail) and expanded for Salesforce-specific consulting.

---

## Scoring Overview

### Composite Health Score

The overall engagement health is a weighted composite of six dimensions:

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| Scope Health | 0.15 | Requirements stability and change request volume |
| Timeline Health | 0.20 | Milestone tracking and delivery velocity |
| Technical Health | 0.15 | Code quality, test coverage, governor limit margins |
| Relationship Health | 0.20 | Client responsiveness and stakeholder engagement |
| Budget Health | 0.15 | Burn rate and remaining budget vs. remaining work |
| Risk Health | 0.15 | Open risk count and unmitigated critical risks |

### Thresholds

| Score Range | Status | Meaning |
|-------------|--------|---------|
| 0.80 - 1.00 | Green | Healthy. Continue current trajectory. |
| 0.50 - 0.79 | Yellow | Caution. Address specific dimension(s) this week. |
| 0.00 - 0.49 | Red | At risk. Escalate to steering committee. Immediate action required. |

---

## Dimension 1: Scope Health

Measures requirements stability and whether the project scope is under control.

### Scoring Criteria

| Score | Condition |
|-------|-----------|
| 1.0 | No scope flags in last 5 meetings. Requirements stable. |
| 0.85 | 1 scope flag. Minor clarification, not a true expansion. |
| 0.70 | 2 scope flags. New requirements surfacing but manageable. |
| 0.55 | 3-4 scope flags. Pattern of scope expansion. Formal review needed. |
| 0.40 | 5+ scope flags. Significant scope drift. SOW amendment required. |
| 0.00 | Requirements fundamentally different from SOW. Project needs rescoping. |

**Formula**: `max(0.0, 1.0 - (scope_flag_count * 0.15))`

### Data Sources
- Meeting intelligence extraction (scope_flags from transcript analysis)
- Change request log (count and magnitude of changes)
- Requirements document version history (delta between versions)
- Sprint backlog churn (stories added/removed mid-sprint)

### Early Warning Signals
- Client mentions features not in the SOW during demos
- "Phase 2" items start appearing in sprint backlogs
- Sprint planning sessions frequently add unplanned work
- Requirements document has been revised 3+ times without SOW update

### Recommended Actions by Score Level

| Status | Actions |
|--------|---------|
| Green | Continue. Log scope decisions for audit trail. |
| Yellow | Schedule a scope review meeting. Document all changes formally. Update requirements document. Assess impact on timeline and budget. |
| Red | Escalate to steering committee. Prepare SOW amendment. Pause new development until scope is agreed. Present impact analysis (timeline, budget, resources). |

---

## Dimension 2: Timeline Health

Measures delivery velocity and milestone adherence.

### Scoring Criteria

| Score | Condition |
|-------|-----------|
| 1.0 | All milestones on track. Sprint velocity stable or improving. |
| 0.85 | One milestone at risk but recoverable within current sprint. |
| 0.70 | One milestone missed. Recovery plan in place. |
| 0.50 | Two milestones missed. Timeline compression required. |
| 0.30 | Three+ milestones missed. Go-live date at risk. |
| 0.00 | Project past end date with significant open work remaining. |

**Formula**: Composite of recency score (last meeting freshness) and cadence score (meeting regularity). `recency * 0.6 + cadence * 0.4`

- Recency: 1.0 if last meeting < 7 days ago; degrades to 0.0 at 30+ days
- Cadence: `min(1.0, meetings_in_30_days / 4.0)`

### Data Sources
- Sprint burndown charts and velocity trends
- Milestone completion dates vs. planned dates
- Meeting cadence (are regular meetings happening?)
- Action item aging (how long items stay open)

### Early Warning Signals
- Sprint velocity declining for 2+ consecutive sprints
- Daily standups being skipped or shortened
- "Carry-over" stories increasing sprint over sprint
- Dependencies on client deliverables (test data, access, decisions) are blocking

### Recommended Actions by Score Level

| Status | Actions |
|--------|---------|
| Green | Continue. Track velocity trends for early drift detection. |
| Yellow | Review sprint capacity and commitments. Identify and remove blockers. Assess whether scope reduction or timeline extension is needed. Communicate risk to client. |
| Red | Escalate to steering committee with options: (1) extend timeline, (2) reduce scope, (3) add resources. Present honest assessment with data. Do not commit to unrealistic recovery plans. |

---

## Dimension 3: Technical Health

Measures code quality, platform health, and technical risk indicators.

### Scoring Criteria

| Score | Condition |
|-------|-----------|
| 1.0 | Code Analyzer clean. Test coverage >85%. No governor limit warnings. Zero tech debt flagged. |
| 0.85 | Minor Code Analyzer warnings (Low severity). Coverage 75-85%. |
| 0.70 | Medium Code Analyzer findings. Coverage 65-75%. Some governor limit usage above 50%. |
| 0.50 | High Code Analyzer findings. Coverage below 65%. Governor limits frequently above 70%. |
| 0.30 | Critical Code Analyzer findings. Coverage below 50%. Governor limit failures in testing. |
| 0.00 | Production governor limit failures. Critical security vulnerabilities. Deployment blocked. |

### Data Sources
- Salesforce Code Analyzer results (Critical/High/Medium/Low violations)
- Apex test coverage percentage (org-wide and per-class)
- Governor limit consumption in test runs (% of limits used)
- Technical debt backlog (count and age of tech debt items)
- Deployment success/failure rate

### Early Warning Signals
- Test coverage declining with new deployments
- Code Analyzer findings increasing over time
- Developers working around governor limits instead of optimizing
- Sandbox environments diverging from production
- Deployment failures increasing in frequency

### Recommended Actions by Score Level

| Status | Actions |
|--------|---------|
| Green | Continue. Run Code Analyzer on every PR. Monitor test coverage trends. |
| Yellow | Dedicate 20% of sprint capacity to tech debt reduction. Review and fix Medium+ Code Analyzer findings. Refactor any code hitting >50% of governor limits. |
| Red | Stop feature development. Address all Critical/High findings. Refactor governor limit violations. Bring test coverage above 75%. Do not deploy to production until resolved. |

---

## Dimension 4: Relationship Health

Measures client engagement quality and stakeholder satisfaction.

### Scoring Criteria

| Score | Condition |
|-------|-----------|
| 1.0 | Regular meetings happening on schedule. All stakeholders engaged. Decisions made promptly. Positive feedback in demos. |
| 0.85 | Occasional scheduling conflicts. Minor delays in decisions (1-3 days). |
| 0.70 | Key stakeholder missing from recent meetings. Decisions taking 5+ days. Email response times increasing. |
| 0.50 | Communication gap of 14+ days. Executive sponsor disengaged. Multiple unanswered questions. |
| 0.30 | Client team unresponsive for 21+ days. No decision-maker available. Escalation attempts unanswered. |
| 0.00 | Complete communication breakdown. Engagement at risk of cancellation. |

### Data Sources
- Meeting attendance (who shows up, who is absent)
- Decision turnaround time (days from question to answer)
- Email/message response latency
- Stakeholder participation in demos and reviews
- Client NPS or satisfaction signals (tone, feedback quality)
- Action item completion by client-owned items

### Early Warning Signals
- Executive sponsor has not attended in 3+ meetings
- Client-assigned action items consistently overdue
- Meeting cancellations increasing
- Client team members expressing frustration or confusion
- Requests for "status reports" increasing (signals loss of visibility)

### Recommended Actions by Score Level

| Status | Actions |
|--------|---------|
| Green | Continue. Maintain regular touchpoints. Send proactive status updates. |
| Yellow | Schedule a dedicated check-in with the primary stakeholder. Address any open concerns directly. Increase communication frequency. Identify if there are internal client issues (reorg, budget review, competing priorities). |
| Red | Escalate through executive sponsor channel. Request an in-person or video meeting (not email). Present a clear picture of what is blocked and what is needed. Prepare a pause/pivot recommendation if engagement is no longer viable. |

---

## Dimension 5: Budget Health

Measures financial sustainability of the engagement.

### Scoring Criteria

| Score | Condition |
|-------|-----------|
| 1.0 | Burn rate on track. Invoice coverage matches progress. Payments current. |
| 0.85 | Slight overburn (<10% ahead of plan). All invoices paid. |
| 0.70 | Overburn 10-20% ahead of plan. One invoice overdue. |
| 0.50 | Overburn >20%. Multiple invoices overdue. Remaining budget tight for remaining work. |
| 0.30 | Budget nearly exhausted with significant work remaining. Payment issues. |
| 0.00 | Budget exceeded. Unpaid invoices >60 days. Project funding at risk. |

**Formula**: `invoice_coverage * 0.5 + payment_ratio * 0.5`

- Invoice coverage: `min(1.0, total_invoiced / estimated_revenue)`
- Payment ratio: `total_paid / total_invoiced` (1.0 if nothing invoiced)

### Data Sources
- Time entries: total hours, billable hours, billable ratio
- Invoice status: sent, paid, overdue
- Budget burn rate vs. project completion percentage
- Rate card utilization (actual vs. planned mix of resource levels)
- Change order revenue vs. additional scope absorbed

### Early Warning Signals
- Billable ratio drops below 60% of total tracked time
- Time-to-payment increasing (invoices taking longer to get paid)
- Burn rate accelerating while progress is linear
- Non-billable administrative tasks consuming increasing share of hours
- Client pushing back on invoice amounts or line items

### Recommended Actions by Score Level

| Status | Actions |
|--------|---------|
| Green | Continue. Review burn rate weekly. Ensure time entries are current. |
| Yellow | Analyze where non-billable time is going. Review rate card against actual resource utilization. Follow up on overdue invoices. Forecast remaining budget against remaining work and flag if insufficient. |
| Red | Escalate to engagement lead and client sponsor. Present budget analysis: spent vs. remaining vs. work remaining. Options: (1) additional funding via change order, (2) scope reduction, (3) resource optimization. Stop work on unbillable activities. |

---

## Dimension 6: Risk Health

Measures the volume and severity of open risks across the engagement.

### Scoring Criteria

| Score | Condition |
|-------|-----------|
| 1.0 | No open risks. No overdue action items. |
| 0.85 | 1-2 low-severity risks with mitigation plans. |
| 0.70 | 3-4 risks or 1 high-severity risk with mitigation. |
| 0.50 | 5+ risks or 1 unmitigated high-severity risk. Overdue actions accumulating. |
| 0.30 | Multiple high-severity risks without mitigation. Critical risk identified. |
| 0.00 | Critical unmitigated risk threatening project viability. |

**Formula**: `max(0.0, 1.0 - (risk_count * 0.15) - (overdue_action_count * 0.2))`

### Data Sources
- Risk register (risk-patterns.md categories and severities)
- Meeting intelligence extraction (risks surfaced in transcripts)
- Action item tracker (overdue count and aging)
- Automated risk detection signals:
  - Timeline risk: engagement past end date with open items
  - Communication gap: no meetings in 14+ days
  - Scope drift: 3+ scope flags in recent meetings
  - Resource constraint: billable ratio below 60%

### Early Warning Signals
- New risks appearing faster than existing risks are being resolved
- Risk mitigation actions consistently deferred to "next sprint"
- Same risk pattern appearing across multiple meetings
- Client raising concerns that map to known but unaddressed risks

### Recommended Actions by Score Level

| Status | Actions |
|--------|---------|
| Green | Continue. Review risk register weekly. Close resolved risks promptly. |
| Yellow | Prioritize risk mitigation in sprint planning. Assign owners to unowned risks. Escalate any risk that has been open >2 sprints without progress. Review risk patterns for systemic issues. |
| Red | Escalate all high/critical risks to steering committee. Dedicate sprint capacity to risk mitigation. Consider engagement pause if risks threaten project viability. Document risk acceptance decisions formally. |

---

## Weekly Health Check Process

1. **Compute scores**: Run health assessment across all six dimensions
2. **Identify trends**: Compare to previous week. Flag any dimension that dropped a tier
3. **Generate signals**: List specific data points driving each score
4. **Prioritize actions**: Focus on the lowest-scoring dimension first
5. **Communicate**: Include health summary in weekly status report to client
6. **Escalate**: Any Red dimension triggers steering committee notification

### Health Report Template

```
## Engagement Health — [ENGAGEMENT_TITLE]
**Week of**: [DATE]  **Overall**: [SCORE] ([GREEN|YELLOW|RED])

| Dimension | Score | Status | Trend | Key Signal |
|-----------|-------|--------|-------|------------|
| Scope | 0.XX | G/Y/R | up/down/stable | [one-liner] |
| Timeline | 0.XX | G/Y/R | up/down/stable | [one-liner] |
| Technical | 0.XX | G/Y/R | up/down/stable | [one-liner] |
| Relationship | 0.XX | G/Y/R | up/down/stable | [one-liner] |
| Budget | 0.XX | G/Y/R | up/down/stable | [one-liner] |
| Risk | 0.XX | G/Y/R | up/down/stable | [one-liner] |

### Actions This Week
1. [Highest priority action from lowest-scoring dimension]
2. [Second priority]
3. [Third priority]
```
