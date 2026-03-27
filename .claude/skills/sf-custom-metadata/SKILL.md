---
name: sf-custom-metadata
description: Design and implement Custom Metadata Types, Custom Settings, Custom Labels, and feature flag patterns for configurable Salesforce solutions.
user-invocable: true
---

# Salesforce Custom Metadata & Configuration

Build configurable solutions using Custom Metadata Types, Custom Settings, and Custom Labels.

## Arguments

- `metadata-type <name>` — create a Custom Metadata Type
- `setting <name>` — create a Custom Setting (hierarchy or list)
- `label <name>` — create a Custom Label
- `feature-flag <name>` — create a feature flag pattern
- `strategy <description>` — design a configuration strategy

## When to Use What

| Mechanism | Deployable | Queryable in SOQL | Per-User/Profile | Use Case |
|---|---|---|---|---|
| Custom Metadata Type | ✅ Yes | ✅ Yes (no limit) | ❌ No | App config, mapping tables, routing rules |
| Hierarchy Custom Setting | ❌ No (data) | ✅ Yes (cached) | ✅ Yes | Per-user/profile overrides, feature flags |
| List Custom Setting | ❌ No (data) | ✅ Yes (cached) | ❌ No | Legacy — use Custom Metadata instead |
| Custom Labels | ✅ Yes | ❌ (use System.Label) | ❌ No | UI strings, error messages, translatable text |
| Custom Permissions | ✅ Yes | ❌ (use FeatureManagement) | ✅ Yes (via permset) | Feature gating, bypass flags |

**Default choice: Custom Metadata Types.** They're deployable, SOQL-queryable, don't count against governor limits for reads, and work in validation rules and formulas.

## Custom Metadata Types

### Design Pattern

```xml
<!-- MyConfig__mdt -->
<CustomObject>
    <label>My Config</label>
    <pluralLabel>My Configs</pluralLabel>
    <visibility>Public</visibility>
    <fields>
        <fullName>Value__c</fullName>
        <type>Text</type>
        <length>255</length>
    </fields>
    <fields>
        <fullName>Is_Active__c</fullName>
        <type>Checkbox</type>
        <defaultValue>true</defaultValue>
    </fields>
</CustomObject>
```

### Common Use Cases

**Field Mapping Table:**
```
Integration_Field_Map__mdt
├── Source_Field__c (Text) — external system field name
├── Target_Field__c (Text) — Salesforce API name
├── Transform__c (Picklist) — Direct, Uppercase, DateFormat, Lookup
├── Is_Required__c (Checkbox)
└── Integration_Name__c (Text) — which integration this mapping belongs to
```

**Trigger Bypass:**
```
Trigger_Setting__mdt
├── Object_Name__c (Text) — API name of the object
├── Is_Active__c (Checkbox) — master switch
├── Disable_Before_Insert__c (Checkbox)
├── Disable_After_Update__c (Checkbox)
└── Disable_Validation__c (Checkbox)
```

**Routing Rules:**
```
Case_Routing__mdt
├── Priority__c (Picklist)
├── Origin__c (Picklist)
├── Queue_Name__c (Text)
├── SLA_Hours__c (Number)
└── Auto_Respond__c (Checkbox)
```

### Querying Custom Metadata

```apex
// No governor limit impact — cached by platform
List<Trigger_Setting__mdt> settings = Trigger_Setting__mdt.getAll().values();

// Or SOQL (also no limit impact for CMT)
List<Integration_Field_Map__mdt> mappings = [
    SELECT Source_Field__c, Target_Field__c, Transform__c
    FROM Integration_Field_Map__mdt
    WHERE Integration_Name__c = :integrationName AND Is_Active__c = true
];

// Single record by DeveloperName
Trigger_Setting__mdt setting = Trigger_Setting__mdt.getInstance('Account');
```

### In Formulas and Validation Rules

```
$CustomMetadata.Trigger_Setting__mdt.Account.Is_Active__c
```

This works in validation rules, formula fields, and flows — making Custom Metadata Types the most versatile configuration mechanism.

## Hierarchy Custom Settings

Use ONLY when you need per-user or per-profile overrides.

```apex
// Returns value for current user > profile > org default (hierarchy)
My_Setting__c setting = My_Setting__c.getInstance();
Boolean isFeatureEnabled = setting.Enable_Feature__c;

// Check for specific user
My_Setting__c userSetting = My_Setting__c.getInstance(userId);
```

**Gotcha:** Custom Settings data is NOT deployable via metadata. You must:
- Set org-wide defaults in post-deploy scripts
- Or use Custom Metadata Types instead (deployable)

## Custom Labels

Best for:
- User-facing error messages (translatable)
- Email template merge fields
- Values that change by language but not by org

```apex
String msg = System.Label.Error_Required_Field;
// In LWC:
import LABEL from '@salesforce/label/c.Error_Required_Field';
```

Metadata: `force-app/main/default/labels/CustomLabels.labels-meta.xml`

## Feature Flag Pattern

Combine Custom Permission + Permission Set for feature gating:

```apex
// Check if current user has the feature enabled
if (FeatureManagement.checkPermission('Enable_New_Dashboard')) {
    // Show new dashboard
}
```

```xml
<!-- In validation rules / flows -->
$Permission.Enable_New_Dashboard
```

**Deployment pattern:**
1. Deploy Custom Permission: `Enable_New_Dashboard`
2. Deploy Permission Set: `New_Dashboard_Access`
3. Assign permission set to pilot users
4. Code checks `$Permission.Enable_New_Dashboard` before showing feature
5. To roll out: assign permission set to all users
6. To remove: delete permission set assignment

This is the Salesforce-native equivalent of feature flags without external tooling.

## Anti-Patterns

- **Hardcoding values in Apex** — use Custom Metadata or Custom Labels instead
- **Using List Custom Settings for new development** — use Custom Metadata Types
- **Storing credentials in Custom Settings** — use Named Credentials
- **Creating a Custom Metadata Type for one value** — use a Custom Label or Custom Permission
- **Not including `Is_Active__c` on Custom Metadata** — always include an active/inactive toggle
