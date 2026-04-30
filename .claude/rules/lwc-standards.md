---
description: "Lightning Web Component development conventions"
globs: ["**/lwc/**/*.js", "**/lwc/**/*.html", "**/lwc/**/*.css"]
---

# LWC Development Standards

## Code Style
- 2-space indentation
- Wire service for data access; minimize imperative Apex calls
- Proper `@api`, `@wire`, `@track` decorators

## Templates
- `lwc:if` / `lwc:elseif` / `lwc:else` (not legacy `if:true` / `if:false`)
- No ternary operators in templates (unsupported)
- SLDS utility classes for styling

## Error Handling
- `ShowToastEvent` for user-facing errors
- Try/catch on all imperative Apex calls

## Linting
- ESLint with `@lwc/eslint-plugin-lwc`
