---
name: sf-decide
description: Route architecture questions to the right Salesforce architect decision guide. Covers automation, async, integration, data, forms, and event-driven patterns.
user-invocable: true
---

# Salesforce Architecture Decision Router

Surface the right architect decision guide for a design question.

## Arguments

The user describes their architecture question in natural language. No specific format required.

## Decision Guide Routing

Map the user's question to the right architecture domain and fetch guidance dynamically:

| Keywords / Intent | Topic | Source URL | Context7 Query |
|---|---|---|---|
| trigger, flow, automation, record-triggered, before/after | Record-Triggered Automation | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/trigger-automation) | Query `/damecek/salesforce-documentation-context` for "record triggered flow vs apex trigger" |
| CDP, customer data platform, unified profile, provisioning | Data 360 Provisioning | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/data-360-provisioning) | Query for "data cloud provisioning setup" |
| data exchange, cross-cloud, data sharing, system of record | Data 360 Interoperability | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/data-360-interoperability) | Query for "data cloud interoperability cross-cloud" |
| step-based, multi-step, wizard, orchestration, approval | Step-Based Async Framework | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/async-framework) | Query for "step based async orchestration framework" |
| async, batch, queueable, schedulable, future, bulk | Asynchronous Processing | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/async-processing) | Query for "batch queueable schedulable future method comparison" |
| form, input, screen flow, dynamic form, data entry | Building Forms | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/forms) | Query for "screen flow dynamic form lightning page input" |
| event, platform event, CDC, pub/sub, streaming | Event-Driven Architecture | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/event-driven) | Query for "platform event change data capture streaming" |
| integration, API, REST, SOAP, callout, middleware, ETL | Data Integration | [architect.salesforce.com](https://architect.salesforce.com/decision-guides/data-integration) | Query for "integration pattern REST SOAP callout middleware" |

## Workflow

### Step 1: Match intent

Parse the user's question for keywords from the table above. If multiple guides match, rank by relevance and ask the user which to explore.

If no guide matches, say so and provide general Salesforce architecture advice based on the rules in `~/.claude/rules/salesforce.md`.

### Step 2: Fetch architecture guidance

Use Context7 to query the relevant documentation dynamically:

```
mcp__context7__query-docs with libraryId="/damecek/salesforce-documentation-context"
```

Also query the Well-Architected framework:
```
mcp__context7__query-docs with libraryId="/websites/architect_salesforce_well-architected"
```

If local guide files exist in `salesforce-architect-decision-guides/`, read those as well (they provide deeper decision frameworks). If not, the Context7 queries provide equivalent guidance.

Provide the source URL from the routing table so the user can read the full official guide.

### Step 3: Apply to the user's context

After reading the guide:
1. **Summarize** the relevant decision framework (the key decision points, not the entire doc)
2. **Map** the user's specific scenario to the decision tree
3. **Recommend** a specific pattern with clear reasoning
4. **Flag trade-offs**: governor limits, scalability, maintainability, testing complexity
5. **Challenge** the user's assumption if a different pattern is clearly better (per consulting governance)

### Step 4: Connect to implementation

If the user confirms a pattern:
- Suggest the right skill to start building:
  - Trigger/handler: `/sf-component apex-trigger <Object>`
  - LWC: `/sf-component lwc <Name>`
  - Test class: `/sf-test-gen <Class>`
  - Deployment: `/sf-deploy <org>`
- Offer to generate an architecture decision record (ADR):

```markdown
# ADR: <Title>

## Status: Accepted
## Date: <today>

## Context
<what the user described>

## Decision
<chosen pattern from the guide>

## Rationale
<why this pattern fits, referencing the decision guide>

## Consequences
- <trade-off 1>
- <trade-off 2>

## Alternatives Considered
- <rejected pattern>: <why not>
```

### Step 5: Document (optional)

If the user wants to persist the decision:
- Save the ADR to the project directory
- Or add a note to the project CLAUDE.md under a "## Architecture Decisions" section

## Important

- Always present trade-offs, never just a single recommendation
- Challenge assumptions when a different pattern is clearly better
- Reference governor limits and scalability where relevant
- The decision guides are authoritative — trust them over general knowledge
- This skill is advisory only — it reads guides and recommends, never generates code
