# Discovery Questions

Structured interview questions for Salesforce consulting engagements.
Organized by engagement type, with questions covering business context,
current state, requirements, constraints, and success criteria.

Ported from ConsultHub's discovery question bank (categories: discovery,
assessment, technical, stakeholder, process) and adapted for Salesforce-
specific consulting scenarios.

---

## New Build (Greenfield Implementation)

### Business Context
1. What business outcomes are you expecting from this Salesforce implementation? What KPIs will you use to measure success 6 and 12 months post-launch?
2. Who are the executive sponsors and key stakeholders? What is their level of engagement and decision-making authority?
3. Walk me through the end-to-end lifecycle of your core business process (lead-to-cash, case-to-resolution, etc.). Where are the biggest pain points today?
4. How many users will need access on day one? How do you expect that to grow over the next 2 years?
5. What does your current technology landscape look like? Are there any systems that Salesforce must replace vs. integrate with?
6. Have you evaluated other platforms? What drew you to Salesforce specifically?
7. What compliance or regulatory requirements apply to your industry (HIPAA, SOX, GDPR, PCI, FedRAMP)?
8. Is there an internal IT team that will own the platform post-launch, or will you need ongoing managed services?
9. What does your fiscal year and planning cycle look like? Are there hard dates tied to business events (board meetings, product launches, regulatory deadlines)?
10. How do your teams currently collaborate? What tools do they use for communication, project management, and document sharing?

### Current State
11. What systems are you using today for CRM, support, or sales tracking? Are they commercial products, spreadsheets, or custom-built?
12. How is your data currently structured? Do you have a single source of truth for customer information, or is it fragmented across systems?

### Requirements
13. Which Salesforce clouds are you considering (Sales, Service, Marketing, Experience, etc.)? What drove that initial assessment?
14. Are there specific features you consider must-have for go-live vs. nice-to-have for a future phase?

### Constraints
15. What is your target go-live date, and what is driving that timeline? Is it flexible?
16. What budget range has been approved for this initiative (implementation + first-year licensing)?
17. Do you have internal resources available for testing, training, and change management?

### Success Criteria
18. If this implementation is wildly successful, what does that look like in concrete terms?
19. What has gone wrong on past technology projects? What would you do differently this time?
20. How will you handle change management and user adoption? Do you have a training plan or team?

---

## Optimization (Existing Org Improvement)

### Business Context
1. What specific problems are you trying to solve with this optimization? What triggered this initiative now?
2. How long has your Salesforce org been in production? How many admins and developers have worked on it over that time?
3. What is your current user adoption rate? Which teams are heavy users and which are resistant?
4. Are there business processes that have changed since the original implementation but Salesforce was never updated to match?
5. What Salesforce licenses do you currently own? Are you using all of them, or are some underutilized?

### Current State
6. When was the last time a technical health check or org audit was performed?
7. How many custom objects, flows, and Apex triggers exist in the org? Do you have documentation for them?
8. Are you on Lightning Experience or still using Classic for any user groups?
9. What is your current test coverage percentage? Are tests meaningful or just coverage padding?
10. How are deployments handled today? Do you have a CI/CD pipeline, change sets, or manual deployments?

### Requirements
11. Are there governor limit issues (SOQL queries, CPU time, heap size) that users encounter regularly?
12. What reports or dashboards do leadership rely on? Are they accurate and trusted?

### Constraints
13. Can we schedule downtime or maintenance windows, or must all changes be zero-disruption?
14. Are there AppExchange packages installed that constrain what we can modify?
15. What is the appetite for rebuilding vs. incremental improvement? Are there sacred cows?

### Success Criteria
16. How will you measure whether the optimization was worth the investment?
17. What does "clean" look like to you -- faster performance, better data quality, simpler admin experience, or all of the above?
18. What is the acceptable timeline for seeing measurable improvements?

---

## Migration (From Another CRM or Legacy System)

### Business Context
1. What system are you migrating from? How long has it been in use, and how much institutional knowledge is embedded in it?
2. What is driving the migration? Is it end-of-life, cost, capability gaps, or strategic alignment?
3. How many records are we talking about? Rough order of magnitude for accounts, contacts, opportunities, cases, and custom entities.
4. Are there integrations with the current system that must be replicated or redesigned in Salesforce?
5. Who are the power users of the current system? What do they rely on most heavily?

### Current State
6. How clean is the existing data? When was the last data quality audit or deduplication effort?
7. Are there custom fields, workflows, or reports in the source system that have no standard Salesforce equivalent?
8. What is the current data model? Do you have an ERD or schema documentation?
9. How are historical records handled -- do you need full history, summary records, or archive-only?
10. Are there any data sovereignty or residency requirements that affect where data can be stored?

### Requirements
11. What is the cutover strategy -- big bang, parallel run, or phased migration by business unit?
12. Which data entities are critical for day-one and which can migrate in subsequent phases?
13. Do you need to maintain the source system in read-only mode for a transition period?

### Constraints
14. Are there contractual obligations with the current vendor (data export formats, retention periods, termination clauses)?
15. What is the data export capability of the source system? APIs, CSV exports, direct database access?
16. Is there a hard deadline (contract expiry, licensing renewal) that constrains the migration timeline?

### Success Criteria
17. How will you validate that the migration is complete and accurate? What is your acceptance criteria?
18. What is the rollback plan if critical issues are discovered post-migration?
19. What does a successful first week on Salesforce look like for your end users?

---

## Integration (Connecting Salesforce to External Systems)

### Business Context
1. What systems need to connect to Salesforce? For each, what is the business process that depends on the integration?
2. What is the direction of data flow -- Salesforce as system of record pushing out, external systems pushing in, or bidirectional sync?
3. How real-time does the integration need to be? Are there processes that require sub-second response, or is batch/near-real-time acceptable?
4. Who owns the external systems? Are they managed internally, by a vendor, or a third party?
5. What happens today when these systems are not connected? Manual data entry, exports/imports, or workarounds?

### Current State
6. Are there existing integrations (middleware, iPaaS, point-to-point) that we need to account for or replace?
7. What APIs do the external systems expose? REST, SOAP, GraphQL, file-based, database-level?
8. What authentication and authorization mechanisms are in place? OAuth, API keys, certificates, IP whitelisting?
9. What is the expected data volume per integration? Records per hour/day, payload sizes, peak vs. average load.
10. Are there existing Salesforce integration tools in use (MuleSoft, Informatica, Jitterbit, native Connected Apps)?

### Requirements
11. What error handling and retry logic is required? What happens when an integration fails at 2 AM?
12. Do you need transformation or mapping logic between systems, or is the data structurally compatible?
13. Are there audit trail or logging requirements for integrated data movements?

### Constraints
14. What are the API rate limits on both sides? Salesforce API call limits vs. external system throttling.
15. Are there network constraints -- VPN requirements, firewall rules, IP restrictions?
16. What is the Salesforce edition? API access and limits vary significantly by license tier.

### Success Criteria
17. How will you monitor integration health in production? Alerting, dashboards, SLA targets?
18. What is the acceptable latency and error rate for each integration?
19. How will you handle schema changes on either side without breaking the integration?

---

## Support (Ongoing Managed Services)

### Business Context
1. What level of support do you need -- reactive break/fix, proactive optimization, or strategic advisory?
2. How many support requests do you typically generate per month? What is the mix of admin tasks, bug fixes, and enhancement requests?
3. Who will be the primary point of contact on your side for day-to-day requests?
4. Are there specific SLAs you need (response time, resolution time, availability hours)?
5. Do you have an internal admin or team that handles tier-1 requests, or do you need full coverage?

### Current State
6. What is the current state of your org -- stable and well-documented, or accumulated technical debt?
7. Are there any known issues, performance problems, or broken functionality that need immediate attention?
8. What is your release cadence? Do you deploy changes quarterly, monthly, or ad-hoc?
9. How are change requests currently tracked and prioritized? Is there a backlog?
10. What documentation exists for the current org configuration, customizations, and integrations?

### Requirements
11. Do you need support for Salesforce releases (3x per year) -- impact analysis, sandbox testing, regression validation?
12. Are there recurring seasonal peaks (year-end close, enrollment periods, product launches) that require surge capacity?
13. Do you need user training or enablement as part of the support engagement?

### Constraints
14. What is the monthly or annual budget allocated for managed services?
15. Are there security or compliance requirements for support personnel (background checks, certifications, data access restrictions)?
16. What tools and access will the support team need? Sandbox access, production read-only, deployment permissions?

### Success Criteria
17. How will you measure the value of the support engagement? Ticket resolution metrics, user satisfaction, org health improvements?
18. What does a successful first 90 days look like?
19. At what point would you consider bringing support capabilities in-house?
