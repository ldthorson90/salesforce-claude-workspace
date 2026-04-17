---
description: "Flow development conventions and naming standards"
globs: ["**/*.flow-meta.xml"]
---

# Flow Development Standards

## Naming
- Pattern: `<Object>_<Action>_<Context>` (e.g., `Account_Update_AfterInsert`)

## Design
- Prefer record-triggered flows over process builders
- Use subflows for reusable logic
- Fault paths on every DML and callout element

## Review Checklist
- No hardcoded IDs or record type names
- Bulkified: handles collections, not single records
- Entry criteria are selective (avoid running on every update)
