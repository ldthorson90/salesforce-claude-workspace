---
name: sf-experience
description: Design and build Salesforce Experience Cloud sites — templates, navigation, audiences, guest user security, sharing sets, themes, and SEO.
user-invocable: true
---

# Salesforce Experience Cloud

Build and configure Experience Cloud sites (communities, portals, help centers).

## Arguments

- `design <site-type>` — design a site architecture
  - Types: `customer-portal`, `partner-portal`, `help-center`, `custom`
- `security <site-name>` — design guest user and authenticated user security
- `theme <site-name>` — configure branding and theming
- `component <name>` — create a custom LWC for Experience Cloud

## Site Architecture

### Template Selection

| Template | Best For | Customizability |
|---|---|---|
| Customer Service | Help center, knowledge base, case management | Medium |
| Customer Account Portal | Self-service account management, order history | Medium |
| Partner Central | Partner deal registration, lead distribution | Medium |
| Build Your Own (LWR) | Fully custom sites, marketing sites | Full |
| Aura-based (legacy) | Avoid for new builds | Full (but deprecated pattern) |

**Always use LWR (Lightning Web Runtime) for new sites.** Aura-based sites are legacy.

### Key Architectural Decisions

1. **Authenticated vs Guest access** — what can unauthenticated users see?
2. **Login options** — username/password, social login, SSO, passwordless
3. **Data visibility** — what objects/records are exposed?
4. **Self-registration** — can users create accounts? What profile/permission set?
5. **Delegated admin** — can partner/customer admins manage their own users?

## Security Model (Critical)

### Guest User Security

The guest user profile is the **highest security risk** in Experience Cloud:

- **NEVER** give guest users access to sensitive objects (Account, Opportunity, Contact with PII)
- Guest user sharing rules: explicit, narrow, criteria-based
- Check "Let guest users see other members of this site" — usually OFF
- Public API access: disable unless specifically needed
- Secure guest user record access: enabled by default (keep it on)

### Sharing Sets

Replace OWD/sharing rules for portal users:

```
Sharing Set: CustomerPortal_AccountSharing
  Access Mapping:
    Account.Id = User.AccountId → Read
    Case.AccountId = User.AccountId → Read/Write
    Contact.AccountId = User.AccountId → Read
```

Sharing sets grant access based on the portal user's relationship to records (usually via Account).

### Permission Model

| User Type | Profile | Permission Sets | Typical Access |
|---|---|---|---|
| Guest (unauthenticated) | Site Guest User | None | Knowledge articles, public pages |
| Customer (authenticated) | Customer Community Login | Portal_CaseAccess | Own account, own cases, knowledge |
| Partner (authenticated) | Partner Community | Partner_DealAccess | Shared accounts, deal registration |
| Customer+ (delegated admin) | Customer Community Plus | Portal_Admin | Manage contacts under their account |

### Object Access Checklist

For each object exposed to portal users:
- [ ] OWD set appropriately (Private recommended)
- [ ] Sharing set or sharing rule grants access
- [ ] Profile grants CRUD (minimum necessary)
- [ ] FLS restricts sensitive fields
- [ ] Record types filter visible records
- [ ] Page layouts hide internal fields

## Site Configuration

### Navigation

- Keep navigation simple (5-7 top-level items max)
- Use audience targeting to show/hide nav items by user type
- Required pages: Home, My Cases, Knowledge, Profile, Contact Us
- Partner portals add: Deals, Leads, Reports

### Audiences

Audiences control component/page visibility:

```
Audience: Authenticated_Customers
  Criteria: User.Profile.Name = 'Customer Community Login User'

Audience: Gold_Partners
  Criteria: User.Contact.Account.Partner_Level__c = 'Gold'
```

Use audiences for:
- Showing/hiding navigation items
- Swapping page content by user type
- Feature gating (e.g., premium features for higher tiers)

### SEO (for public-facing sites)

- Set page titles and meta descriptions on each page
- Enable SEO-friendly URLs (human-readable slugs)
- Create a sitemap (`/sitemap.xml` auto-generated)
- Set canonical URLs for duplicate content prevention
- robots.txt: allow search engines for public pages, block authenticated areas

### Branding / Theme

- Use Experience Builder theme panel for colors, fonts, images
- CSS overrides: `siteAssets/` directory for custom CSS
- Header/Footer: custom LWC components for full control
- Responsive: all LWR sites are mobile-responsive by default

## Custom LWC for Experience Cloud

Experience Cloud components follow the same LWC model but with extra considerations:

```xml
<!-- myComponent.js-meta.xml -->
<LightningComponentBundle>
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightningCommunity__Page</target>
        <target>lightningCommunity__Default</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightningCommunity__Default">
            <property name="heading" type="String" default="Welcome"/>
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

**Rules:**
- Use `lightningCommunity__Page` and `lightningCommunity__Default` targets
- Expose configurable properties via `targetConfigs` (admins configure in Experience Builder)
- Use `@wire` with `CurrentPageReference` for URL parameters
- Guest user context: test all components as unauthenticated user
- Navigation: use `NavigationMixin` with `comm__namedPage` for internal links

## Metadata Location

- Site definitions: `force-app/main/default/sites/`
- Network (community): `force-app/main/default/networks/`
- Experiences: `force-app/main/default/experiences/`
- Navigation menus: `force-app/main/default/navigationMenus/`
