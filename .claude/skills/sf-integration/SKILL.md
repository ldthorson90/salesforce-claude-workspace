---
name: sf-integration
description: Design and build Salesforce integrations — Connected Apps, Named Credentials, External Services, callouts, Platform Events, and CDC.
user-invocable: true
---

# Salesforce Integration

Design and implement integrations between Salesforce and external systems.

## Arguments

- `design <description>` — design an integration pattern
- `connected-app <name>` — create a Connected App configuration
- `named-credential <name>` — create a Named Credential
- `callout <name>` — generate Apex callout class with mock
- `platform-event <name>` — create a Platform Event definition
- `external-service <name>` — set up External Services from OpenAPI spec

## Integration Pattern Selection

Refer to `/sf-decide` with the `08-Data-Integration.md` guide for pattern selection. Quick reference:

| Pattern | Direction | Timing | Volume | Best For |
|---|---|---|---|---|
| REST API callout | SF → External | Real-time | Low-medium | Single record ops, on-demand |
| Platform Events | SF ↔ External | Near real-time | Medium | Event-driven, decoupled |
| Change Data Capture | SF → External | Near real-time | Medium | Replicate SF changes |
| Outbound Messages | SF → External | Near real-time | Low | Simple notifications |
| Bulk API | External → SF | Batch | High | Data loads, migrations |
| MuleSoft/Middleware | SF ↔ External | Any | Any | Complex transformations |
| External Services | SF → External | Real-time | Low | Declarative REST callouts |
| Salesforce Connect | External → SF | Real-time | Read-only | External data display |

## Connected App Setup

```xml
<!-- ConnectedApp metadata -->
<ConnectedApp>
    <label>MyIntegration</label>
    <contactEmail>admin@company.com</contactEmail>
    <oauthConfig>
        <callbackUrl>https://login.salesforce.com/services/oauth2/callback</callbackUrl>
        <scopes>Api</scopes>
        <scopes>RefreshToken</scopes>
    </oauthConfig>
</ConnectedApp>
```

**Rules:**
- Always use Connected App + Named Credential (never hardcode credentials)
- Minimum scopes needed (don't use Full Access unless required)
- Set IP restrictions where possible
- Enable "Require Proof Key for Code Exchange (PKCE)" for public clients
- Set session policies (token expiration, refresh token policy)

## Named Credentials

**Always use Named Credentials for external callouts.** They:
- Store auth credentials securely (not in code)
- Handle token refresh automatically
- Support OAuth 2.0, JWT, AWS Signature, custom auth
- Are deploy-safe (credentials stay in the target org)

```xml
<NamedCredential>
    <label>ExternalAPI</label>
    <endpoint>https://api.example.com</endpoint>
    <principalType>NamedUser</principalType>
    <protocol>Oauth</protocol>
</NamedCredential>
```

**In Apex:**
```apex
HttpRequest req = new HttpRequest();
req.setEndpoint('callout:ExternalAPI/v1/endpoint');
req.setMethod('GET');
Http http = new Http();
HttpResponse res = http.send(req);
```

## Apex Callout Pattern

```apex
public class ExternalApiService {
    private static final String NAMED_CRED = 'callout:ExternalAPI';

    public static HttpResponse makeRequest(String endpoint, String method, String body) {
        HttpRequest req = new HttpRequest();
        req.setEndpoint(NAMED_CRED + endpoint);
        req.setMethod(method);
        req.setTimeout(120000); // 2 min max
        req.setHeader('Content-Type', 'application/json');
        if (String.isNotBlank(body)) {
            req.setBody(body);
        }
        return new Http().send(req);
    }

    public static List<ExternalRecord> getRecords() {
        HttpResponse res = makeRequest('/v1/records', 'GET', null);
        if (res.getStatusCode() == 200) {
            return (List<ExternalRecord>) JSON.deserialize(
                res.getBody(), List<ExternalRecord>.class
            );
        }
        throw new CalloutException('API returned ' + res.getStatusCode());
    }
}
```

**Always generate the mock:**
```apex
@IsTest
public class ExternalApiServiceMock implements HttpCalloutMock {
    private Integer statusCode;
    private String body;

    public ExternalApiServiceMock(Integer statusCode, String body) {
        this.statusCode = statusCode;
        this.body = body;
    }

    public HttpResponse respond(HttpRequest req) {
        HttpResponse res = new HttpResponse();
        res.setStatusCode(this.statusCode);
        res.setBody(this.body);
        return res;
    }
}
```

## Platform Events

```xml
<CustomObject>
    <label>Order Status Change</label>
    <pluralLabel>Order Status Changes</pluralLabel>
    <deploymentStatus>Deployed</deploymentStatus>
    <eventType>HighVolume</eventType>
    <publishBehavior>PublishAfterCommit</publishBehavior>
    <fields>
        <fullName>Order_Id__c</fullName>
        <type>Text</type><length>18</length>
    </fields>
</CustomObject>
```

**Rules:**
- Use `PublishAfterCommit` unless you need immediate publication
- `HighVolume` for production workloads (standard has lower limits)
- Subscribe via Flow (preferred) or Apex trigger
- Batch size for PE triggers: 2,000 (not 200)
- Use `EventBus.publish()` — returns `SaveResult`, check for errors
- ReplayId for subscriber recovery after failures

## Governor Limits

| Limit | Value |
|---|---|
| Callout timeout | 120 seconds max |
| Callout response size | 6MB sync, 12MB async |
| Callouts per transaction | 100 |
| Platform Events per transaction | 150 (publishAfterCommit) |
| Concurrent long-running callouts | 10 |

## Context7 References

- `/damecek/salesforce-documentation-context` — "Named Credentials callout Platform Events"
- `/salesforcecli/cli` — "sf org create connected-app"
