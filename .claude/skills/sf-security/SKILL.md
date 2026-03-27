---
name: sf-security
description: Design and implement Salesforce security model — OWD, sharing rules, profiles, permission sets, FLS, and record-level access. Audit existing security posture.
user-invocable: true
---

# Salesforce Security Model

Design, implement, or audit the security model for a Salesforce org.

## Arguments

- `design` — design a security model for a new implementation
- `audit <org-alias>` — audit an existing org's security posture
- `permset <name>` — create a permission set for a specific use case
- `sharing <object>` — design sharing rules for an object

## Security Model Design Workflow

### Step 1: Gather Requirements

Ask the user about:
- User roles/personas and what data each needs to access
- Sensitive objects/fields requiring restricted access
- Record ownership model (territory, role hierarchy, manual)
- External user access (communities, portals)
- Compliance requirements (HIPAA, SOX, PCI)

### Step 2: OWD (Organization-Wide Defaults)

Start with the MOST RESTRICTIVE setting per object, then open up with sharing rules:

| Access Level | When to Use |
|---|---|
| Private | Default. Users see only their own records |
| Read Only | Users can see all but edit only their own |
| Public Read/Write | All users can see and edit all records |
| Controlled by Parent | Detail in master-detail; inherits parent sharing |

**Rules:**
- NEVER set OWD to Public Read/Write for objects containing sensitive data
- Master-Detail child objects MUST use Controlled by Parent
- Start Private, justify every relaxation

### Step 3: Role Hierarchy

- Design from the bottom up (individual contributors first)
- Roles grant upward visibility — a manager sees their reports' records
- Keep the hierarchy flat (3-4 levels max for performance)
- Do NOT use roles as a substitute for profiles or permission sets

### Step 4: Sharing Rules

When OWD is Private but specific groups need access:

- **Criteria-Based Sharing:** Share records matching field criteria (e.g., Region__c = 'West')
- **Owner-Based Sharing:** Share records owned by a role/group with another role/group
- **Apex Managed Sharing:** Programmatic sharing for complex rules (requires `without sharing` — document why)
- **Guest User Sharing Rules:** For Experience Cloud — never share more than necessary

### Step 5: Permission Sets over Profiles

**Always prefer Permission Sets and Permission Set Groups over Profiles:**
- Profiles: assign ONE per user for object CRUD baseline + login settings + page layout assignment
- Permission Sets: stack additional permissions as needed
- Permission Set Groups: bundle related permission sets for role-based assignment
- Muting Permission Sets: remove specific permissions within a group

**Anti-patterns to flag:**
- Cloning the System Administrator profile
- Granting "Modify All Data" or "View All Data" outside admin profiles
- Using profiles for field-level security (use permission sets instead)
- More than 5-6 profiles in an org (signals profile abuse)

### Step 6: Field-Level Security

- Default: fields are hidden until explicitly granted via permission set
- Sensitive fields (SSN, salary, health data): restrict to minimum necessary permission sets
- Formula fields: check that referenced fields are also accessible
- Encrypted fields: require "View Encrypted Data" permission

### Step 7: Generate Artifacts

Output:
- OWD settings table (Object | Internal | External)
- Role hierarchy diagram (via Mermaid)
- Permission set matrix (Permission Set | Objects | Fields | System Permissions)
- Sharing rules table (Object | Type | Share From | Share To | Access Level)

## Audit Workflow

When auditing an existing org:
1. Query OWD settings via Setup or Metadata API
2. Query permission sets and their assignments
3. Query sharing rules per object
4. Flag: Modify All Data grants, Public Read/Write on sensitive objects, empty permission sets, unused profiles
5. Generate security posture report with recommendations

## Context7 References

For Salesforce security documentation, query:
- `/damecek/salesforce-documentation-context` — "sharing rules OWD permission sets"
- `/websites/architect_salesforce_well-architected` — "data security authorization"
