# Risk Patterns

Common Salesforce consulting risk patterns. Each pattern includes trigger
signals, severity classification, impact analysis, and mitigation strategies.

Ported from ConsultHub's risk tracker (pattern types: timeline_risk,
communication_gap, scope_drift, resource_constraint, technical_debt)
and expanded with Salesforce platform-specific patterns.

---

### 1. Scope Creep via "Quick Adds"

- **Category**: Scope
- **Trigger Signals**: 3+ scope flags across recent meetings; client repeatedly says "while you're in there, can you also..."; requirements document growing without corresponding SOW amendments
- **Severity**: High
- **Impact**: Budget overrun, timeline slip, team burnout. Unbillable work accumulates. Original deliverables get deprioritized.
- **Mitigation**: Maintain a change request log. Every addition gets sized, approved, and added to a formal change order. Set a threshold (e.g., 10% of project hours) that triggers a scope review meeting.
- **Salesforce-Specific Notes**: Common with declarative customizations ("it's just a flow") that appear trivial but cascade into validation rules, page layouts, profiles, and test coverage.

### 2. Timeline Overrun with Open Action Items

- **Category**: Timeline
- **Trigger Signals**: Engagement past end date with open action items still pending; sprint velocity declining; milestone dates slipping without formal replan
- **Severity**: High
- **Impact**: Client loses confidence. Cost overruns if T&M; margin erosion if fixed-price. Downstream projects queue up behind the delay.
- **Mitigation**: Weekly burn-down tracking visible to both teams. Formal replan when any milestone slips by more than one week. Escalate to steering committee if two consecutive sprints miss targets.
- **Salesforce-Specific Notes**: Often caused by underestimating data migration complexity, sandbox refresh timing, or Salesforce release impact windows (3x per year).

### 3. Communication Gap

- **Category**: Organizational
- **Trigger Signals**: No client meetings in 14+ days on an active engagement; emails going unanswered for 48+ hours; key stakeholders not attending scheduled sessions
- **Severity**: Medium (escalates to High if sustained 21+ days)
- **Impact**: Decisions stall, assumptions go unvalidated, rework increases. Risk of building the wrong thing grows exponentially with silence duration.
- **Mitigation**: Establish a communication cadence in the SOW (weekly at minimum). Escalate to executive sponsor if primary contact goes dark for 10+ business days. Document all decisions with written confirmation.
- **Salesforce-Specific Notes**: Common during UAT phases when client teams are pulled back to operational duties. Also common when the client admin leaves the organization mid-project.

### 4. Data Quality Disaster

- **Category**: Data
- **Trigger Signals**: Source system exports reveal >20% null values in required fields; duplicate rates >10%; no canonical record ID across systems; data owner cannot explain field semantics
- **Severity**: Critical
- **Impact**: Migration timelines double or triple. Data cleansing becomes an unplanned workstream. Post-migration, users distrust the new system, killing adoption.
- **Mitigation**: Run a data profiling exercise in discovery (before SOW). Include a data cleansing phase with explicit client ownership. Define acceptance criteria for data quality before migration starts.
- **Salesforce-Specific Notes**: Salesforce duplicate rules and matching rules help post-migration, but garbage-in during Bulk API loads creates record locking issues and trigger failures at scale. External IDs are essential for upsert-based migration.

### 5. Governor Limit Risk

- **Category**: Technical
- **Trigger Signals**: Code reviews reveal SOQL or DML inside loops; trigger handlers without bulkification; synchronous callouts in trigger context; test classes using `@isTest(SeeAllData=true)`
- **Severity**: High
- **Impact**: Runtime failures in production at scale. Issues may not surface until data volumes grow past test thresholds (200+ records). Difficult to retrofit bulkification into non-bulk code.
- **Mitigation**: Enforce code review standards before any deployment. Run Salesforce Code Analyzer on every PR. Load test with realistic data volumes in a full sandbox. Establish governor limit budgets per transaction.
- **Salesforce-Specific Notes**: Key limits: 100 SOQL queries, 150 DML statements, 10-second CPU time, 6MB heap, 100 callouts per transaction. Async limits differ (200 SOQL, 12-minute CPU). Platform events and Change Data Capture help offload processing.

### 6. Integration Fragility

- **Category**: Technical
- **Trigger Signals**: Point-to-point integrations without middleware; no retry or dead-letter queue; hard-coded endpoints or credentials; no monitoring or alerting on integration failures
- **Severity**: High
- **Impact**: Silent data loss. Business processes break without anyone knowing until a user reports missing records. Cascading failures across connected systems.
- **Mitigation**: Use middleware (MuleSoft, Workato, etc.) or Salesforce Platform Events for decoupling. Implement circuit breakers, retry with exponential backoff, and dead-letter queues. Monitor via custom objects or external APM.
- **Salesforce-Specific Notes**: Salesforce API limits (daily and concurrent) are hard caps. Named Credentials for auth management. External Services for declarative API integration. Outbound Messages are fire-and-forget with built-in retry.

### 7. Single Point of Failure (Key Person Risk)

- **Category**: Organizational
- **Trigger Signals**: Only one person understands the system architecture or holds admin credentials; no documentation of customizations; tribal knowledge concentrated in a single resource
- **Severity**: High
- **Impact**: Project stalls if that person is unavailable. Post-launch support becomes impossible. Client is locked into a single vendor or individual.
- **Mitigation**: Require documentation as a deliverable, not an afterthought. Cross-train at least two people on every critical component. Ensure admin credentials are in a shared vault, not personal accounts.
- **Salesforce-Specific Notes**: Common with "accidental admins" who built the org over years without formal training. Login-as access and connected app tokens are frequently tied to individual accounts rather than service users.

### 8. Insufficient Test Coverage

- **Category**: Technical
- **Trigger Signals**: Test coverage below 75% (Salesforce deployment minimum); tests that assert nothing ("coverage-only" tests); no bulk tests; no negative tests; test classes with `SeeAllData=true`
- **Severity**: Medium
- **Impact**: Production bugs that should have been caught. Deployments blocked by coverage gates. Technical debt that compounds with every release.
- **Mitigation**: Require meaningful assertions in every test method. Test with 200+ record bulk operations. Include positive, negative, and boundary test cases. Use `@TestSetup` for efficient data creation.
- **Salesforce-Specific Notes**: Salesforce enforces 75% org-wide coverage for production deployments, but this is a floor, not a target. Test classes should cover trigger handler logic, batch apex, scheduled apex, and all web service endpoints. Mixed DML errors in tests require `System.runAs()` patterns.

### 9. Change Management Failure

- **Category**: Organizational
- **Trigger Signals**: No training plan in the project plan; end users were not involved in requirements or UAT; executive sponsor disengaged; no communication plan for go-live
- **Severity**: High
- **Impact**: Low adoption post-launch. Users revert to old tools or workarounds. The implementation is technically successful but functionally unused. ROI is never realized.
- **Mitigation**: Include change management as an explicit workstream with its own budget. Involve end users in requirements validation and UAT. Train-the-trainer model. Post-launch adoption metrics with a 90-day action plan.
- **Salesforce-Specific Notes**: Salesforce's In-App Guidance, Path, and Guidance Center features can embed training directly in the UI. Adoption dashboards using Login History and setup audit trail help measure engagement. Chatter groups for support communities.

### 10. Sandbox Strategy Failure

- **Category**: Technical
- **Trigger Signals**: Development happening directly in production; only one sandbox for all purposes; sandbox refresh schedule not aligned with release cadence; full sandbox not available for performance testing
- **Severity**: Medium
- **Impact**: Untested changes reach production. Conflicting work overwrites other developers. No ability to reproduce production issues. Performance problems discovered only after deployment.
- **Mitigation**: Establish a sandbox strategy (Dev, QA/UAT, Staging, Hotfix) with clear promotion paths. Automate sandbox seeding. Align refresh cadence with sprint schedule. Use scratch orgs for feature isolation.
- **Salesforce-Specific Notes**: Sandbox types matter: Developer sandboxes lack data; Partial Copy includes a subset; Full Copy mirrors production. Sandbox refresh limits vary by edition. DevOps Center and sf CLI can automate promotion between environments.

### 11. Security and Access Model Complexity

- **Category**: Technical
- **Trigger Signals**: Requirements include complex record sharing beyond OWD + role hierarchy; matrix-based visibility (region x product x team); request for "they can see everything except..." logic
- **Severity**: Medium
- **Impact**: Overly complex sharing rules degrade query performance. Sharing recalculation jobs time out at scale. Users either see too much (compliance risk) or too little (productivity impact).
- **Mitigation**: Start with the simplest sharing model that meets requirements. Design OWD, role hierarchy, and sharing rules before building. Use permission sets and permission set groups over profiles. Apex managed sharing only as a last resort.
- **Salesforce-Specific Notes**: OWD recalculation can take hours on large orgs. Implicit sharing through lookups (Account-Contact, Account-Opportunity) adds hidden complexity. Shield Platform Encryption adds another access control layer. Territory Management is powerful but adds significant complexity.

### 12. Data Volume and Performance Degradation

- **Category**: Data
- **Trigger Signals**: Objects approaching 1M+ records; reports timing out; list views slow to load; batch jobs hitting CPU time limits; SOQL queries returning 50K+ rows
- **Severity**: High
- **Impact**: User experience degrades. Business-critical reports become unreliable. Batch processing windows extend into business hours. Governor limits hit more frequently.
- **Mitigation**: Implement data archiving strategy (Big Objects, external storage, Heroku Postgres). Add skinny tables and custom indexes via Salesforce support. Use Async SOQL for large queries. Implement pagination in all list views and APIs.
- **Salesforce-Specific Notes**: Salesforce has a 50K row query limit per transaction. Large Data Volume (LDV) best practices include indexed fields in WHERE clauses, avoiding null comparisons, and using skinny tables. Big Objects support billions of rows but with limited query capabilities. Data Cloud can offload analytics workloads.

### 13. Multi-Cloud Coordination Risk

- **Category**: Scope
- **Trigger Signals**: Project involves 3+ Salesforce clouds; different consultants or teams own each cloud; no unified data model design; conflicting automation across clouds
- **Severity**: Medium
- **Impact**: Duplicate automation (e.g., Marketing Cloud and Sales Cloud both updating lead status). Data inconsistencies between clouds. Licensing cost surprises when cross-cloud features require additional SKUs.
- **Mitigation**: Appoint a cross-cloud architect. Design the unified data model before cloud-specific configuration. Map all automation touchpoints across clouds. Validate licensing requirements upfront with Salesforce AE.
- **Salesforce-Specific Notes**: Marketing Cloud and Sales Cloud have separate data stores connected via Marketing Cloud Connect. Service Cloud and Sales Cloud share the same org but compete for governor limits. Experience Cloud sites add public-facing load patterns that differ from internal usage.

### 14. Resource Constraint (Low Billable Ratio)

- **Category**: Organizational
- **Trigger Signals**: Less than 60% of tracked time is billable; high volume of internal meetings, rework, or context switching; team members split across too many engagements simultaneously
- **Severity**: Medium
- **Impact**: Margin erosion. Consultant fatigue and turnover. Client perceives slow progress because available capacity is lower than staffing suggests.
- **Mitigation**: Track billable vs. non-billable time weekly. Cap concurrent engagement assignments per consultant. Reduce internal meeting load. Automate administrative tasks (time entry, status reporting).
- **Salesforce-Specific Notes**: Salesforce projects often have high non-billable overhead during environment setup, sandbox management, and deployment pipeline configuration. Front-load these activities and amortize across the engagement.

### 15. Post-Go-Live Support Vacuum

- **Category**: Timeline
- **Trigger Signals**: No hypercare period defined in the SOW; support handoff plan is vague or absent; client admin team not identified or trained; no runbook for common issues
- **Severity**: High
- **Impact**: Users hit issues with no one to call. Workarounds become permanent. Data quality degrades rapidly. Client satisfaction drops sharply in the first 30 days.
- **Mitigation**: Include a 2-4 week hypercare period in every SOW. Create runbooks for top 10 anticipated issues. Train admin team before go-live, not after. Establish an escalation path and SLA for the transition period.
- **Salesforce-Specific Notes**: Salesforce releases (Spring, Summer, Winter) can break customizations. Ensure the support handoff includes release impact analysis procedures. Omni-Channel routing and assignment rules are frequent sources of post-go-live issues.

### 16. Compliance and Audit Trail Gaps

- **Category**: Data
- **Trigger Signals**: Industry requires full audit trails but field history tracking not configured; no archival strategy for audit data; PII stored without encryption; sharing model allows overly broad access to sensitive records
- **Severity**: Critical
- **Impact**: Regulatory violations, fines, legal liability. Failed audits that block business operations. Loss of customer trust.
- **Mitigation**: Map compliance requirements to Salesforce features during discovery. Configure field history tracking, Setup Audit Trail. Evaluate Shield Platform Encryption and Event Monitoring. Design retention and purge policies.
- **Salesforce-Specific Notes**: Field history tracking is limited to 20 fields per object (standard limit). Shield Event Monitoring provides comprehensive audit data but requires an additional license. Data Mask for sandbox compliance. Salesforce Backup is a separate SKU with limited recovery granularity.

### 17. AppExchange Package Dependency

- **Category**: Technical
- **Trigger Signals**: Solution architecture relies on 3+ third-party AppExchange packages; packages have overlapping functionality; package vendor is a small company with uncertain roadmap; package requires broad permissions
- **Severity**: Medium
- **Impact**: Package updates can break customizations. Vendor discontinuation leaves orphaned components. Package conflicts create hard-to-diagnose errors. Licensing costs add up.
- **Mitigation**: Evaluate each package against build-vs-buy criteria. Check vendor viability (funding, customer count, release history). Test package interactions in a sandbox before committing. Have a contingency plan for each critical package.
- **Salesforce-Specific Notes**: Managed packages cannot be easily removed once deployed to production. Namespace conflicts between packages are common. Package governor limits are separate from org limits but can still cause contention.

### 18. Underestimated Data Migration Complexity

- **Category**: Data
- **Trigger Signals**: Migration scoped as a single sprint; no data profiling done before estimation; source system has 10+ years of data; multiple source systems feeding into one Salesforce org; no ETL tool selected
- **Severity**: High
- **Impact**: Migration becomes the critical path, delaying the entire project. Data quality issues surface during UAT, requiring multiple migration reruns. Go-live delayed by weeks or months.
- **Mitigation**: Always run a data profiling exercise before scoping migration. Plan for at least 3 trial migrations before cutover. Include data cleansing as a client responsibility with defined acceptance criteria. Use Salesforce Data Loader or Bulk API 2.0 for large volumes.
- **Salesforce-Specific Notes**: Salesforce Bulk API 2.0 has a 150MB file size limit per job. Record type assignment during migration requires careful mapping. External IDs are essential for maintaining relationships across objects. Consider order of operations: Accounts before Contacts before Opportunities.

### 19. Automation Conflict (Flow vs. Trigger vs. Process Builder)

- **Category**: Technical
- **Trigger Signals**: Same object has active triggers, flows, process builders, and workflow rules; order of execution not documented; "before" and "after" automation modifying the same fields; recursive execution detected
- **Severity**: Medium
- **Impact**: Unpredictable behavior when records are saved. Difficult to debug because multiple automation layers fire in sequence. Performance degradation from redundant processing. Governor limit consumption multiplied.
- **Mitigation**: Audit all automation per object as a first step in any optimization. Consolidate to a single automation framework (preferably flows + trigger handler pattern). Document the order of execution. Add recursion guards.
- **Salesforce-Specific Notes**: Salesforce order of execution: validation rules, before triggers, after triggers, assignment rules, auto-response rules, workflow rules, process builders, flows (in API version order), then another round of before/after triggers if fields were updated. Record-triggered flows run after after-triggers in the same transaction.

### 20. Fixed-Price Engagement with Vague Requirements

- **Category**: Scope
- **Trigger Signals**: SOW uses phrases like "as needed," "best practice configuration," or "standard implementation" without specifics; requirements document is under 5 pages for a 6-month project; client expects "Salesforce to work like our old system"
- **Severity**: Critical
- **Impact**: Every ambiguity becomes a scope dispute. Consultant absorbs unbounded work. Relationship deteriorates as both sides feel the other is not meeting expectations.
- **Mitigation**: Never sign a fixed-price SOW without detailed requirements. Use a paid discovery phase to produce the requirements that feed the fixed-price estimate. Include explicit exclusions and assumptions. Define a change request process with pricing.
- **Salesforce-Specific Notes**: Salesforce implementations have a particularly wide variance in complexity for seemingly similar requirements. "Configure Sales Cloud" can mean 2 weeks or 6 months depending on the client's process complexity, data model, and integration needs.
