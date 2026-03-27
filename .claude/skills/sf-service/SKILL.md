---
name: sf-service
description: Design and implement Service Cloud — case management, entitlements, milestones, knowledge, omnichannel routing, service console, and chatbots. Aligned to Service Cloud Consultant certification.
user-invocable: true
---

# Salesforce Service Cloud

Design and implement service operations on the Salesforce platform.

## Arguments

- `design <requirements>` — design a service solution
- `case-management` — configure case processes, queues, assignment, escalation
- `entitlements` — set up entitlements, milestones, and SLA tracking
- `knowledge` — configure Knowledge base
- `omnichannel` — configure Omni-Channel routing
- `console` — design Service Console layout

## Case Management

### Case Process Design

| Component | Purpose | Configuration |
|---|---|---|
| Support Process | Define valid case statuses per record type | Setup > Support Processes |
| Assignment Rules | Route new cases to queues/users | Based on criteria (origin, priority, type) |
| Escalation Rules | Escalate cases not resolved in time | Based on age + criteria |
| Auto-Response Rules | Send confirmation to customer | Based on case origin/criteria |
| Case Teams | Multi-person case collaboration | Predefined or ad-hoc teams |

### Queue Design
```
Queue: Tier1_Support
  Supported Objects: Case
  Members: Support_Rep role
  Email: tier1@company.com
  Assignment Rule: Origin = 'Web' OR Origin = 'Email'

Queue: Tier2_Engineering
  Supported Objects: Case
  Members: Engineering role
  Assignment Rule: Escalated = true OR Type = 'Bug'
```

### Case Lifecycle
```
New → In Progress → Waiting on Customer → In Progress → Resolved → Closed
                  → Escalated → In Progress (Tier 2)
```

- Auto-close cases after N days in Resolved status (scheduled Flow)
- Re-open on customer reply (email-to-case integration)
- Track first response time and resolution time

## Entitlements & Milestones

### Entitlement Process
```
Entitlement: Premium_Support
  SLA: 4 hours first response, 24 hours resolution
  Business Hours: Mon-Fri 8AM-6PM PST

Milestones:
  1. First Response (4 business hours)
     - Warning: 2 hours remaining → notify owner
     - Violation: 0 hours → escalate to manager
  2. Resolution (24 business hours)
     - Warning: 4 hours remaining → notify queue
     - Violation: 0 hours → escalate to VP Support
```

### Configuration Steps
1. Enable Entitlement Management
2. Create Business Hours
3. Create Milestones (First Response, Resolution)
4. Create Entitlement Process with milestone actions
5. Create Entitlements on Account or Asset
6. Add Milestone Tracker to Case page layout

## Knowledge

### Article Types and Data Categories
```
Article Types:
  - FAQ (Short question/answer, public)
  - How-To (Step-by-step, internal + portal)
  - Troubleshooting (Problem/cause/resolution, internal)
  - Policy (Reference documentation, internal)

Data Categories:
  Products > Product_A, Product_B, Product_C
  Topics > Billing, Technical, Account_Management
```

### Knowledge Configuration
- Enable Knowledge in Setup
- Create article types with custom fields
- Set up data categories for organization
- Configure article visibility (internal, portal, public)
- Enable suggested articles on case (matches keywords)
- Lightning Knowledge: use record types instead of article types (modern approach)

### Knowledge-Centered Service (KCS)
- Agents create/update articles as they resolve cases
- Link articles to cases for reuse tracking
- Review cycle: Draft → Published → Archived
- Measure: article attachment rate, reuse rate, deflection rate

## Omni-Channel

### Routing Configuration

| Routing Type | Use Case |
|---|---|
| Queue-Based | Simple FIFO within a queue |
| Skills-Based | Match case attributes to agent skills |
| External Routing | Third-party ACD integration |

### Skills-Based Routing
```
Skill: Spanish_Language (proficiency 1-10)
Skill: Premium_Account (boolean)
Skill: Technical_Troubleshooting (proficiency 1-5)

Routing Rule: Premium_Technical_Case
  Condition: Account.Is_Premium__c = true AND Type = 'Technical'
  Required Skills: Premium_Account, Technical_Troubleshooting >= 3
  Preferred Skills: Spanish_Language >= 5 (if Contact.Language = 'Spanish')
```

### Agent Capacity
- Set capacity per channel: Chat = 3, Case = 5, Phone = 1
- Interruptible vs non-interruptible work items
- Agent presence statuses: Available, Busy, Away, Offline
- Capacity-based assignment prevents overload

### Channels
| Channel | Setup |
|---|---|
| Chat | Embedded Service, Messaging |
| Email | Email-to-Case |
| Phone | Service Cloud Voice (Amazon Connect) |
| SMS | Messaging |
| WhatsApp | Messaging |
| Social | Social Customer Service |

## Service Console

### Console Design Principles
- **Split view:** Case list on left, detail on right
- **Utility bar:** Knowledge, Macros, History, Softphone
- **Highlights panel:** Key case info (Priority, Status, Account, SLA)
- **Subtabs:** Related records open as subtabs (not new windows)
- **Quick actions:** Close Case, Escalate, Change Owner, Log a Call

### Recommended Components
1. Highlights Panel (case priority, status, owner, SLA timer)
2. Case Feed (activity timeline)
3. Knowledge sidebar (suggested articles)
4. Related Lists (Contacts, Assets, Entitlements)
5. Milestone Tracker
6. Macros utility (bulk actions)

## Metrics

| KPI | Formula |
|---|---|
| First Contact Resolution (FCR) | Cases resolved in first interaction / Total cases |
| Average Handle Time (AHT) | Total handle time / Number of cases |
| Customer Satisfaction (CSAT) | Survey responses (post-case) |
| SLA Compliance | Cases within SLA / Total cases with entitlements |
| Case Deflection Rate | Self-service resolutions / Total inquiries |
| Agent Utilization | Active work time / Available time |
