---
name: sf-agentforce
description: Build and configure Salesforce Agentforce — AI agents, prompt templates, agent actions, Einstein Trust Layer, agent testing, and Models API. Aligned to AI Specialist certification.
user-invocable: true
---

# Salesforce Agentforce

Build, configure, and test AI agents on the Salesforce platform.

## Arguments

- `agent <name>` — create an Agentforce agent configuration
- `action <name>` — create an agent action (Flow-based or Apex-based)
- `prompt-template <name>` — create a Prompt Template
- `test <agent-name>` — design agent test scenarios
- `trust` — review Einstein Trust Layer configuration

## Agent Architecture

### Agent Components

```
Agent (GenAiAgent)
├── Topics — scope what the agent can handle
│   ├── Instructions — natural language guidance per topic
│   ├── Actions — what the agent can DO
│   │   ├── Flow Actions (invocable flows)
│   │   ├── Apex Actions (@InvocableMethod)
│   │   ├── Prompt Template Actions
│   │   └── MCP Server Actions (external tools)
│   └── Guardrails — what the agent must NOT do
└── Channels — where the agent is deployed
    ├── Messaging (web chat, WhatsApp, SMS)
    ├── Voice
    ├── Slack
    └── API (headless)
```

### Agent Types

| Type | Use Case | Builder |
|---|---|---|
| Service Agent | Customer service, case deflection | Agent Builder (declarative) |
| Sales Agent | Lead qualification, opportunity coaching | Agent Builder |
| Custom Agent | Domain-specific workflows | Agent Builder + code |
| Code-First Agent | Complex multi-step, API-driven | Apex + Agent Script DSL |

## Agent Builder (Declarative)

### Topic Design

Each topic represents a capability domain:

```yaml
Topic: Order Management
  Description: "Help customers check order status, modify orders, and process returns"
  Instructions:
    - "Always verify customer identity before sharing order details"
    - "For returns, check if the order is within the 30-day return window"
    - "Escalate to human agent if the customer is frustrated"
  Actions:
    - Get_Order_Status (Flow)
    - Process_Return (Flow)
    - Update_Shipping_Address (Apex)
  Guardrails:
    - "Never share other customers' order information"
    - "Do not process refunds over $500 without human approval"
```

**Rules:**
- One topic per capability domain (don't overload)
- Instructions: clear, specific, imperative ("Do X", "Never Y")
- 3-5 actions per topic (agent selection degrades with too many)
- Always include escalation path to human agent

### Action Design

**Flow Actions (preferred for declarative):**
```
Flow: Get_Order_Status
  Input: orderId (Text) — @InvocableVariable
  Steps:
    1. Get Records: Order where Id = orderId
    2. Get Records: OrderItems where OrderId = orderId
    3. Output: orderStatus, itemList, estimatedDelivery
```

**Apex Actions:**
```apex
public class OrderActions {
    @InvocableMethod(label='Process Return' description='Initiates a return for an order')
    public static List<ReturnResult> processReturn(List<ReturnRequest> requests) {
        // Bulk-safe implementation
        List<ReturnResult> results = new List<ReturnResult>();
        for (ReturnRequest req : requests) {
            // Process each return
            results.add(new ReturnResult(req.orderId, 'Return initiated'));
        }
        return results;
    }

    public class ReturnRequest {
        @InvocableVariable(required=true description='The order to return')
        public Id orderId;
        @InvocableVariable(description='Reason for return')
        public String reason;
    }

    public class ReturnResult {
        @InvocableVariable
        public Id orderId;
        @InvocableVariable
        public String status;
    }
}
```

**Rules for actions:**
- Always use `@InvocableMethod` with `label` and `description` (agent uses these for selection)
- Input/output classes with `@InvocableVariable` and `description` on each
- Bulkify: accept and return `List<>` even for single-record operations
- Keep actions focused: one action = one capability
- Return structured data the agent can interpret

## Prompt Templates

```
Prompt Template: Summarize_Case_History
  Type: Flex
  Inputs:
    - caseId: Id (merge field: {!caseId})
  Data Sources:
    - Case: Id, Subject, Description, Status, Priority
    - CaseComments: CommentBody, CreatedDate, CreatedBy.Name
  Template:
    "Summarize the following case for a service agent:
     Case: {!Case.Subject} (Status: {!Case.Status}, Priority: {!Case.Priority})
     Description: {!Case.Description}
     Recent comments:
     {!CaseComments}

     Provide a 2-3 sentence summary focused on current status and next steps."
```

### Template Types
- **Sales Emails** — generate outreach with merge fields from Lead/Opportunity
- **Case Summary** — summarize case history for agent handoff
- **Knowledge Search** — generate search queries from customer input
- **Field Generation** — auto-populate fields from context

### Prompt Builder
- Use Prompt Builder UI for template creation and testing
- Ground templates with merge fields ({!Record.Field}) for accuracy
- Test with representative records before deploying
- Version templates (prompt engineering is iterative)

## Einstein Trust Layer

### Configuration Checklist
- [ ] Data masking rules: PII fields (SSN, credit card) masked before LLM
- [ ] Toxicity detection: enabled for customer-facing agents
- [ ] Audit trail: log all agent interactions
- [ ] Zero data retention: verify LLM provider settings
- [ ] Grounding: connect to Knowledge, Data Cloud, or CRM data

### Grounding Strategy
- **CRM Grounding:** Agent retrieves records via SOQL/actions before responding
- **Knowledge Grounding:** Agent searches Knowledge articles for answers
- **Data Cloud Grounding:** Agent queries unified customer profiles
- **Hybrid:** Combine CRM + Knowledge for best results

### Security Rules
- Agents inherit the running user's permissions (sharing, FLS, CRUD)
- Guest user agents: extremely restricted, Knowledge-only recommended
- Service agents: run as the assigned service user, not the customer
- API agents: use Connected App permissions

## Agent Testing

### Test Scenarios
For each topic, test:
1. **Happy path** — standard request, expected response
2. **Edge case** — ambiguous input, missing data
3. **Guardrail test** — attempt to violate a guardrail (should refuse)
4. **Escalation test** — scenario that should route to human
5. **Multi-turn** — conversation that requires context across messages

### CLI Testing
```bash
sf agent generate test --agent-name <name> -o <org-alias>
sf agent test run -o <org-alias> --result-format human
```

### Evaluation Metrics
- **Topic classification accuracy** — did the agent pick the right topic?
- **Action selection accuracy** — did it call the right action?
- **Grounding accuracy** — did it use real data, not hallucinate?
- **Guardrail adherence** — did it refuse when it should?
- **Resolution rate** — % of conversations resolved without escalation

## Metadata Location

- Agents: `force-app/main/default/genAiAgents/`
- Prompt Templates: `force-app/main/default/genAiPromptTemplates/`
- Agent Actions: implemented as Flows or Apex classes

## Context7 References

- `/damecek/salesforce-documentation-context` — "Agentforce agent builder prompt template"
