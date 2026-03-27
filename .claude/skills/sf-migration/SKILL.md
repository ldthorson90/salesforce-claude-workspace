---
name: sf-migration
description: Plan and execute Salesforce data migrations — field mapping, ETL patterns, Data Loader configs, Bulk API, reference resolution, and validation.
user-invocable: true
---

# Salesforce Data Migration

Plan, execute, and validate data migrations into Salesforce.

## Arguments

- `plan <source-description>` — create a migration plan
- `mapping <source-object> <target-object>` — create field mapping document
- `template <ObjectName> <org-alias>` — generate CSV template from org schema
- `validate <ObjectName> <org-alias>` — run post-migration validation queries
- `rollback <ObjectName>` — generate rollback strategy

## Migration Planning

### Step 1: Source Analysis
- What system is the data coming from?
- How many records per object?
- What's the data quality? (duplicates, nulls, format inconsistencies)
- Are there relationships that need to be preserved? (parent-child, lookups)
- What's the migration window? (big bang vs phased)

### Step 2: Object Loading Order

Load objects in dependency order (parents before children):

```
1. Users / Queues (if needed for record ownership)
2. Accounts (or whatever the top-level parent is)
3. Contacts (related to Accounts)
4. Opportunities (related to Accounts)
5. Custom child objects
6. Junction objects (after both parents exist)
7. Activities (Tasks, Events — related to any object)
8. Attachments / Files (related to any object)
```

**Key rule:** Load parents FIRST so External ID lookups resolve for children.

### Step 3: External ID Strategy

Every migrated record needs a way to match between source and target:

- **Add External ID fields** to each target object: `Legacy_Id__c` (Text, External ID, Unique)
- Use `upsert` with External ID to make migrations idempotent (re-runnable)
- For lookups: reference the parent's External ID, not Salesforce ID

```
Source: Account.LegacyAccountId → Target: Account.Legacy_Id__c
Source: Contact.LegacyAccountId → Target: Contact.Account.Legacy_Id__c (relationship via External ID)
```

## Field Mapping Document

### Format
```markdown
# Field Mapping: <Source> → <Target Object>

| Source Field | Source Type | Target Field | Target Type | Transform | Notes |
|---|---|---|---|---|---|
| acct_name | varchar(100) | Name | Text(255) | Direct | Required |
| acct_type | int | Type | Picklist | Lookup table | Map codes to SF values |
| created_dt | datetime | Legacy_Created__c | DateTime | Format | ISO 8601 |
| phone | varchar(20) | Phone | Phone | Clean | Remove non-digits |
| | | Legacy_Id__c | Text(50) | Source PK | External ID |
```

### Common Transforms
- **Direct:** No transformation needed
- **Lookup table:** Map source values to Salesforce picklist values
- **Format:** Date/time format conversion
- **Clean:** Remove special characters, trim whitespace
- **Concatenate:** Combine multiple source fields
- **Default:** Set a default value when source is null
- **Skip:** Source field has no target (document why)

## CSV Template Generation

When an org alias is provided, query the target object schema:

```bash
sf data query -o <org-alias> -q "SELECT QualifiedApiName, DataType, IsRequired, Length, ReferenceTo FROM FieldDefinition WHERE EntityDefinition.QualifiedApiName = '<Object>' AND IsCreatable = true" --json
```

Generate a CSV with:
- Column headers matching API names
- A sample row with data type hints
- Required fields marked

## Data Loader Configuration

### For < 50,000 records: Data Loader GUI or CLI
```bash
sf data import bulk -o <org-alias> -f data.csv -s <Object> --external-id Legacy_Id__c
```

### For > 50,000 records: Bulk API 2.0
```bash
sf data import bulk -o <org-alias> -f data.csv -s <Object> --external-id Legacy_Id__c --api-version 62.0
```

### For > 5,000,000 records: Bulk API 2.0 with chunking
Split files into 10MB chunks, process sequentially.

### Settings
- Batch size: 200 (default) — increase to 2,000 for simple objects
- Serial mode for objects with complex triggers/flows
- Disable triggers/flows during migration if possible (via Custom Permission or Custom Metadata toggle)

## Validation Queries

Run after each object load:

```sql
-- Record count match
SELECT COUNT() FROM <Object> WHERE Legacy_Id__c != null

-- Null check on required fields
SELECT Id, Legacy_Id__c FROM <Object> WHERE Required_Field__c = null AND Legacy_Id__c != null

-- Orphan check (children without parents)
SELECT Id, Legacy_Id__c FROM Contact WHERE AccountId = null AND Legacy_Id__c != null

-- Duplicate check
SELECT Legacy_Id__c, COUNT(Id) FROM <Object> GROUP BY Legacy_Id__c HAVING COUNT(Id) > 1

-- Sample spot check (compare source to target)
SELECT Id, Name, Legacy_Id__c, Field1__c, Field2__c FROM <Object> WHERE Legacy_Id__c IN ('LEGACY001', 'LEGACY002', 'LEGACY003')
```

## Rollback Strategy

Before migration:
- Document which records will be created (new) vs updated (existing)
- For updates: export current values BEFORE overwriting (backup CSV)
- For inserts: record all new Salesforce IDs for potential mass delete

Rollback options:
1. **Delete inserted records:** `sf data delete bulk -o <org> -f rollback_ids.csv -s <Object>`
2. **Restore updated records:** Re-import the backup CSV
3. **Sandbox refresh:** If migration is in sandbox, refresh from production

## Gotchas

- **Triggers fire during migration** — disable or add bypass logic
- **Validation rules block bad data** — consider temporary deactivation or bypass permission
- **Workflow rules may fire** — audit and deactivate if needed
- **Sharing recalculation** — defer sharing calc for large loads via Setup
- **API limits** — Bulk API has 15,000 batches per 24h rolling window
- **Attachments vs Files** — ContentVersion (Files) is the modern approach; Attachment is legacy
