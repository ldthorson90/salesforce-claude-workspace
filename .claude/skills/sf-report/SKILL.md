---
name: sf-report
description: Design Salesforce reports, dashboards, custom report types, and analytics — including cross-filters, bucket fields, and dynamic dashboards.
user-invocable: true
---

# Salesforce Reports & Dashboards

Design and build reporting solutions in Salesforce.

## Arguments

- `report <ObjectName> <description>` — design a report
- `dashboard <name>` — design a dashboard layout
- `report-type <name>` — create a custom report type
- `strategy <requirements>` — design a reporting strategy for a project

## Report Types

### Standard vs Custom Report Types

Use **standard report types** when:
- Reporting on a single object or standard object relationships
- The built-in joins cover your needs

Create a **custom report type** when:
- You need a specific object relationship not available in standard types
- You need to include records WITHOUT related records (left outer join)
- You need to limit which fields are available to report builders

### Custom Report Type Design

```
Primary Object: Account
  └── Related Object: Opportunity (A with or without B)
       └── Related Object: OpportunityLineItem (B with or without C)
```

**"With or without" is the key choice:**
- "A with B" = INNER JOIN (only shows A records that have B)
- "A with or without B" = LEFT OUTER JOIN (shows ALL A records, B columns null if no match)

**Rules:**
- Max 4 object levels deep (A → B → C → D)
- Choose "with or without" when users need to find records MISSING related data
- Name: `<Primary>_with_<Related>` (e.g., `Accounts_with_Opportunities`)
- Category: match the primary object's category

## Report Design

### Report Formats

| Format | Use When |
|---|---|
| Tabular | Simple list, no grouping needed |
| Summary | Group by 1-3 fields, subtotals |
| Matrix | Cross-tabulate two groupings (rows × columns) |
| Joined | Combine multiple report blocks (different report types) |

### Filters

- **Standard filters:** Date range, field value, owner
- **Cross-filters:** "Accounts WITH Opportunities" or "Accounts WITHOUT Cases" — powerful for finding gaps
- **Row-level formula filters:** `Amount > 10000 AND StageName = "Negotiation"`
- **Bucket fields:** Group continuous values into categories (e.g., Deal Size: Small/Medium/Large)
- **Filter logic:** AND/OR combinations with up to 25 conditions

### Summary Formulas

```
Win Rate: WON:SUM / RowCount
Average Deal Size: Amount:SUM / WON:SUM
Days to Close: CloseDate - CreatedDate (requires custom formula field)
```

### Performance Tips

- Limit date range (reports timing out = too much data)
- Use indexed fields in filters (Id, Name, CreatedDate, SystemModstamp, RecordType, Division, custom indexed fields)
- Avoid "contains" filters on large text fields
- Max 2,000 rows displayed (use async export for more)
- Summary/Matrix formulas evaluate on displayed rows only

## Dashboard Design

### Layout Principles

- Max 20 components per dashboard (aim for 8-12)
- Group related metrics visually
- Use consistent color coding across components
- Put KPIs (big numbers) at the top
- Trend charts in the middle
- Detail tables at the bottom

### Component Types

| Component | Best For |
|---|---|
| Chart (bar/line/pie) | Trends, comparisons, distributions |
| Gauge | Single metric against target |
| Metric | Single number (KPI) |
| Table | Top N lists, detail data |
| Funnel | Stage-based conversion (pipeline) |
| Scatter | Correlation between two metrics |

### Dynamic Dashboards

- Show data as the **viewing user** (not the dashboard creator)
- Required for: managers seeing their team's data, reps seeing their own pipeline
- License consideration: limited number of dynamic dashboard viewers per edition
- Alternative: "My" filter on dashboard components

### Dashboard Filters

- Up to 3 filters per dashboard
- Filters apply to ALL components (can't filter individual components)
- Use picklist, lookup, or text fields as filter fields
- Default filter value = what shows on first load

## Metadata Location

- Reports: `force-app/main/default/reports/`
- Dashboards: `force-app/main/default/dashboards/`
- Report Types: `force-app/main/default/reportTypes/`

## Context7 References

- `/damecek/salesforce-documentation-context` — "custom report type dashboard filters cross-filter"
