---
description: "Apex development conventions, SOQL patterns, and code analysis gates"
globs: ["**/*.cls", "**/*.trigger", "**/*.apex"]
---

# Apex Development Standards

## Code Style
- 4-space indentation, K&R brace style
- Explicit access modifiers on all members
- Trigger + handler pattern (thin trigger delegates to handler class)

## Bulkification
- No SOQL or DML inside loops, use collections
- `USER_MODE` for CRUD/FLS enforcement
- Static recursion guards in trigger handlers

## Testing
- `@TestSetup` for shared test data
- Bulk tests (200+ records), positive, negative, and boundary cases
- Test classes named `<ClassName>Test`

## SOQL
- Always use bind variables (never string concatenation for dynamic SOQL)
- Selective queries: indexed fields in WHERE clauses
- Limit large result sets; use FOR loops for bulk processing

## Code Analysis Gate
- Run Code Analyzer before deployment: `sf scanner:run --target <path> --format table`
- ApexGuru for Apex-specific anti-pattern detection (via MCP)
- Zero Critical/High violations required before delivery

## Test Execution
Per-project. Each SFDX project defines in its own CLAUDE.md:
```bash
sf apex run test --test-level RunLocalTests --result-format human
```
