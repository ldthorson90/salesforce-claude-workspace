---
name: sf-layout
description: Design and configure Salesforce page layouts, Lightning Record Pages, Dynamic Forms, record types, and compact layouts.
user-invocable: true
---

# Salesforce Layout Configuration

Design and build UI configurations for Lightning Experience.

## Arguments

- `page <ObjectName>` — create a Lightning Record Page for an object
- `layout <ObjectName>` — design a page layout
- `dynamic-form <ObjectName>` — convert a page layout to Dynamic Forms
- `record-type <ObjectName> <TypeName>` — create a record type with layout assignment

## Lightning Record Pages

### Design Principles
- Lead with the most important information (summary fields, key metrics)
- Use tabs to organize sections (Details, Related, Activity, Chatter)
- Limit visible components to 6-8 per tab (cognitive load)
- Use conditional visibility to show/hide based on record type, field values, or permissions
- Mobile: single column, most critical info first

### Component Selection Guide

| Need | Component | Notes |
|---|---|---|
| Field display/edit | Record Detail (Dynamic Form) | Preferred over standard detail |
| Related records | Related List - Single | One list per component, filterable |
| Multiple related lists | Related Lists | Grouped, less configurable |
| Highlights | Highlights Panel | Key fields at top of page |
| Quick actions | Actions & Recommendations | Context-aware action bar |
| Visual summary | Rich Text / Custom LWC | For dashboards or status displays |
| Path/stages | Path | For opportunity stages, case status |
| Activity | Activity / Activity Timeline | Emails, tasks, events, calls |

### Dynamic Forms

**Always prefer Dynamic Forms over standard page layouts for supported objects.**

Benefits:
- Field-level visibility rules (show/hide fields based on criteria)
- Flexible field placement (not locked to 2-column grid)
- Field sections with conditional visibility
- Better mobile responsive behavior

Conversion process:
1. Navigate to Lightning App Builder for the object's record page
2. Click "Upgrade Now" on the Record Detail component
3. Drag fields into sections with visibility filters
4. Set visibility rules per field or section:
   - Field value conditions (e.g., show Discount__c only when Amount > 10000)
   - Record type conditions
   - Permission-based conditions
   - Device-based (desktop vs mobile)

**Supported objects:** Account, Contact, Opportunity, Lead, Case, Campaign, Custom Objects, Person Accounts, and most standard objects. NOT supported: Task, Event, User.

### Record Types

When to create record types:
- Different business processes on the same object (e.g., B2B vs B2C Account)
- Different page layouts per type
- Different picklist value sets per type
- Different compact layouts per type

**Rules:**
- Name: PascalCase, descriptive (e.g., `B2B_Account`, `Support_Case`)
- Always set a default record type
- Assign record types to profiles/permission sets
- Map each record type to a page layout
- Consider the impact on reporting (record type is a standard filter)

### Compact Layouts

- 10 field limit (first field becomes the object name in lookups)
- Choose fields that uniquely identify the record at a glance
- Used in: Kanban, record highlights, lookup cards, mobile headers
- Create per record type if different views are needed

### Page Layout Best Practices

If Dynamic Forms aren't available:
- Put required fields at the top
- Group related fields in sections with descriptive headers
- Use 2-column layout, left column for primary data, right for secondary
- Related lists: most frequently used at the top
- Limit to 4-6 sections maximum
- Use blank spaces intentionally (don't cram every field)

## Metadata Format

Page layouts are in `force-app/main/default/layouts/`
Lightning pages are in `force-app/main/default/flexipages/`
Record types are in `force-app/main/default/objects/<Object>/recordTypes/`

## Context7 References

- `/websites/v1_lightningdesignsystem` — SLDS form layouts, components
- `/damecek/salesforce-documentation-context` — "Dynamic Forms Lightning Record Page"
