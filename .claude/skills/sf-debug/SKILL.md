---
name: sf-debug
description: Troubleshoot Salesforce issues — debug logs, governor limit diagnosis, SOQL performance, async job tracking, and error pattern analysis.
user-invocable: true
---

# Salesforce Debug & Troubleshoot

Structured approach to diagnosing Salesforce issues.

## Arguments

- `logs <org-alias> [username]` — enable and retrieve debug logs
- `governor <org-alias>` — analyze governor limit consumption
- `query <org-alias> <soql>` — analyze SOQL query performance
- `async <org-alias>` — check async job status (batch, queueable, scheduled)
- `error <description>` — diagnose an error message or behavior

## Debug Log Workflow

### Enable Logging

```bash
# Enable trace flag for a user (8-hour window)
sf data create record -o <org-alias> -s TraceFlag -v "TracedEntityId='005...' LogType='USER_DEBUG' DebugLevelId='...' StartDate='...' ExpirationDate='...'"
```

Or via the tsmztech MCP server if connected:
- `salesforce_manage_debug_logs` tool handles enable/disable/retrieve

### Log Level Guide

| Category | Level for Debugging | Notes |
|---|---|---|
| Apex Code | FINEST | Full variable values, loop iterations |
| Apex Profiling | INFO | Method entry/exit with timing |
| Database | FINE | SOQL queries with bind values |
| System | DEBUG | System.debug output |
| Validation | INFO | Validation rule evaluation |
| Workflow | FINE | Flow/Process Builder execution |
| Callout | FINE | HTTP request/response bodies |

**Production rule:** Use FINEST only temporarily. Reset to NONE within 1 hour. Logs consume storage.

### Log Analysis Patterns

**Governor limit approach:**
```
Search for: "Number of SOQL queries: X out of 100"
Search for: "Number of DML statements: X out of 150"
Search for: "Maximum CPU time: X out of 10000"
Search for: "Number of callouts: X out of 100"
```

**SOQL performance:**
```
Search for: "SOQL_EXECUTE_BEGIN"
Note the elapsed time between BEGIN and END
Flag queries > 500ms
```

**Exception tracking:**
```
Search for: "EXCEPTION_THROWN"
Search for: "FATAL_ERROR"
Trace the stack to find the originating line
```

## Governor Limit Diagnosis

### Common Limit Violations

| Error | Cause | Fix |
|---|---|---|
| `Too many SOQL queries: 101` | SOQL in loop or trigger recursion | Move query before loop; add recursion guard |
| `Too many DML statements: 151` | DML in loop | Collect records, single DML after loop |
| `Maximum CPU time exceeded` | Complex logic, inefficient loops | Optimize algorithms, reduce iterations |
| `Callout from trigger` | HTTP callout in synchronous trigger | Move to @future or Queueable |
| `Too many future calls: 51` | Chaining futures in loops | Use Queueable (chainable) instead |
| `MIXED_DML_OPERATION` | DML on setup + non-setup in same tx | Separate into @future for setup objects |
| `Heap size exceeded: 12MB` | Large collections or response bodies | Process in chunks, use SOQL FOR loop |
| `RegEx too complicated` | Complex regex pattern | Simplify pattern, use multiple passes |

### Diagnostic Queries

```sql
-- Recent Apex failures
SELECT Id, AsyncApexJobId, Message, StackTrace, ExceptionType
FROM ApexLog WHERE Operation LIKE '%ERROR%' ORDER BY StartTime DESC LIMIT 20

-- Governor limit usage by class
SELECT ApexClass.Name, COUNT(Id), AVG(DurationMilliseconds)
FROM ApexLog GROUP BY ApexClass.Name ORDER BY COUNT(Id) DESC
```

## SOQL Performance Analysis

### Query Plan

```bash
# Get query plan (via Tooling API)
sf data query -o <org-alias> -q "SELECT Id, Name FROM Account WHERE Custom_Field__c = 'value'" --use-tooling-api --query-plan
```

**What to look for:**
- `leadingOperationType`: Should be `Index` not `TableScan`
- `relativeCost`: Lower is better; > 1.0 means SF chose table scan
- `sobjectCardinality`: Total records in object
- `fields`: Which index (if any) was used

### Selectivity Rules

A query is selective if the filter returns < 10% of total records (or < 333,333 for standard index, < 100,000 for custom index):

- **Indexed fields:** Id, Name, OwnerId, CreatedDate, SystemModstamp, RecordType, lookup fields, External ID fields, custom fields marked as unique or External ID
- **Custom indexes:** Request via Salesforce Support for high-volume objects
- **Skinny tables:** Request for large objects (> 2M records) to denormalize frequently queried fields

### Anti-patterns

- `LIKE '%keyword%'` — leading wildcard = table scan always
- `NOT IN (...)` — often non-selective
- `OR` with non-indexed fields — forces table scan
- `NULLS LAST` / `NULLS FIRST` — can prevent index usage
- Missing `WHERE` clause on large objects

## Async Job Monitoring

```sql
-- Batch Apex status
SELECT Id, JobType, Status, NumberOfErrors, TotalJobItems, JobItemsProcessed,
       ExtendedStatus, CreatedBy.Name, CompletedDate
FROM AsyncApexJob WHERE JobType = 'BatchApex' ORDER BY CreatedDate DESC LIMIT 10

-- Queueable status
SELECT Id, Status, NumberOfErrors, ExtendedStatus
FROM AsyncApexJob WHERE JobType = 'Queueable' ORDER BY CreatedDate DESC LIMIT 10

-- Scheduled jobs
SELECT Id, CronJobDetail.Name, State, NextFireTime, PreviousFireTime
FROM CronTrigger WHERE CronJobDetail.JobType = '7' ORDER BY NextFireTime
```

## Error Pattern Reference

| Error Message | Category | Quick Fix |
|---|---|---|
| `DUPLICATE_VALUE` | Data | Check unique fields, duplicate rules |
| `ENTITY_IS_DELETED` | Data | Record deleted between query and DML |
| `FIELD_CUSTOM_VALIDATION_EXCEPTION` | Config | Check validation rules on the object |
| `INSUFFICIENT_ACCESS_ON_CROSS_REFERENCE_ENTITY` | Security | Sharing/FLS issue on related record |
| `LIMIT_EXCEEDED` | Governor | See governor limits table above |
| `UNABLE_TO_LOCK_ROW` | Concurrency | Record locked by another transaction; retry |
| `STRING_TOO_LONG` | Data | Value exceeds field length |
| `REQUIRED_FIELD_MISSING` | Data/Config | Check required fields and page layouts |
