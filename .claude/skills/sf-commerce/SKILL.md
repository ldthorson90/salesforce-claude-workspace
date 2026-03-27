---
name: sf-commerce
description: Design and implement Salesforce B2C Commerce (Commerce Cloud) — storefront architecture, catalog management, checkout, OCAPI/SCAPI, cartridges, and PWA Kit. Aligned to B2C Commerce Architect certification.
user-invocable: true
---

# Salesforce B2C Commerce

Design and build B2C Commerce (formerly Demandware/SFCC) storefronts.

## Arguments

- `design <requirements>` — design a commerce architecture
- `storefront <name>` — configure storefront settings
- `catalog <name>` — design catalog/product structure
- `checkout` — design checkout flow
- `integration <type>` — design commerce integration (OMS, ERP, payment)
- `cartridge <name>` — design a custom cartridge

## Architecture Overview

```
Customer → CDN → Storefront (SFRA or PWA Kit)
                     │
                     ├── OCAPI/SCAPI (APIs)
                     ├── Controllers (server-side logic)
                     ├── Templates (ISML or React)
                     ├── Models (business objects)
                     └── Cartridges (modular code packages)
                              │
                              └── Business Manager (admin)
                                   ├── Catalogs & Products
                                   ├── Promotions & Campaigns
                                   ├── Content Slots & Assets
                                   └── Site Preferences
```

### Storefront Reference Architecture (SFRA) vs PWA Kit

| Aspect | SFRA | PWA Kit |
|---|---|---|
| Rendering | Server-side (ISML) | Client-side (React) |
| API | Controller-based | SCAPI (headless) |
| Performance | Good | Excellent (SPA) |
| Customization | Cartridge overlay | React components |
| SEO | Built-in SSR | Requires SSR setup |
| Mobile | Responsive | Progressive Web App |
| Future direction | Maintenance mode | Active development |

**For new projects: prefer PWA Kit** unless the client has heavy SFRA investment.

## Catalog Design

### Hierarchy
```
Master Catalog (owns products)
  └── Category: Men's Clothing
      ├── Subcategory: Shirts
      │   ├── Product: Oxford Button-Down
      │   │   ├── Variation Group: Color (White, Blue, Pink)
      │   │   └── Variation Group: Size (S, M, L, XL)
      │   │       └── Variants: 12 SKUs (3 colors × 4 sizes)
      │   └── Product: Casual T-Shirt
      └── Subcategory: Pants

Storefront Catalog (assigns products to site)
  └── Maps products from Master Catalog
  └── Controls navigation and category pages
```

### Product Types

| Type | Use Case |
|---|---|
| Standard Product | Single SKU, no variations |
| Variation Master | Parent with color/size variants |
| Product Set | Bundle sold as one (outfit, kit) |
| Product Bundle | Components sold together at bundle price |
| Option Product | Base product + configurable add-ons |

### Product Attributes
- System attributes: name, ID, brand, price, searchable flag
- Custom attributes: site-specific fields (material, care instructions)
- Attribute groups: organize attributes on PDP
- Localized attributes: name/description per locale

## Checkout Flow

### Standard Checkout Steps
```
Cart → Shipping Address → Shipping Method → Payment → Order Review → Confirmation
```

### Design Decisions

| Decision | Options |
|---|---|
| Guest vs Registered | Guest checkout required for conversion, registration optional |
| Address validation | Real-time via service (SmartyStreets, Google) |
| Payment methods | Credit card, PayPal, Apple Pay, Google Pay, gift cards |
| Tax calculation | Native or external (Avalara, Vertex) |
| Shipping | Table rates, real-time carrier (FedEx, UPS), store pickup |
| Order splitting | Ship from multiple locations (requires OMS integration) |

### Payment Integration
```
Payment Processor Integration:
  1. Create payment cartridge (or use existing: Adyen, Stripe, PayPal)
  2. Implement hooks: authorize, capture, refund, void
  3. Configure in Business Manager: payment methods, processors
  4. PCI compliance: use hosted payment fields (iframe) for PCI SAQ-A
```

## APIs

### OCAPI (Open Commerce API) — Legacy
```
GET /s/{site_id}/dw/shop/v24_5/products/{product_id}
Authorization: Bearer {access_token}
```
- Shop API: customer-facing (products, cart, checkout)
- Data API: admin operations (catalogs, orders, content)
- REST-based, JSON responses
- Being superseded by SCAPI

### SCAPI (Salesforce Commerce API) — Modern
```
GET /commerce/shopper/products/v1/products/{productId}?siteId={siteId}
Authorization: Bearer {access_token}
```
- Shopper APIs: headless commerce operations
- Admin APIs: backend management
- PWA Kit built on SCAPI
- Better performance, modern auth (SLAS)

### SLAS (Shopper Login and API Access Service)
- OAuth 2.0 based authentication for shoppers
- Guest tokens, registered user tokens, refresh tokens
- PKCE flow for public clients (PWA)

## Cartridge Architecture

### Overlay Pattern
```
Cartridge Path: app_custom : app_storefront_base
  (custom)          (SFRA base)

Custom code in app_custom OVERRIDES base by:
  - Same file path = replaces the base file
  - New file path = extends functionality
  - Controller extension = wrap base controller logic
```

### Cartridge Types

| Type | Purpose |
|---|---|
| app_storefront_base | SFRA core (never modify directly) |
| app_custom | Site-specific overrides |
| int_* | Integration cartridges (payment, tax, shipping) |
| bc_* | Business component cartridges |
| bm_* | Business Manager extensions |
| plugin_* | Feature plugins (wishlist, gift registry) |

### Best Practices
- Never modify base cartridges — always overlay
- One cartridge per integration (int_adyen, int_avalara)
- Keep the cartridge path minimal (performance impact)
- Use controller middleware for cross-cutting concerns (logging, auth)

## Promotions & Campaigns

### Promotion Types
- **Product promotions:** % off, fixed amount off, BOGO, tiered discounts
- **Order promotions:** free shipping, $ off order total, gift with purchase
- **Shipping promotions:** free/discounted shipping by method

### Campaign Structure
```
Campaign: Summer_Sale_2026
  Start: 2026-06-01 | End: 2026-08-31
  Promotions:
    - 20% off Summer Collection (product)
    - Free shipping over $50 (shipping)
  Customer Groups: All Shoppers
  Qualifiers: Coupon code OR automatic
```

## Performance

### Key Metrics
- **Time to First Byte (TTFB):** < 200ms
- **Largest Contentful Paint (LCP):** < 2.5s
- **Cumulative Layout Shift (CLS):** < 0.1
- **Page weight:** < 1MB compressed

### Optimization
- CDN configuration (edge caching for static assets)
- Page caching: cache full pages for anonymous users
- Component caching: cache expensive ISML includes
- Image optimization: responsive images, WebP, lazy loading
- Script management: defer non-critical JS, minimize third-party scripts

## Site Configuration

### Business Manager
- Site preferences: locale, currency, time zone, default catalog
- URL rules: SEO-friendly URLs, redirects
- Content slots: dynamic content areas (homepage hero, PDP recommendations)
- Content assets: static pages (about, privacy, terms)
- Customer groups: segment shoppers for promotions/content
