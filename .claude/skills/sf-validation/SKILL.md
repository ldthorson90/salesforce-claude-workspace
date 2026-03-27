---
name: sf-validation
description: Create Salesforce validation rules and formula fields with proper error handling, cross-object references, and null safety.
user-invocable: true
---

# Salesforce Validation Rules & Formula Fields

Build validation rules and formula fields with correct syntax and edge case handling.

## Arguments

- `rule <ObjectName> <description>` — create a validation rule
- `formula <ObjectName> <FieldName> <description>` — create a formula field
- `audit <ObjectName>` — review existing validation rules for conflicts and gaps

## Validation Rules

### Structure
```xml
<ValidationRule>
    <fullName>Object__c.Rule_Name</fullName>
    <active>true</active>
    <description>Business justification for this rule</description>
    <errorConditionFormula>/* formula that returns TRUE to block save */</errorConditionFormula>
    <errorDisplayField>Field_Name__c</errorDisplayField>
    <errorMessage>User-friendly error message explaining what to fix</errorMessage>
</ValidationRule>
```

### Best Practices

1. **Name convention:** `<Object>_<What_It_Validates>` (e.g., `Opp_Require_Close_Date_When_Won`)
2. **Always include a description** explaining the business rule, not just the formula
3. **Error message:** Tell the user WHAT to fix, not just that something is wrong
4. **errorDisplayField:** Point to the specific field that needs correction
5. **Null safety:** ALWAYS check for null before comparing
6. **Active by default:** Deploy active. Use metadata to deactivate in sandboxes if needed

### Common Patterns

**Require field when status changes:**
```
AND(
  ISPICKVAL(Status__c, 'Closed Won'),
  ISBLANK(Close_Date__c)
)
```

**Prevent backdating:**
```
AND(
  ISCHANGED(Start_Date__c),
  Start_Date__c < TODAY(),
  NOT($Permission.Can_Backdate_Records)
)
```

**Cross-object validation:**
```
AND(
  ISCHANGED(Amount__c),
  Account.Is_Locked__c = TRUE
)
```
Note: Cross-object formulas can only traverse UP (child to parent), not down.

**Regex for email/phone:**
```
AND(
  NOT(ISBLANK(Email__c)),
  NOT(REGEX(Email__c, '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'))
)
```

**Permission bypass:**
```
AND(
  /* validation logic */,
  NOT($Permission.Bypass_Validation)
)
```
Always include an admin bypass via custom permission.

### Gotchas

- **PRIORVALUE:** Only works in before-save triggers and validation rules, NOT in formula fields
- **ISCHANGED:** Only works in validation rules, NOT in formula fields or workflows
- **Null handling:** `ISBLANK()` for text, `ISNULL()` for numbers (but `ISBLANK` works for both — prefer it)
- **Picklist comparison:** Always use `ISPICKVAL()`, never `==` with picklist fields
- **Cross-object limits:** Max 10 unique cross-object references per formula
- **Character limit:** 5,000 characters compiled size (not character count)
- **Record type check:** Use `RecordType.DeveloperName` not `RecordTypeId` (portable across orgs)

## Formula Fields

### Return Types
- Checkbox (Boolean)
- Currency
- Date / Date/Time
- Number
- Percent
- Text

### Common Patterns

**Days since creation:**
```
TODAY() - DATEVALUE(CreatedDate)
```

**Business days between dates:**
```
(5 * (FLOOR((End_Date__c - Start_Date__c) / 7))) +
MIN(5, MOD(End_Date__c - Start_Date__c, 7))
```

**Null-safe concatenation:**
```
TRIM(
  IF(ISBLANK(FirstName), '', FirstName & ' ') &
  IF(ISBLANK(MiddleName), '', MiddleName & ' ') &
  LastName
)
```

**Conditional image/icon:**
```
IF(Health_Score__c >= 80, '🟢',
  IF(Health_Score__c >= 50, '🟡', '🔴'))
```

**Cross-object rollup (formula approach):**
Cannot do true rollups in formulas. Use:
- Roll-Up Summary fields (master-detail only)
- DLRS (Declarative Lookup Rollup Summaries) for lookup relationships
- Flow-based rollups for complex logic

### Gotchas

- **Compile size:** 5,000 char limit. Use helper formula fields to break up complex logic
- **SOQL in formulas:** Not possible. Formulas cannot query data
- **TEXT() on picklists:** Returns API name, not label. Use for comparisons
- **HYPERLINK:** Works in Classic but renders differently in Lightning
- **Treat blanks as:** Choose "Blanks are zeros" or "Blanks are blanks" carefully — this changes math behavior

## Metadata Location

- Validation rules: `force-app/main/default/objects/<Object>/validationRules/`
- Formula fields: `force-app/main/default/objects/<Object>/fields/`
