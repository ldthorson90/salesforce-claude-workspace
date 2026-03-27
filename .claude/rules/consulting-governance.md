# Consulting Governance Rules

## Org Safety

- NEVER deploy metadata directly to a Production org. All AI-generated work targets
  Developer Sandboxes or Scratch Orgs exclusively.
- Before any destructive metadata operation (delete, overwrite), confirm the target org
  alias and verify it is not a production org.
- When connecting to a new org, always run `sf org display` and check the `OrgType` field.
  If it shows "Production", refuse deploy/push operations and warn the user.

## Architectural Challenge Pattern

Do not blindly implement requirements. When the user describes a feature or design:
1. Check the architect decision guides in `salesforce-architect-decision-guides/` for
   relevant patterns
2. Identify governor limit risks, data volume concerns, or pattern mismatches
3. Propose alternatives if the requested approach has known pitfalls
4. Only proceed with implementation after the approach is confirmed

This is especially important for: trigger vs. flow decisions, synchronous vs. async
processing, data model design, and integration patterns.

## Client Engagement Hygiene

- Each client engagement gets its own SFDX project directory with its own CLAUDE.md
- Never cross-reference data or metadata between client projects
- Org credentials and aliases are per-project, never workspace-global
- When generating documentation or deliverables, never include org-specific IDs,
  endpoint URLs, or credentials

## Code Review Posture

For consultant-delivered code, apply stricter standards than internal dev:
- All Apex must pass Code Analyzer with zero Critical/High violations
- All triggers must use handler pattern (no logic in trigger body)
- All LWC must use wire service unless imperative calls are justified
- Test classes must cover bulk (200+), positive, negative, and boundary cases
- No hardcoded IDs, URLs, or org-specific values anywhere
