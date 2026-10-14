# Suruj Enterprise
## Master Technical Specification
**Status:** Implementation baseline — ready for repository setup and development  
**Business:** Suruj Enterprise  
**Primary domain:** `surujenterprise.cc`  
**Organization:** `https://github.com/Suruj-Enterprise`  
**Existing asset repository:** `se-assets`  
**Document version:** 1.0  
**Audience:** Product owner, developers, coding agents, QA, operations, finance/accounting, and future maintainers

---

## 0. How to use this specification

This document is the single implementation baseline for the Suruj Enterprise platform. Treat the requirements marked **MUST** as release blockers unless the owner records an explicit, dated exception. **SHOULD** means strongly recommended; **MAY** means optional.

The platform is designed as a multi-app commerce business with independently deployable frontends and a shared business backend. Build and release in dependency order, but model the complete product from the beginning. “Complete scope” does not mean turning on live payments, advertising, marketplace integrations, or AI before their accounts, credentials, legal checks, and tests are ready.

### Fixed business decisions
- India-based, GST-registered business.
- Suruj Enterprise owns/holds the stock it sells on its main storefront and fulfils its own orders.
- Online payment only. **No cash on delivery.**
- Paytm is the initial payment gateway.
- AdSense is allowed only on the blog app/domain.
- One owner/admin account is authorized to control the business. No public admin sign-up.
- `noreply@surujenterprise.cc` is the intended transactional sender; `support@surujenterprise.cc` is the intended customer-support address.
- Use Unsplash only for decorative/editorial design imagery. Actual product photography is supplied/approved by the owner.
- Start on free tiers where feasible; avoid paid software commitments initially. Gateway, shipping, domain, taxes, inventory, import, and other real business costs remain payable.
- Mobile-first responsive UI, optimized especially for desktop productivity.
- Separate GitHub repositories/apps; shared data model and API contracts.
- A free-tier AI provider may be used only after its quota, commercial-use terms, privacy, and availability are checked. The system must remain useful when AI is unavailable.

### Non-goals / constraints
- Do not promise guaranteed AdSense approval, affiliate approval, marketplace API access, or free unlimited hosting/AI.
- Do not hardcode tax rates, shipping prices, marketplace fees, AI quotas, or payment behavior that can change.
- Do not treat an order as paid based only on a browser redirect.
- Do not allow an LLM to execute arbitrary SQL or autonomously perform high-impact business actions.
- Do not store card/payment credentials. Use Paytm’s supported hosted/redirect/SDK flow.
- Do not use generated or Unsplash imagery to misrepresent actual products.

---

# 1. Product and system overview

## 1.1 Applications and domains

| App | Production host | Purpose | Public? |
|---|---|---|---|
| Storefront | `https://surujenterprise.cc` | Owned inventory, product discovery, cart, checkout, customer orders | Yes |
| Affiliate shop | `https://affiliates.surujenterprise.cc` | Curated third-party products, comparisons, tracked outbound referrals | Yes |
| Editorial blog | `https://blogs.surujenterprise.cc` | Original editorial content, shopping guides, eligible AdSense placements | Yes |
| Admin command center | `https://admin.surujenterprise.cc` | Private operations, catalog, stock, orders, finance, content, settings, AI workspace | Owner only |
| Shared platform | Configurable API origin, e.g. `https://api.surujenterprise.cc` if deployed separately | Authentication integration, business rules, database access, webhooks, jobs | API; protected |
| Assets | Existing `se-assets` repository | Brand logo and approved static assets | Public assets only |

The API hostname is a deployment choice, not a dependency on a particular hosting provider. If Supabase is the primary backend, clients may use Supabase for approved public reads and authenticated operations, but privileged business logic, Paytm callbacks, secret-bearing tasks, and sensitive admin actions MUST run server-side.

## 1.2 Proposed repositories

Under `Suruj-Enterprise`:

1. `se-storefront` — Next.js storefront.
2. `se-affiliates` — affiliate discovery and outbound referrals.
3. `se-blog` — editorial CMS frontend and public blog.
4. `se-admin` — private owner console.
5. `se-platform` — migrations, shared contracts/types, server-side business logic, scheduled jobs, webhook handlers, security policies, architecture decision records.
6. `se-assets` — existing logo and approved public assets; preserve its current ownership and history.
7. `se-docs` — operator manuals, policies, release runbooks, decision records, recovery instructions.

Each app MUST build and deploy independently. Avoid copying business rules between apps. Share versioned API contracts/types through a package or generated schema, not by importing another app’s private implementation. Pin runtime and dependency versions; commit lockfiles.

## 1.3 Recommended technology baseline

- Frontend: Next.js App Router, TypeScript, React.
- Styling: Tailwind CSS plus a documented token-based design system and custom components.
- Validation: Zod or equivalent shared schema validation.
- Database/Auth/Storage: Supabase Postgres, Supabase Auth, Supabase Storage where appropriate.
- Hosting: Netlify free tier initially, provided framework support, build limits, and usage remain suitable. Keep the app deployable elsewhere.
- Source control and CI: GitHub, pull requests, automated checks, secret scanning.
- Email: Resend or an equivalent provider with a suitable free tier, after sender-domain verification.
- Payment: Paytm merchant account and official supported integration.
- Tests: Vitest/Jest for unit tests, Playwright for end-to-end tests, database/RLS tests, accessibility checks.
- Observability: structured server logs, error tracking and uptime checks within free quotas; alert on quota exhaustion.
- AI: provider-adapter interface with configurable provider/model, explicit quotas, safe fallback templates, and no provider lock-in.

Before committing to a provider, verify its current free-plan limits, commercial-use terms, data retention, region, uptime, and inactivity behavior. Free-tier terms change. Record selected versions/providers in `docs/architecture/decisions/`.

---

# 2. Information architecture and navigation

## 2.1 Shared brand and UI principles

Brand direction: confident, useful, editorial retail — not a generic template. The interface should feel intentionally art-directed, trustworthy, and easy to operate.

### Design tokens
- Forest green: `#17251F` — primary brand/action.
- Warm ivory: `#F5F1E8` — page backgrounds and editorial surfaces.
- Paper white: `#FFFFFF` — cards and form surfaces.
- Terracotta: `#B75D3C` — restrained accent.
- Graphite: `#333632` — primary text.
- Muted text and borders: define accessible neutral tokens after contrast testing.
- Typography: Inter or another legible sans-serif for UI; a restrained editorial serif may be used for campaign headlines only.
- Corners: modest rounded cards and controls; avoid excessive pill-shaped everything.
- Motion: subtle 120–220 ms transitions, respecting `prefers-reduced-motion`.
- Images: Unsplash only for decorative/editorial imagery; product images must be owner-supplied or rights-cleared product assets.

### Interaction rules
- Clear hover, focus, active, disabled, loading, empty, error, and success states for every interactive component.
- Keyboard access, visible focus ring, semantic headings, labels, landmarks, and sufficient color contrast.
- No important information conveyed by color alone.
- No fake urgency, fake review counts, fake stock scarcity, invented ratings, or fabricated testimonials.
- Responsive navigation and tables must remain usable on narrow screens; dense admin tables may switch to stacked cards or horizontal scrolling with clear cues.
- Preserve form input on recoverable errors. Confirm destructive or irreversible actions.
- Provide skeletons only where they improve perceived performance; use explicit empty states and retry actions.

## 2.2 Storefront navigation

Primary desktop navigation:
- Shop
- Categories
- New / Featured (only if merchandising rules and content exist)
- Guides (links to blog)
- About
- Search
- Account
- Cart

Mobile navigation:
- Compact header with menu, logo, search, cart.
- Accessible drawer or full-screen menu.
- Persistent cart access without covering checkout controls.
- Filter/sort controls use a mobile-friendly sheet with apply/reset actions.

### Storefront pages

| Route | Purpose and required content |
|---|---|
| `/` | Campaign hero, category discovery, featured products, value/trust information, editorial links, footer |
| `/shop` | All published/available products, search, filters, sort, pagination/infinite loading with accessible fallback |
| `/category/[slug]` | Category description, category product grid, subcategory navigation, filters |
| `/product/[slug]` | Gallery, title, price, tax display policy, variants, stock status, quantity, delivery estimate if reliable, details/specifications, returns/shipping links, related products |
| `/search` | Search results, suggestions, filters, no-result help |
| `/cart` | Items, variants, quantity, availability validation, subtotal, estimated shipping/tax if available, checkout CTA |
| `/checkout` | Contact, delivery address, shipping option, order review, Paytm payment initiation, terms/privacy consent as legally required |
| `/checkout/result` | Pending/verified/failed state with server-verified order status; never assume success from query string |
| `/account` | Profile, saved addresses, order history, wishlist, account security |
| `/account/orders` | Paginated order list |
| `/account/orders/[orderNumber]` | Status timeline, items, invoice, shipment tracking, support/return request |
| `/wishlist` | Saved products, unavailable-item handling |
| `/track-order` | Secure tracking by authenticated account or carefully designed order-number + verification flow; do not expose private order data by guessable number |
| `/help` | Customer support channels, FAQ, order/payment/shipping help |
| `/contact` | Support form and published support email |
| `/about` | Accurate company story and business identity |
| `/shipping-policy` | Regions, handling times, carrier/tracking rules, exclusions |
| `/returns-refunds` | Eligibility, timelines, condition, process, refund statuses |
| `/cancellation-policy` | Cancellation window and limits |
| `/privacy-policy` | Data categories, purposes, retention, rights, contact |
| `/terms` | Customer terms and business disclosures |
| `/payment-policy` | Accepted online methods and failed/pending payment guidance |
| `/gst-information` | Applicable business/invoice disclosures as approved |
| `/accessibility` | Accessibility contact and support path |
| `/404` and global error boundary | Helpful recovery links, no stack traces |

All policy copy is a required deliverable but MUST be reviewed for actual business practice and applicable Indian law before production. Do not publish placeholder legal text.

## 2.3 Affiliate app navigation and pages

Affiliate app is a referral publisher, not a merchant of record for third-party inventory. It must not imply that Suruj Enterprise owns, ships, warranties, or accepts payment for an affiliate product unless that is factually true.

Required routes:
- `/` — curated collections, clear affiliate disclosure, search.
- `/categories/[slug]` — curated category listing.
- `/products/[slug]` — editorial product summary, key facts, source/merchant, price freshness timestamp if shown, pros/cons, alternatives, CTA to merchant.
- `/compare/[slug]` — structured comparisons with dated criteria and clear disclosure.
- `/guides/[slug]` — shopping guides and buying advice.
- `/search` — product/guide search.
- `/about`, `/disclosure`, `/privacy-policy`, `/terms`, `/contact`, `/404`.

Outbound links must pass through a controlled redirect/tracking endpoint or approved analytics mechanism; sanitize destination IDs against an allowlisted merchant/URL record. Record click timestamp, page, product, merchant, and campaign with data minimization. Do not store unnecessary personal data. Use `rel="sponsored"` and appropriate `nofollow` treatment where applicable. Clearly disclose affiliate relationships near recommendations and on a dedicated disclosure page.

Amazon and other networks have different rules for price display, images, link formats, refresh intervals, trademark use, and attribution. Use each network’s approved APIs/assets and terms; otherwise use manually entered factual data without restricted assets. A missing, expired, or broken offer should show a clear unavailable state, not stale availability presented as current.

## 2.4 Blog app navigation and pages

Required routes:
- `/` — editorial home with featured story, topics, recent stories.
- `/category/[slug]`
- `/tag/[slug]`
- `/[year]/[month]/[slug]` or another stable canonical article route.
- `/search`
- `/about`
- `/editorial-policy`
- `/affiliate-disclosure`
- `/privacy-policy`
- `/contact`
- `/rss.xml`, `/sitemap.xml`, `/robots.txt`
- `/404`.

Content pillars may include style/living, practical technology, and smart shopping, but the taxonomy must be editable. Article page requirements: author/byline policy, publication and updated dates when meaningful, table of contents for long articles, related articles, descriptive alt text, sources where factual claims need support, canonical URL, Open Graph metadata, structured data where eligible, and a visible disclosure for affiliate content.

AdSense placements may exist **only** on `blogs.surujenterprise.cc`. Implement ad slots as explicit components and feature flags. Never inject ads into admin, checkout, account, or storefront. Do not reserve large blank ad spaces when ads are disabled. AdSense approval is not guaranteed; publish original, useful content and comply with current program policies before enabling.

## 2.5 Admin command center navigation and pages

The admin app is private, with no public sign-up. Required areas:

1. **Overview**
   - Revenue summaries clearly distinguishing order value, verified payments, refunds, fees, and settled/net figures.
   - Orders awaiting action, low stock, failed jobs/webhooks, pending returns, content awaiting review.
   - Date range and channel filters; each metric links to its underlying records.
   - Data freshness timestamps and explicit empty states.

2. **Catalog**
   - Product list with filters by category, status, channel, stock, supplier, and margin.
   - Create/edit product, variant editor, image manager, attribute/specification editor, SEO preview.
   - Bulk edit with preview and confirmation.
   - Import wizard for CSV/XLSX with column mapping, validation, dry-run preview, row-level errors, duplicate strategy, upsert policy, import history, downloadable error report, and reversible changes where safe.
   - Export with role check and audit log.

3. **Categories & attributes**
   - Nested category tree; slugs, descriptions, navigation visibility, SEO metadata.
   - Attribute definitions by category (size, color, storage, compatibility, material, etc.) without hardcoding one product niche.
   - Variant option values and normalization.

4. **Procurement & suppliers**
   - Supplier contacts, supplier SKUs, purchase orders, incoming stock, cost currency, exchange rate, freight, duties/customs, other landed costs, supplier documents.
   - Receiving workflow and discrepancy records.
   - Supplier data is private and never exposed to public clients.

5. **Inventory**
   - On-hand, reserved, available, incoming, damaged, returned, quarantined quantities.
   - Stock movement ledger with reason, actor, source record, timestamp, and before/after values.
   - Reorder thresholds and low-stock alerts.
   - Adjustments require a reason; bulk adjustments require a preview and confirmation.
   - Inventory must be updated transactionally and never oversell due to concurrent checkout.

6. **Orders & fulfilment**
   - Search by order number, status, date, payment state, carrier, and customer.
   - Order detail with immutable item/price/tax snapshots, payment attempts, event timeline, address, shipment, notes, invoice, return status.
   - Pick/pack queue, packing slips, dispatch, tracking number, partial shipments if supported, delivery exceptions, lost/damaged/delayed/RTO handling.
   - Never dispatch an unpaid/unverified order.
   - Customer-facing notes separated from internal notes.

7. **Payments & finance**
   - Paytm transaction state, provider reference IDs, idempotent callback/webhook event history, reconciliation exceptions, refunds, gateway fee entries.
   - Daily/periodic reconciliation export.
   - Payment status must be server-verified; refund status must distinguish requested, submitted, pending, succeeded, and failed.
   - Refund actions require explicit confirmation and a permission check.

8. **GST & documents**
   - GST profile, invoice series by financial year, invoice/credit note templates, HSN/SAC as applicable, tax configuration versions, supplier purchase documents, marketplace settlement files.
   - Export data for accountant review. Do not represent the app as a substitute for a qualified tax professional.
   - Never hardcode one GST rate for every product.

9. **Sales channels**
   - Channel listing eligibility/status, product mapping, channel-specific title/attributes/price/stock overrides, export templates, sync logs.
   - Marketplace API integration remains disabled until seller account, permissions, API approval, category compliance, and test access exist.
   - Failed sync must not silently overwrite canonical catalog or stock.

10. **Affiliate management**
    - Networks, merchants, offers, destination URLs, disclosure templates, link health, price timestamps, click reports, expiry, product collections, import/export.
    - No assumption of network approval or API access.

11. **Editorial studio**
    - Drafts, review, schedule, publish, unpublish, revisions, categories/tags, author, SEO fields, social preview, image rights/source, alt text, internal links, affiliate disclosures, ad-slot settings.
    - Preview must be visually close to published rendering.
    - Publishing is auditable and only authorized users can do it.

12. **AI workspace**
    - Product description/title/SEO drafts, attribute extraction, catalog cleanup suggestions, image alt suggestions, blog outlines and drafts, internal link suggestions, support-response drafts, and read-only business summaries.
    - Display provider/model, usage, remaining quota if known, errors, and source records.
    - AI output is always a draft until approved. Require confirmation for publish, price/stock changes, refunds, bulk email, tax settings, and credentials.
    - AI must use allowlisted tools with validated arguments; no arbitrary SQL, shell, secrets access, or unrestricted write actions.

13. **Marketing & analytics**
    - Funnel, channel attribution, campaign tags, product performance, repeat purchases, refunds/returns, contribution margin, content performance, outbound affiliate clicks.
    - Consent/privacy configuration and data retention controls.
    - Do not fabricate attribution when signals are unavailable.

14. **Communications**
    - Transactional email templates and event logs, support inbox/ticket status, notification preferences, delivery failures.
    - Preview and test-send actions; bulk marketing requires lawful basis/consent, unsubscribe management, and rate limits.

15. **Settings & security**
    - Business profile, addresses, GST details, invoice sequence, tax settings, shipping settings, payment configuration status (never reveal secrets), domain/brand links, email sender status, feature flags, admin session management, MFA status, audit log, backup/export status, retention/deletion workflows.
    - Owner account bootstrap is a documented, one-time process. No public admin signup or default shared password.

## 2.6 Shared design-system components

Implement and document reusable components:
- App shell, public header/footer, admin sidebar/top bar, breadcrumbs.
- Button variants, icon button, link button, form fields, selects, combobox, checkbox, radio, switch, date range, currency input.
- Product card, product gallery, variant selector, price display, stock badge, filter drawer, sort menu, pagination.
- Empty state, error panel, skeleton, toast, inline validation, confirmation dialog, modal/drawer, tooltip.
- Data table with sort/filter/pagination/column visibility/export where appropriate.
- Status timeline, order status chip, payment state chip, audit log row.
- Rich text/editor blocks with safe HTML handling.
- Responsive chart wrappers with textual equivalents.
- File uploader with MIME/type/size validation and progress.
- AI draft panel showing source/context, generated draft, regenerate, edit, accept/reject, and usage disclosure.

---

# 3. Business rules and canonical workflows

## 3.1 Product and variant model

Products are category-agnostic. A product can have one or more variants. Each sellable variant has a unique SKU, price configuration, inventory record, and optional barcode. Product-level fields include:
- Name, slug, short/long description, brand (optional), category, tags, status.
- Media records with order, alt text, source/rights, and primary-image flag.
- Attribute/specification values.
- SEO title/description, canonical metadata, structured-data eligibility.
- Channel publication flags and channel-specific overrides.
- Cost history, tax classification/HSN, shipping dimensions/weight.
- Created/updated timestamps and actor IDs.

Never infer product specifications, warranty, compatibility, origin, certifications, or safety claims from a vague image or AI guess. Mark unknown values as unknown and require review.

## 3.2 Pricing and margin

Store amounts as integer minor units (paise) and currency code; do not use floating-point for money. Preserve a price/tax snapshot on each order item. Keep selling price, discount, shipping, tax, gateway fee, marketplace fee, landed cost, and refund amounts separate. Show contribution margin estimates as estimates when costs or fees are incomplete.

Use a single canonical pricing service for storefront, checkout, invoice calculations, and admin previews. The browser may display a quote, but the server recalculates price, promotion eligibility, tax configuration, shipping, and stock at checkout.

## 3.3 Cart and checkout

1. Customer builds cart; client cart is a convenience, not an authority.
2. Checkout submits variant IDs and quantities only; server reloads canonical product, current price, tax configuration, and stock.
3. Validate product publication/availability, quantity limits, shipping destination, address, policy version, and duplicate submission.
4. Create a pending order and reserve inventory transactionally with a unique idempotency key and reservation expiry.
5. Create Paytm transaction/order using server-held credentials.
6. Customer completes Paytm flow.
7. Browser return page shows “verifying” until the server confirms status.
8. Paytm callback/webhook/server status verification updates payment/order idempotently.
9. Confirm order only when verified success; release reservation after verified failure/expiry according to the documented reconciliation policy.
10. Send transactional confirmation only after the verified state transition.

Webhook processing must tolerate duplicate, delayed, and out-of-order events. Use an event ledger and deterministic state transitions. A late success after reservation expiry must enter a reconciliation path; it must not silently create an unfulfillable order.

## 3.4 Order state machine

Recommended order states:
- `pending_payment`
- `payment_verification_pending`
- `confirmed`
- `processing`
- `partially_shipped`
- `shipped`
- `delivered`
- `cancellation_requested`
- `cancelled`
- `return_requested`
- `return_in_transit`
- `returned`
- `closed`
- `exception_review`

Payment state is separate:
- `not_started`, `initiated`, `pending`, `succeeded`, `failed`, `cancelled`, `refund_pending`, `partially_refunded`, `refunded`, `reconciliation_required`.

Do not collapse order, payment, shipment, and refund state into a single status. State changes must be validated server-side and written to an event timeline.

## 3.5 Inventory consistency

- Track `on_hand`, `reserved`, `damaged`, `quarantined`, `incoming`; derive available stock from the defined ledger model.
- Reserve stock in a database transaction with row locking or a safe atomic conditional update.
- Enforce non-negative available inventory.
- Every stock change creates a movement record; no silent direct edits.
- Reservation expiry is processed by an idempotent scheduled job.
- Returns do not automatically restore sellable stock; received items go to inspection/quarantine until explicitly restocked.
- Import operations must report changes and avoid accidental stock overwrite unless the operator explicitly chooses a stock update mode.

## 3.6 Shipping and returns

- Start with manual carrier selection/tracking if no shipping aggregator account/API exists.
- Store package dimensions/weight, shipment ID, carrier, tracking code/URL, shipped time, delivered time, label reference, and exception notes.
- Shipping price/ETA is shown only when based on configured and supportable rules; otherwise disclose that final shipping details are confirmed at checkout.
- Return requests capture reason, requested items/quantities, evidence if required, eligibility decision, inspection outcome, refund amount, and event history.
- Customer-facing return policy must match actual operating practice and legal obligations.
- Never label a refund “completed” until the gateway/provider confirms it.

## 3.7 GST and invoices

- Business GST profile, customer billing/shipping data, place of supply, HSN/SAC as applicable, item taxable value, applicable rate, CGST/SGST/IGST split as determined by approved rules, discounts, shipping charges and tax treatment, invoice number/date, financial year, and credit-note references.
- Tax configuration must be versioned with effective dates and an audit trail.
- Invoice numbers must be unique within the configured series and financial-year policy.
- Store immutable invoice snapshots and generated document references.
- Export orders, refunds, gateway fees, purchases, inventory costs, and channel settlements for accountant reconciliation.
- Have an Indian tax professional review the rules, invoice format, tax treatment, and import obligations before production. The software must not guess tax rates from category names.

## 3.8 Email and notifications

Required event types:
- Order received (only after server creates valid order; state clearly if payment is pending).
- Payment verified/failed/pending instructions.
- Order confirmed.
- Shipment dispatched with tracking.
- Delivery update when reliable.
- Cancellation/return status.
- Refund state changes.
- Support request received.
- Admin security/login alerts, payment/webhook errors, low-stock alerts, backup/usage warnings.

Sender setup:
- `noreply@surujenterprise.cc`: transactional sender, monitored bounce/complaint logs.
- `support@surujenterprise.cc`: real mailbox or forwarding/ticket route that is tested.
- Configure and verify SPF, DKIM, and DMARC in DNS. Verify actual sending and receiving before launch.
- Addresses do not exist merely because they are written in the UI.
- Templates must be responsive, accessible, plain-text compatible, and avoid leaking private data.

---

# 4. Shared data architecture

Use PostgreSQL as the source of truth. All table names and relationships below are a logical baseline; migrations may refine them, but any change must preserve the required invariants and be documented.

## 4.1 Core tables

### Identity and access
- `profiles`: user ID, display name, email metadata, created/updated timestamps.
- `admin_allowlist`: authorized user ID, status, granted/revoked timestamps, actor.
- `customer_addresses`: owner ID, address fields, default flag, timestamps.
- `consents`: user/customer, purpose, policy/version, choice, timestamp, source.
- `sessions/security_events`: only where supported and necessary; do not duplicate auth-provider secrets.

### Catalog
- `products`: ID, slug, name, descriptions, brand, status, category ID, SEO fields, created/updated actor.
- `product_variants`: ID, product ID, SKU, barcode, option values, active status, weight/dimensions.
- `categories`: ID, parent ID, slug, name, description, status, sort order, SEO.
- `attribute_definitions`, `product_attribute_values`.
- `product_media`: product/variant ID, storage key/URL, alt text, sort order, source, rights status, primary flag.
- `channel_product_settings`: product/variant, channel, publish status, channel-specific fields.
- `price_records`: variant, currency, amount minor units, effective dates, source, actor.
- `tax_classifications`: variant/product, HSN/SAC, classification status, reviewer, effective dates.

### Procurement and inventory
- `suppliers`, `supplier_products`, `purchase_orders`, `purchase_order_lines`, `goods_receipts`, `goods_receipt_lines`.
- `inventory_balances`: variant/location, on-hand/reserved/damaged/quarantined/incoming values or equivalent derived balance.
- `inventory_movements`: variant, delta, movement type, reason, source entity, actor, timestamp.
- `stock_reservations`: order, variant, quantity, state, expiry, idempotency key.
- `landed_cost_entries`: purchase line, currency, exchange rate, freight, duties, handling, allocation method, evidence reference.

### Commerce and payments
- `carts` and `cart_items` if server-side carts are required.
- `orders`: order number, customer/contact snapshot, address snapshot, order/payment/shipping state, currency, monetary totals, tax snapshot, timestamps.
- `order_items`: immutable SKU/name/variant/price/discount/tax/cost snapshots, quantity.
- `order_events`: event type, prior/new state, actor/source, metadata allowlist, timestamp.
- `payment_attempts`: order, provider, provider order/transaction IDs, amount, status, verification timestamp.
- `payment_webhook_events`: provider event/reference, payload hash or minimized safe payload, processing status, idempotency key, received/processed timestamps.
- `refunds`: order/payment, amount, reason, provider ID, state, requested/processed timestamps.
- `invoices`, `credit_notes`, `financial_entries`, `reconciliation_runs`, `reconciliation_items`.

### Fulfilment and customer care
- `shipments`, `shipment_items`, `shipment_events`.
- `return_requests`, `return_items`, `return_events`, `inspection_records`.
- `support_tickets`, `support_messages`, `internal_notes` with separate visibility fields.

### Affiliate and editorial
- `affiliate_networks`, `affiliate_merchants`, `affiliate_offers`, `affiliate_links`, `affiliate_click_events`, `affiliate_collections`.
- `articles`, `article_revisions`, `article_categories`, `article_tags`, `article_tag_links`, `editorial_assets`, `publication_events`.
- `ad_slot_settings` scoped to blog host only.

### Marketing, automation, platform
- `campaigns`, `event_analytics` (data-minimized), `email_templates`, `notification_jobs`, `job_runs`, `audit_logs`, `feature_flags`, `integration_settings` (no plaintext secrets), `ai_generation_jobs`, `ai_usage_events`, `data_export_jobs`.

## 4.2 Data and database requirements

- UUID primary keys for internal entities; human-readable order number is separate and non-secret.
- Unique constraints for slug within appropriate scope, SKU, invoice number/series, provider transaction IDs, idempotency keys, and other natural identifiers.
- Foreign keys and check constraints for valid amounts, quantities, state values, and relationships.
- Money stored as integer minor units with currency; quantities use suitable decimal precision only where fractional inventory is a real business need.
- All operational tables include appropriate timestamps; records that need accountability include actor/source.
- Avoid destructive cascades for financial, payment, order, invoice, and audit records.
- Use migrations checked into `se-platform`; never edit production schema manually without a reviewed migration and incident record.
- Add indexes for order number, created date, status, SKU, slug, foreign keys, provider references, and common admin filters.
- Paginate all unbounded list APIs.
- Store product images and generated documents in private/public storage according to access needs. Use signed URLs for private invoices/evidence.
- Define retention and deletion rules. Keep only data needed for legal, tax, security, and customer-service purposes; obtain professional guidance on statutory retention.

## 4.3 RLS and access control

- Enable Row Level Security on exposed tables.
- Anonymous users may read only explicitly public, published catalog/content/affiliate data.
- Customers may read/update only their own profile/address and read only their own orders, shipments, invoices, and return requests.
- Only server-side trusted functions may perform privileged payment verification, invoice numbering, stock reservation, refund initiation, admin changes, and webhook processing.
- Admin access requires a valid authenticated user whose ID is in the server-checked allowlist and who satisfies MFA/session policy.
- Never ship Supabase service-role keys or payment/email/AI secrets to the browser.
- Do not rely on hidden navigation/buttons as authorization.
- Test RLS using anonymous, customer A, customer B, authorized admin, and unauthorized authenticated user fixtures.
- Log privileged actions with actor, action, target, timestamp, outcome, and safe metadata; never log passwords, tokens, full payment payload secrets, or unnecessary PII.

---

# 5. API and integration contracts

Use typed request/response schemas and a versioned API convention. REST endpoints below are logical examples; implementation may use server actions or RPCs where justified, but contracts and invariants must remain explicit.

## 5.1 Public catalog
- `GET /api/v1/products` — published products, filters, sort, cursor/page.
- `GET /api/v1/products/{slug}` — product and active variants.
- `GET /api/v1/categories`
- `GET /api/v1/search?q=...` — rate-limited, bounded query length, safe output.
- `GET /api/v1/content/articles` and `GET /api/v1/content/articles/{slug}` — published blog content if served through shared backend.

Public responses must never contain supplier cost, private notes, inventory ledger, customer data, unpublished content, or secret integration settings.

## 5.2 Checkout and orders
- `POST /api/v1/checkout/quote` — validates cart and returns server-calculated totals/expiry; no order created.
- `POST /api/v1/checkout/orders` — creates pending order and stock reservation idempotently.
- `POST /api/v1/payments/paytm/initiate` — initiates transaction server-side.
- `GET /api/v1/orders/{orderNumber}/status` — authenticated or secure-token-gated status.
- `POST /api/v1/orders/{orderNumber}/returns` — create return request if eligible.
- `POST /api/v1/support/tickets` — validated support ticket.

## 5.3 Paytm integration
- Use official Paytm documentation and the current merchant-approved SDK/API version.
- Keep merchant credentials and checksum/signature secrets server-side.
- Verify signatures/checksum and confirm status with Paytm’s server-side status mechanism as required by the selected integration.
- Validate merchant/order reference, amount, currency, expected status, and replay/idempotency conditions.
- A browser return URL is not proof of payment.
- Webhook/callback handlers must be public only as required by Paytm, signature-verified, rate-limited where compatible, idempotent, and observable.
- Store minimal safe event metadata and a payload hash; avoid logging secrets or full sensitive payloads.
- Add a reconciliation job for transactions stuck pending and callback failures.
- Refund initiation is server-side and permission-protected; verify final refund status.
- Production activation requires Paytm merchant approval, sandbox tests, live callback URL registration, and low-value controlled production verification.

## 5.4 Email integration
- Server-side send function accepts a known template ID and validated variables, not arbitrary HTML from the browser.
- Idempotency key prevents duplicate transactional messages.
- Retry transient failures with bounded backoff; dead-letter permanently failing jobs and alert admin.
- Suppress invalid/bounced addresses according to provider rules.
- Marketing email requires separate consent/unsubscribe logic and must not be conflated with transactional notices.

## 5.5 AI provider contract
Create a provider-neutral interface, for example:
- `generateProductDraft(input, constraints)`
- `generateArticleOutline(input, constraints)`
- `generateArticleDraft(approvedBrief, sources, constraints)`
- `suggestImageAlt(imageMetadata, productFacts)`
- `draftSupportReply(ticketContext, policyContext)`
- `summarizeMetrics(aggregatedReadOnlyMetrics)`

Requirements:
- Schema-validated structured output.
- Provider/model and usage recorded.
- Quota, timeout, retry, rate limits, and cost controls.
- Redact personal data and secrets; do not send full customer records unless strictly necessary and approved.
- Prompt injection defenses: retrieved content is data, never authority to bypass system policy.
- No arbitrary SQL execution or direct database credentials.
- Use a read-only analytics tool for summaries.
- Human approval for publishing, product price/stock changes, refunds, tax configuration, bulk email, and external marketplace actions.
- If provider unavailable or quota exceeded, show a useful manual/template fallback.

---

# 6. Security, privacy, reliability, and accessibility

## 6.1 Security baseline
- Enforce HTTPS and secure headers appropriate to the framework; apply a restrictive Content Security Policy where feasible.
- Secure, HttpOnly, SameSite cookies for sessions where applicable.
- MFA for the owner/admin account; no shared account.
- Rate limit login, search, support forms, checkout creation, coupon validation, AI endpoints, and public redirect endpoints.
- Validate all inputs server-side; encode outputs; sanitize rich text; prevent SQL injection, XSS, CSRF, SSRF, open redirects, and mass assignment.
- Use least-privilege API keys, rotate credentials, and separate development/staging/production secrets.
- Do not commit `.env` files or credentials. Add `.env.example` with names only.
- Scan dependencies and secrets in CI; pin versions and review security advisories.
- Upload validation: allowlisted MIME/type, file-size limits, safe filenames, image decoding checks, access control, and malware scanning where available.
- Audit admin actions, exports, tax changes, inventory adjustments, content publication, refunds, and security settings.
- Never place secrets in client-side environment variables or public logs.

## 6.2 Privacy
- Publish accurate privacy notice and cookie/analytics choices as applicable.
- Collect only required customer data; define purposes, access, retention, deletion, and breach response.
- Keep analytics data minimized/pseudonymous where possible.
- Do not send customer PII to AI providers by default.
- Use private access for invoices, returns evidence, supplier documents, and support attachments.
- Ensure customer A cannot access customer B’s data through IDs, filters, exports, or storage paths.
- Have qualified counsel review privacy, consumer, e-commerce, and data-protection obligations before launch.

## 6.3 Reliability and performance
- Set reasonable request timeouts, bounded retries, and idempotency keys for all retryable writes.
- Use durable job records for email, imports, reconciliation, reservation expiry, and AI tasks.
- Monitor failed jobs, payment state mismatches, email failures, inventory anomalies, database/storage quota, build minutes, and hosting credits.
- Avoid unbounded queries; paginate lists and index filter columns.
- Optimize images, lazy-load below-fold media, reserve image dimensions, and minimize client JavaScript.
- Target good Core Web Vitals on representative mobile and desktop devices; record baselines rather than claiming unmeasured scores.
- Graceful degradation: public catalog/blog remain readable during noncritical AI outages; checkout clearly blocks or recovers when payment/stock verification is unavailable.
- Backups must be paired with a tested restore procedure. A backup that has never been restored is not considered verified.

## 6.4 Accessibility
Target WCAG 2.2 AA where practical:
- Semantic HTML, logical headings, keyboard-only operation, visible focus.
- Form labels, instructions, error associations, accessible dialog behavior.
- Alt text for meaningful images; decorative images ignored by assistive technology.
- Reduced motion, sufficient contrast, zoom/reflow support.
- Accessible tables and chart summaries.
- Screen-reader announcements for cart updates, async validation, and payment status changes.

---

# 7. SEO, analytics, and content quality

## 7.1 Storefront SEO
- Unique title/meta description for canonical category and product pages.
- Canonical URLs, robots rules, XML sitemap, redirects for changed slugs.
- Product structured data only when fields are accurate and policy-compliant.
- Accurate price/availability matching visible page content; never mark unavailable products as in stock.
- Open Graph/social metadata and optimized images.
- Faceted/filter URLs need an explicit crawl/index strategy to avoid duplicate-content bloat.
- Search Console and relevant free merchant listings can be added after account verification and policy review.

## 7.2 Blog quality and ads
- Original useful articles, clear editorial standards, fact-checking, human review, accurate authoring and source attribution.
- AI-assisted content must be reviewed for accuracy, originality, unsupported claims, and repetition.
- Do not bulk-publish low-value AI content or use fabricated authors/experience.
- AdSense code and ad-slot configuration are restricted to the blog app. Add a test ensuring no ad script appears on storefront, affiliate, or admin domains.
- Ads must not obscure navigation/content or create accidental-click UI.

## 7.3 Analytics event vocabulary
Use a documented, versioned event vocabulary, for example:
- `view_item_list`, `view_item`, `search`, `add_to_cart`, `remove_from_cart`, `begin_checkout`, `payment_initiated`, `payment_verified`, `purchase`, `shipment_dispatched`, `delivery_confirmed`, `refund_completed`, `affiliate_outbound_click`, `article_view`, `newsletter_consent`.
- Do not fire `purchase` until payment/order criteria are server-confirmed.
- Avoid duplicate purchase events by using order ID as an idempotency key.
- Keep operational analytics separate from financial truth. Finance reports must come from order/payment/refund/reconciliation records.

---

# 8. Local development documentation

## 8.1 Prerequisites
Document and pin supported versions for:
- Git.
- Node.js LTS and the selected package manager.
- Supabase CLI / Docker if local Supabase is used.
- Paytm sandbox merchant account for payment tests.
- Email provider sandbox/test mode.
- Playwright browser dependencies for E2E tests.

The repository must state exact tested versions in `docs/development/prerequisites.md`. Do not assume a developer has global tools installed.

## 8.2 Repository conventions
Each frontend repository should include:
- `README.md`
- `.env.example`
- lockfile and pinned runtime metadata
- lint/typecheck/test/build scripts
- unit and component tests
- Playwright E2E setup where relevant
- `docs/` or a link to `se-docs`
- `CONTRIBUTING.md`
- `SECURITY.md`
- license/ownership information as decided by the owner
- CI workflow for pull requests and protected main branch.

`se-platform` should include:
- database migrations and seed data
- RLS policies
- generated/shared API types
- edge/server functions
- webhook handlers
- scheduled jobs
- database and integration tests
- local development and deployment instructions.

## 8.3 Environment variables
Provide `.env.example` with placeholder values only. Exact names can be standardized during scaffolding; suggested groups:
- Public: `NEXT_PUBLIC_SITE_URL`, `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
- Server-only: `SUPABASE_SERVICE_ROLE_KEY`, `PAYTM_MID`, `PAYTM_MERCHANT_KEY`, `PAYTM_WEBSITE`, `PAYTM_CALLBACK_URL`, `PAYTM_ENV`, `RESEND_API_KEY`, `AI_PROVIDER`, `AI_API_KEY`, `CRON_SECRET`.
- Analytics and integrations: only when enabled, with consent/config documented.

Do not assume every app needs every variable. Keep server-only values out of public prefixes. Never paste production secrets into issues, prompts, logs, or documentation.

## 8.4 Local startup sequence
Each repo README must give copy-pasteable commands matching the chosen package manager. Baseline workflow:
1. Clone the repository and install the pinned runtime/package manager.
2. Copy `.env.example` to `.env.local` and fill only local/test credentials.
3. Start local database/services, or point to a dedicated non-production Supabase project.
4. Apply migrations from a clean database.
5. Seed deterministic demo data: categories, sample products, variants, stock, sample orders in non-paid states, demo article, and affiliate offer.
6. Run lint, typecheck, unit tests, RLS tests, and build.
7. Start the app and test each route.
8. Run Playwright E2E tests against local services.
9. Confirm that test mode cannot call live Paytm, send real bulk email, publish to production, or alter production data.

## 8.5 Local test accounts and seed data
- Never commit real customer or supplier data.
- Use clearly fictional names, addresses, phone numbers, and emails reserved for testing.
- Seed one authorized test admin through a documented secure script, not a hardcoded default password.
- Seed customer A and customer B to verify RLS isolation.
- Include products with variants, zero stock, low stock, tax classification needing review, discontinued status, and missing images.
- Include orders in pending, paid/confirmed, shipped, return requested, partially refunded, and reconciliation-required states.
- Payment sandbox fixtures must cover success, failure, pending, duplicate callback, invalid signature, amount mismatch, late callback, timeout, and refund failure.

## 8.6 Definition of local-ready
A developer can follow the README on a clean machine and:
- install dependencies;
- configure non-production environment variables;
- migrate and seed database;
- run each app;
- sign in as test admin;
- browse products and blog;
- simulate checkout and Paytm sandbox status;
- run tests and produce a production build;
- understand how to reset data without affecting production.

---

# 9. Testing and quality gates

## 9.1 Required test layers
1. Unit tests: pricing, tax-rule selection, discounts, stock availability, state transitions, slugging, import parsing, affiliate redirect validation.
2. Component tests: product variants, cart, checkout form, admin forms, modal accessibility, filters, editor states.
3. Database tests: constraints, transaction behavior, idempotency, stock reservation race, invoice uniqueness, audit events.
4. RLS/security tests: anonymous/customer/admin role matrix; cross-customer access denied.
5. Integration tests: Paytm sandbox, email provider test mode, storage upload/download permissions, AI schema validation.
6. E2E tests: key storefront and admin journeys in desktop and mobile viewports.
7. Accessibility tests: automated checks plus keyboard/screen-reader spot checks.
8. Performance tests: representative catalog size, search, product page, admin list filters, checkout endpoints.
9. Recovery tests: restore backup to isolated environment and validate record counts/invariants.
10. Manual UAT: owner validates content, tax/invoice layout, product facts, policies, checkout wording, and admin workflows.

## 9.2 Minimum release-blocking scenarios
- Concurrent checkouts for the last unit cannot oversell.
- Price/stock changed after cart creation is recalculated before order creation.
- Duplicate checkout requests do not create duplicate orders or reservations.
- Invalid Paytm signature, wrong amount, wrong order ID, or replay cannot mark an order paid.
- Browser redirect alone cannot mark paid.
- Duplicate and out-of-order callbacks are handled safely.
- Paid orders are not accidentally cancelled by late failure callbacks.
- Reservation expiry and late payment success lead to a safe reconciliation state.
- Customer A cannot access Customer B’s order, invoice, address, return, or attachment.
- Public API cannot reveal supplier costs, private inventory, draft articles, admin data, or secrets.
- Refund cannot exceed refundable amount or be initiated twice accidentally.
- Invoice number collision is impossible under concurrent order processing.
- CSV import dry-run detects malformed rows, duplicate SKUs, missing required columns, and unintended stock overwrite.
- AI output cannot publish or change sensitive fields without owner approval.
- Affiliate redirect rejects arbitrary untrusted URLs.
- Blog AdSense code is absent from storefront, affiliate, and admin builds.
- Email failure does not roll back a valid order; it creates a retryable job and alert.
- Restore process works and documented backups can be accessed by the owner.
- Keyboard-only checkout/admin navigation works; no critical accessibility blocker remains.

## 9.3 CI release gates
Pull requests must pass:
- formatting/lint
- TypeScript typecheck
- unit/component tests
- relevant database/RLS tests
- build
- secret/dependency scan
- accessibility smoke tests
- E2E smoke suite for affected app
- migration validation.

Production deploys require approved pull request, passing CI, reviewed migrations, configured secrets, and a rollback/recovery note for risky changes.

---

# 10. Production deployment and operations

## 10.1 Environments
- **Local:** developer machine; fake data and sandbox integrations.
- **Preview:** pull-request deployment; isolated test backend or safe fixtures; never production secrets.
- **Staging:** production-like config, sandbox Paytm, test email recipients, representative synthetic data.
- **Production:** live keys, real customer data, restricted deploy access, reviewed migrations, monitoring.

Use distinct credentials and databases per environment. Never let preview deploys write to production data.

## 10.2 Domain and DNS plan
- `surujenterprise.cc` → storefront deployment.
- `affiliates.surujenterprise.cc` → affiliate deployment.
- `blogs.surujenterprise.cc` → blog deployment.
- `admin.surujenterprise.cc` → admin deployment.
- API hostname only if needed by the chosen architecture.

Set DNS records exactly as instructed by the selected host; do not guess A/CNAME values. Verify domain ownership, HTTPS certificate issuance, redirects, canonical host, and TLS renewal. Ensure cookie domain/session behavior is intentionally configured; do not share admin cookies with public apps unless necessary and securely designed.

## 10.3 Production secrets and integrations
Before production:
- Verify Supabase project and RLS policies.
- Configure storage buckets and policies.
- Verify email sender domain, SPF/DKIM/DMARC, inbound support route.
- Configure Paytm production merchant credentials and registered callback/webhook URLs.
- Configure AI provider only after commercial/privacy/quota review; set usage limits and a kill switch.
- Configure analytics and consent policy.
- Configure blog AdSense only after eligibility/approval.
- Configure cron/job secrets and schedules.
- Verify no test key or test endpoint is present in production builds.

## 10.4 Database release procedure
1. Back up/export current production data and verify access.
2. Review migration SQL and lock/long-running-operation impact.
3. Apply backward-compatible additive migration first.
4. Deploy application compatible with old and new schema.
5. Backfill in bounded batches and verify invariants.
6. Switch reads/writes only after validation.
7. Remove deprecated fields in a later release.
8. Record migration ID, operator, time, results, and rollback/forward-fix plan.

Never rely on a code rollback to undo a destructive database migration. Prefer forward fixes and tested restore for data incidents.

## 10.5 Monitoring and operational alerts
Alert the owner on:
- payment callback verification failures and pending payments beyond threshold;
- order/payment mismatch, duplicate transaction reference, or amount mismatch;
- negative/oversold inventory invariant;
- email bounce/complaint and repeated delivery failures;
- scheduled job failure or growing queue;
- backup/restore failure;
- storage/database/build/hosting quota approaching limits;
- unexpected admin login/security events;
- failed marketplace/affiliate imports and stale offers;
- excessive AI usage or provider outage.

Logs should be structured and searchable, but never contain secrets or full payment authentication payloads. Include request/correlation IDs and safe order references.

## 10.6 Rollback and incident response
For a frontend release: revert to the last known-good deploy if compatible with the database schema. For a payment incident: disable new checkout if verification is unreliable, preserve callback/event evidence, reconcile transactions with Paytm, and contact affected customers only with verified information. For credential exposure: revoke/rotate the credential, review access logs, assess impact, and document the incident. For data corruption: stop affected writes, preserve evidence, restore to an isolated environment first, validate, and execute a reviewed recovery plan.

## 10.7 Backup and restore
- Define database backup/export frequency according to plan capabilities and acceptable data loss.
- Export critical operational data periodically where appropriate.
- Keep backup access independent from the production app credentials.
- Document RPO/RTO targets selected by the owner.
- Perform an initial restore drill before launch and recurring restore drills after major changes.
- Test restoration of schema, data, storage references, and critical documents; verify counts and invariants.

---

# 11. Production readiness checklist

Do not launch real customer checkout until all applicable items are checked.

## Business and legal
- [ ] Business identity, GST details, contact details, policies, pricing, tax handling and invoice format reviewed.
- [ ] Product facts, images, warranty/returns claims and category compliance verified.
- [ ] Shipping regions, carrier processes, handling times and returns operations are real and staffed.
- [ ] Privacy notice, terms, consumer disclosures and retention rules reviewed by a qualified professional as needed.
- [ ] Affiliate disclosures and network-specific rules reviewed.
- [ ] AdSense enabled only after approval and only on blog domain.

## Technical and security
- [ ] All apps build from clean checkout and deploy independently.
- [ ] RLS and server-side authorization tests pass.
- [ ] Admin MFA enabled; no public admin registration.
- [ ] Secrets absent from repository, client bundle, logs and screenshots.
- [ ] Rate limiting, validation, safe redirects and security headers configured.
- [ ] Database backup and restore drill completed.
- [ ] Monitoring, alerts, error logs and quota checks configured.
- [ ] Accessibility and performance smoke tests pass.

## Payments and fulfilment
- [ ] Paytm sandbox success/failure/pending/duplicate/invalid signature/refund tests pass.
- [ ] Production credentials and callback URLs verified by owner.
- [ ] Server-side transaction status verification works.
- [ ] No order is marked paid from redirect alone.
- [ ] Reconciliation process documented and tested.
- [ ] Stock reservation, expiry, late success, refund and cancellation paths tested.
- [ ] Real order confirmation email and support inbox tested.
- [ ] One controlled low-value production transaction/refund is reconciled, if allowed and approved.

## Content and discovery
- [ ] Sitemap, robots, canonical links, metadata and 404s checked.
- [ ] Structured data matches visible, accurate content.
- [ ] Search Console and analytics configured appropriately.
- [ ] Affiliate links and disclosures tested; broken/stale offer behavior works.
- [ ] Blog editorial review and publication workflow tested.
- [ ] No placeholder copy, broken images, fake metrics, or test content is public.

---

# 12. Initial implementation plan

The complete feature set is specified now. Implement in dependency order so later apps do not invent separate rules.

### Stage A — Engineering foundation
- Create repos and branch protections.
- Establish design tokens, shared contracts, lint/type/test/CI conventions.
- Set up local and staging environments.
- Create database migrations, RLS baseline, seed data, audit logging, and error handling.
- Document architecture decisions and environment variables.

### Stage B — Catalog, procurement, inventory
- Categories, attributes, products, variants, media rights, prices, tax classification.
- Import/export wizard and audit trail.
- Suppliers, purchase orders, receiving, landed costs.
- Stock ledger, reservations, low-stock reporting.

### Stage C — Commerce, payments, fulfilment
- Public storefront and search.
- Cart/checkout, server pricing, pending order, stock reservation.
- Paytm sandbox and verified status lifecycle.
- Customer orders, invoices, pick/pack, shipments, returns/refunds, emails.
- GST/accounting exports and reconciliation.

### Stage D — Affiliate and editorial
- Affiliate offers, merchant rules, disclosures, click tracking, stale-link controls.
- Blog CMS, revision/review/publish, SEO, sitemap/RSS.
- Ad slots feature-gated and restricted to blog host.

### Stage E — Admin and AI workspace
- Admin dashboard and operational modules.
- AI provider adapter, quotas, safe tools, review/approval workflow, manual fallback.
- Analytics, reports, alerting, settings, data export.

### Stage F — Hardening and launch
- Security/RLS review, payment race tests, accessibility, performance, restore drill.
- Legal/accounting review, DNS, email sender, Paytm production configuration.
- Controlled launch, daily reconciliation, monitor quota/operational alerts.

Each stage must deliver tested, deployable increments. Do not leave the whole system untestable until the end.

---

# 13. Acceptance criteria by application

## Storefront
- Customers can browse published products, search/filter/sort, view variants, manage cart, check out with online Paytm payment, and see verified order status.
- Out-of-stock/unpublished products cannot be purchased.
- Price, tax, discount, and stock are recalculated server-side.
- Guest and authenticated order flows work without exposing another customer’s data.
- Customer can access policies, support, invoices, tracking, and eligible return request.

## Affiliate
- Editor can create/update offers and collections.
- Public user sees clear affiliate disclosure and accurate product facts.
- Outbound click tracking works without an open redirect.
- Expired/broken offers are handled clearly.
- Affiliate pages never create Suruj-owned orders or claim Suruj fulfils third-party orders.

## Blog
- Authorized editor can draft, revise, preview, schedule, publish, and unpublish.
- Public routes, categories, search, RSS, sitemap, metadata, and canonical links work.
- Editorial and affiliate disclosures are visible where needed.
- AdSense integration is isolated to the blog host.
- Public pages work when AI and ad services are disabled.

## Admin
- Only allowlisted owner can access protected pages and APIs.
- Admin can manage catalog, inventory, orders, fulfilment, finance exports, affiliate offers, blog, emails, AI drafts, settings and audit logs.
- Destructive actions have explicit confirmation and audit records.
- Every dashboard metric links to source records and distinguishes estimates from reconciled figures.

## Platform
- Migrations apply from empty database.
- RLS tests prove isolation.
- Payment/order/inventory operations are idempotent and race-safe.
- Jobs retry safely and surface terminal errors.
- Logs, alerts, backups, and restore procedures are documented and tested.
- Each app can deploy independently against the shared versioned contracts.

---

# 14. Documentation deliverables required in Git

Create these documents before production:

## `se-docs`
- `README.md` — documentation index and ownership.
- `architecture/system-overview.md` — apps, data flow, trust boundaries.
- `architecture/data-model.md` — ERD, table dictionary, data lifecycle.
- `architecture/api-contracts.md` — endpoints, auth, errors, idempotency.
- `architecture/decisions/` — ADRs for stack, hosting, auth, storage, search, AI, analytics.
- `development/prerequisites.md`
- `development/local-setup.md`
- `development/test-data-and-reset.md`
- `development/debugging.md`
- `testing/test-plan.md`
- `testing/payment-test-matrix.md`
- `testing/rls-security-matrix.md`
- `deployment/environments.md`
- `deployment/domains-dns.md`
- `deployment/production-release.md`
- `deployment/database-migrations.md`
- `operations/payments-and-reconciliation.md`
- `operations/orders-and-fulfilment.md`
- `operations/inventory-and-imports.md`
- `operations/gst-and-invoices.md`
- `operations/email-and-support.md`
- `operations/affiliate-compliance.md`
- `operations/blog-and-adsense.md`
- `operations/ai-safety-and-quotas.md`
- `operations/backups-and-restore.md`
- `operations/incident-response.md`
- `security/secrets-and-rotation.md`
- `security/access-control.md`
- `security/privacy-and-retention.md`
- `runbooks/payment-outage.md`
- `runbooks/email-failure.md`
- `runbooks/oversell-or-stock-mismatch.md`
- `runbooks/data-restore.md`
- `runbooks/credential-exposure.md`
- `runbooks/hosting-quota-exceeded.md`
- `business/policies/` — owner-reviewed customer policies.
- `business/gst-review-notes.md` — accountant-approved tax rules and assumptions.

## Per-app README minimum
Each app README must include purpose, route map, prerequisites, exact install/run/test/build commands, environment variables, deployment settings, integration mocks, troubleshooting, and links to the shared docs.

---

# 15. Open configuration items (must be decided before relevant production feature)

These are not reasons to delay scaffolding, but they must not be silently guessed:
1. Final hosting plan and current free-tier limits.
2. Supabase project region, backup capabilities, and staging/prod project layout.
3. Paytm merchant account, approved integration version, production callback URLs and settlement process.
4. Email provider, inbound support mailbox/forwarding, sender-domain verification.
5. Shipping carriers, shipping zones, rate/label process, dispatch cut-off and service-level promises.
6. Accountant-approved GST/HSN rules, invoice series, credit-note policy, financial-year rules and import treatment.
7. Return/cancellation/warranty policy and actual customer support coverage.
8. AI provider, free-tier quota, commercial terms, privacy and model availability.
9. Affiliate networks and permission to use their feeds/assets/prices.
10. Marketplace seller accounts and API eligibility, if marketplace sync is pursued.
11. Analytics provider and consent requirements.
12. Backup RPO/RTO, alert channel, incident contact and production release approver.
13. Actual product data, images, supplier details, landed costs and opening stock.
14. Domain DNS access and verification for all subdomains.

Record each decision in an ADR or settings record, with date, owner, reason, and consequences.

---

# 16. Final engineering rules

1. The database and server-side business rules are authoritative; the browser is not.
2. Payment, order, inventory, refund, invoice, and audit records must be consistent and traceable.
3. No COD.
4. No live payment without server verification and reconciliation.
5. No admin access based on UI hiding alone.
6. No secret in Git, browser bundles, public storage, logs, or AI prompts.
7. No public data query may reveal supplier costs, customer data, drafts, or operational secrets.
8. No arbitrary AI SQL or autonomous high-impact writes.
9. No AdSense outside the blog app/domain.
10. No fake reviews, urgency, stock claims, prices, or product facts.
11. No production release without tests, reviewed migrations, monitored integrations, and a recovery path.
12. No claiming a third-party integration works until credentials, permissions, and end-to-end tests prove it.
13. Every import, refund, stock adjustment, publication, tax change, and privileged configuration change is auditable.
14. Every key workflow has loading, empty, error, success, retry, and permission-denied states.
15. Free-tier assumptions must be rechecked periodically; architecture must allow upgrades without a full rewrite.

**Definition of done:** the owner can manage real products, inventory, online-only Paytm checkout, fulfilment, GST-reviewed invoicing and reconciliation, affiliate offers, original blog content, AdSense on the blog only when approved, and all operational settings through the admin console; customers and public visitors see only authorized data; every critical financial and stock transition is verified, tested, and auditable; all apps can be locally run, tested, independently deployed, monitored, backed up, and recovered using the committed documentation.
