# Requirement Decisions

**Documentation revision:** 2026-10-04 — confirmed decision set and BRD synchronization reviewed.

This document is the decision log for `docs/BRD.md`.

Confirmed items are **product decisions**. They override earlier recommended defaults and any conflicting wording in the BRD.

Unresolved items remain open. Recommended defaults for unresolved items may be used only as working assumptions until a decision owner confirms them.

---

## Decision log

| Decision ID | Status | Topic | Date recorded |
|-------------|--------|-------|----------------|
| DEC-001 | Open | Commercial pricing model | — |
| DEC-002 | Open | Organization scale targets | — |
| DEC-003 | Confirmed | Warehouse permission model | 2026-10-04 |
| DEC-004 | Confirmed | Product variants | 2026-10-04 |
| DEC-005 | Confirmed | Unit conversion | 2026-10-04 |
| DEC-006 | Confirmed | Inventory valuation | 2026-10-04 |
| DEC-007 | Confirmed | Organization currency | 2026-10-04 |
| DEC-008 | Confirmed | Tax handling | 2026-10-04 |
| DEC-009 | Confirmed | Purchase order approval | 2026-10-04 |
| DEC-010 | Confirmed | AI action approval | 2026-10-04 |
| DEC-011 | Open | Data retention | — |
| DEC-012 | Open | SUPER_ADMIN access to tenant operational data | — |
| DEC-013 | Open | Multi-organization users | — |
| DEC-014 | Confirmed | Damaged and expired stock / inventory terminology | 2026-10-04 |
| DEC-015 | Open | Partial sales fulfillment | — |
| DEC-016 | Open | Invoicing depth | — |
| DEC-017 | Open | Payment recording depth | — |
| DEC-018 | Confirmed | Batch and expiry tracking | 2026-10-04 |
| DEC-019 | Confirmed | Sales approval | 2026-10-04 |
| DEC-020 | Confirmed | Stock transfer approval | 2026-10-04 |
| DEC-021 | Confirmed | Negative stock | 2026-10-04 |
| DEC-022 | Confirmed | Backorders | 2026-10-04 |
| DEC-023 | Confirmed | Cost visibility | 2026-10-04 |
| DEC-024 | Confirmed | AI data freshness | 2026-10-04 |
| DEC-025 | Confirmed | Localization | 2026-10-04 |
| DEC-026 | Confirmed | Opening stock | 2026-10-04 |
| DEC-027 | Confirmed | Supplier and customer uniqueness | 2026-10-04 |
| DEC-028 | Confirmed | Reserved stock sources | 2026-10-04 |
| DEC-029 | Confirmed | Purchase returns | 2026-10-04 |
| DEC-030 | Confirmed | Customer returns | 2026-10-04 |
| DEC-031 | Confirmed | Reorder point ownership | 2026-10-04 |
| DEC-032 | Confirmed | Organization self-registration | 2026-10-04 |
| DEC-033 | Confirmed | Inactive products | 2026-10-04 |
| DEC-034 | Confirmed | Concurrent competing sales | 2026-10-04 |
| DEC-035 | Confirmed | AI recommendation acceptance | 2026-10-04 |

---

## Confirmed decisions

| Decision ID | Topic | Question | Confirmed decision | Why it matters | Alternatives not chosen | Impact |
|-------------|-------|----------|--------------------|----------------|-------------------------|--------|
| DEC-003 | Warehouse permission model | Should operational access be restricted by assigned warehouse? | **Warehouse-level access control is required.** SUPER_ADMIN: all warehouses across the organization. ADMIN: all warehouses in the organization. MANAGER: all warehouses in the organization. INVENTORY_STAFF, SALES_STAFF, and VIEWER: assigned warehouses only. Restrictions are enforced by the backend, not only hidden in the frontend. Operational users must not access inventory, transactions, reports, or other warehouse-scoped information for warehouses they are not assigned. | Prevents cross-warehouse leakage and accidental operations at the wrong location. | Organization-wide access for all roles; optional warehouse restriction; a separate warehouse-manager role. | User assignment, operational workflows, reporting, AI answers, and audit. |
| DEC-004 | Product variants | Are variants first-class catalog entities? | **No variant matrix in the initial version.** Each inventory item is a unique SKU/product record (for example T-Shirt Blue Medium, T-Shirt Blue Large, and T-Shirt Red Medium are separate products if they need separate stock). A parent-product/variant system is future scope. | Inventory identity, purchasing, and sales stay SKU-based. | Parent with child SKUs; attributes without inventory split. | Catalog, inventory, purchasing, sales, and AI recommendations. |
| DEC-005 | Unit conversion | How complex is unit conversion? | **Simple conversion only**, using explicit configured factors (example: 1 Box = 12 Pieces). Conversions are deterministic. No complex universal unit-of-measure engine. | Conversion errors affect inventory accuracy and purchasing. | Stocking unit only with no conversion; packing hierarchies; multi-step conversion tables. | Receiving, sales quantities, reporting, and AI quantity suggestions. |
| DEC-006 | Inventory valuation | Which valuation method is used? | **Weighted Average Cost.** The system maintains sufficient cost information to calculate weighted-average inventory cost correctly. No FIFO, LIFO, or other complex accounting valuation methods in the initial version. | Valuation drives inventory value reports and management decisions. | FIFO; LIFO; standard cost; last cost; organization-selectable methods. | Receiving, adjustments, returns, reports, and AI explanations of value. |
| DEC-007 | Organization currency | One currency or many? | **One base currency per organization.** Currency is organization-level configuration. No multi-currency accounting or multi-currency inventory valuation in the initial version. | Prices, documents, payments, and reports must be interpretable. | Multi-currency documents; per-warehouse currency. | Catalog pricing, purchasing, sales, payments, reporting, and AI numeric statements. |
| DEC-008 | Tax handling | How is tax treated? | **Basic optional tax fields only** (rate/amount where required for purchasing and sales). No full tax engine and no complex jurisdictional tax calculations. | Avoids turning the product into a tax/accounting platform. | No tax fields; jurisdiction-specific tax engines; statutory compliance. | Document totals and the accounting-scope boundary. |
| DEC-009 | Purchase order approval | Who must approve POs, and are there thresholds? | **Submitted purchase orders require approval** before they proceed to the next operational stage. **ADMIN and MANAGER** can approve. No approval matrices and no configurable monetary thresholds in the initial version. | Controls spending and receiving against unauthorized buys. | Amount thresholds; supplier-based rules; no approval for small orders. | Purchasing workflow, notifications, audit, and AI-assisted PO actions. |
| DEC-010 | AI action approval | Which AI actions need a human? | **Any AI-generated action that can materially change business data or operational state requires explicit human approval.** Examples: creating or modifying purchase orders; changing reorder settings; creating inventory adjustments; confirming transfers; other material write operations. AI may analyze and recommend automatically, but must not silently execute high-impact writes. Approved actions still go through normal application authorization and business-rule validation. | This is the boundary between assistance and unauthorized change. | Amount-based auto-execute; silent low-risk writes. | AI design, audit, operational risk, and user trust. |
| DEC-014 | Damaged and expired stock | How do non-sellable quantities relate to available stock? | **Separate sellable and non-sellable concepts.** Physical Stock includes sellable and non-sellable inventory. **Available Stock = Sellable Stock − Reserved Stock.** Damaged and expired stock are not available for normal sales. Inventory movements must identify transitions involving damaged or expired stock. Reserved stock is a commitment against sellable stock; it is not an extra quantity added on top of physical stock. | Using “available” to mean all physical stock would allow selling damaged or expired goods. | Treat damaged/expired as immediate write-off only; keep a single on-hand number with no buckets. | Inventory meaning, valuation, warehouse processes, reports, and AI answers. |
| DEC-018 | Batch and expiry tracking | Are lots and expiry required? | **Batch/lot tracking and expiry dates are in the initial version.** Serial-number tracking is not. Batch and expiry information must be available where relevant to receiving, stock tracking, sales, returns, reporting, and AI analysis. | Expiry and lot traceability need identity below SKU without serialization. | No lots; mandatory serialization. | Receiving, sales, returns, expiry, transfers, reports, and AI stock answers. |
| DEC-019 | Sales approval | Do sales require approval before they reserve stock? | **Normal sales do not require manual approval.** Cancellations, exceptional overrides, and other high-impact operational exceptions require MANAGER or ADMIN authorization. No complex configurable sales approval workflow in the initial version. | Keeps fulfillment fast while controlling exceptions. | All sales require approval; value-threshold approval. | Sales workflow, reservations, and audit. |
| DEC-020 | Stock transfer approval | Do transfers require approval? | **Warehouse transfers require approval by an authorized MANAGER or ADMIN before final confirmation.** Transfers must respect warehouse permissions and inventory availability rules. | Transfers move value between locations and can hide stock. | No approval; quantity/value thresholds; warehouse-manager-only approval. | Transfer workflow, notifications, AI transfer drafts, and audit. |
| DEC-021 | Negative stock | May stock go negative? | **Negative stock is not allowed.** The system must prevent confirmation of transactions that would cause available or sellable stock to become negative. Concurrent competing consumption of the same available inventory must not succeed. | Negative stock destroys fulfillment reliability and trust. | Allow negative stock with a warning; allow oversell. | Sales, transfers, adjustments, concurrency, and AI actions. |
| DEC-022 | Backorders | May a sale be confirmed without available stock? | **Backorders are not supported in the initial version.** If sufficient available stock does not exist, the sale cannot be confirmed. Backorders may be future scope. | Backorders would change reservation, fulfillment, and customer promises. | Accept backorders; reserve what is available and backorder the rest. | Sales, notifications, and availability meaning. |
| DEC-023 | Cost visibility | Which roles may see inventory cost? | **ADMIN and MANAGER can view inventory cost information. INVENTORY_STAFF, SALES_STAFF, and VIEWER cannot view inventory cost information by default.** Backend authorization enforces this. AI tools apply the same rules. | Cost is commercially sensitive. | All roles see cost; cost only for ADMIN; user-configurable grants in the initial version. | Reporting, AI answers, and permission design. |
| DEC-024 | AI data freshness | How must AI treat non-current data? | **AI responses and recommendations must communicate data freshness where relevant.** AI must not present stale, cached, delayed, or historical data as live/real-time. When analysis depends on non-current information, the relevant timestamp or freshness must be identified. | Stale advice can cause bad purchasing or fulfillment decisions. | Always imply live data; omit timestamps. | Trust, notifications, and AI answers. |
| DEC-025 | Localization | Which languages at launch? | **Initial release language is English.** Single localization language and organization base currency. Additional languages / internationalization are future scope. | Language affects adoption and AI responses. | Multi-language at launch; language per user. | UI, notifications, and AI responses. |
| DEC-026 | Opening stock | How is opening stock established? | **Authorized users may enter or import opening stock.** Opening-stock operations must validate the product and warehouse, validate quantities, create inventory movements, preserve audit history, and respect organization and warehouse permissions. Opening stock must not bypass inventory integrity rules. | Opening stock is the foundation of inventory accuracy. | Opening stock only via purchase receipts; no opening stock. | Onboarding, inventory ledger, valuation, and audit. |
| DEC-027 | Supplier and customer uniqueness | How unique are partner codes? | **Supplier codes and customer codes are unique within an organization.** They are not globally unique across organizations. Cross-organization identifiers must never create data leakage or conflicts. | Duplicates damage history, performance, and AI recommendations; global uniqueness would collide tenants. | Unique names only; global uniqueness; no uniqueness. | Master data, reports, isolation, and AI supplier advice. |
| DEC-028 | Reserved stock sources | Which documents reserve stock? | **Confirmed sales create reservations where applicable. Draft sales do not. Approved stock transfers may reserve inventory where required by the transfer workflow. Purchase orders do not reserve sellable inventory.** Physical, reserved, and available stock must remain clearly distinct. | Reservation changes Available Stock. | Reserve at draft; never reserve transfers; reserve on invoice only. | Availability, sales, transfers, and AI stock answers. |
| DEC-029 | Purchase returns | When does a supplier return change stock? | **Stock impact occurs when the purchase return is confirmed/dispatched according to the workflow.** Draft purchase returns do not change inventory. All stock changes create traceable inventory movements. | Timing affects physical and sellable stock during return disputes. | Decrease only after supplier acknowledgement. | Inventory, purchasing, and audit. |
| DEC-030 | Customer returns | When do customer returns become sellable? | **Customer returns do not automatically become sellable stock.** Returned goods go through inspection. After inspection: acceptable goods may return to sellable inventory; damaged goods remain non-sellable; expired goods remain non-sellable where applicable. All resulting inventory changes are recorded and auditable. | Restocking unsellable goods creates false availability. | Always restock immediately; never restock. | Inventory, returns, valuation, and available stock. |
| DEC-031 | Reorder point ownership | Who maintains reorder points? | **Authorized users maintain reorder points.** AI may recommend changes using demand, lead time, stock levels, and other available data. AI must not silently overwrite user-maintained reorder points. AI-generated changes are recommendations or drafts and follow the approval workflow. | Accountability for planning parameters stays with the business. | AI-managed min/max only; no user thresholds. | Catalog, notifications, purchasing, and AI recommendations. |
| DEC-032 | Organization self-registration | Who can create an organization? | **Qualified users may create an organization through initial onboarding and become its initial ADMIN.** SUPER_ADMIN platform administration may also create organizations. No complex organization approval workflow in the initial version. | Affects onboarding speed and platform duties. | SUPER_ADMIN-only creation; invite-only; approval workflow. | Onboarding workflow and security. |
| DEC-033 | Inactive products | Can inactive products be used on new documents? | **Inactive products cannot be added to new purchasing or sales transactions.** They remain visible in historical transactions, inventory history, reports, and audit records. Historical data remains intact. | Retires products from operations without destroying history. | Allow new sales of inactive products; hard-hide immediately. | Catalog, operations, reporting, and audit. |
| DEC-034 | Concurrent competing sales | What happens when two users consume the last available units? | Availability must be checked inside a database transaction; relevant inventory records must be locked appropriately; availability must be revalidated before confirmation; only transactions with sufficient available stock may succeed; competing transactions must fail gracefully when stock is no longer available. **Overselling because of a race condition is never allowed.** | This is a core inventory integrity expectation. | Oversell with warning; first-draft wins. | Sales, inventory integrity, and user experience. |
| DEC-035 | AI recommendation acceptance | Does accepting AI advice execute the action? | **AI recommendations do not directly execute high-impact actions.** Workflow: AI analyzes data → produces recommendation → user reviews → user accepts/edits → system creates draft action → normal validation/authorization → required approval → execution → audit log. Recommendations remain distinguishable from confirmed business actions. | Acceptance is an operational control point. | Immediate execute on accept; bookmark-only with no draft. | AI actions, approvals, and audit. |

### Confirmed inventory terminology (DEC-014)

These definitions are authoritative across the BRD.

| Term | Definition |
|------|------------|
| Physical Stock | The total physical quantity recorded at a warehouse/product level, including sellable and non-sellable inventory. |
| Sellable Stock | Physical inventory that is currently fit for normal sale (not damaged or expired). |
| Reserved Stock | Sellable inventory committed/reserved for confirmed business transactions according to the reservation rules. Reserved quantity is part of sellable quantity, not an additional physical bucket. |
| Available Stock | Sellable inventory that is not reserved. **Available Stock = Sellable Stock − Reserved Stock.** Available Stock must not mean total physical inventory. |
| Damaged Stock | Physical inventory that is not currently sellable because it is damaged. |
| Expired Stock | Physical inventory that is not currently sellable because it has expired. |

Conceptual model:

```
Physical Stock
├── Sellable Stock
│   ├── Available Stock
│   └── Reserved Stock
├── Damaged Stock
└── Expired Stock
```

Therefore: **Physical Stock = Sellable Stock + Damaged Stock + Expired Stock**, and **Sellable Stock = Available Stock + Reserved Stock**.

---

## Unresolved decisions

These items are **not** confirmed. Do not treat recommended defaults as approved.

| Decision ID | Topic | Question | Why It Matters | Recommended Default | Alternatives | Impact |
|-------------|-------|----------|----------------|---------------------|-------------|--------|
| DEC-001 | Commercial pricing model | How will the product itself be priced and packaged for customers (for example subscription tiers, usage, or per-organization licensing)? | Packaging affects which capabilities are included, expected organization size, and commercial constraints. | Defer commercial packaging. Treat the initial product as a single full-capability offering for SMB inventory operations. | Tiered plans; per-warehouse pricing; per-user pricing; usage-based AI pricing. | Affects sales, onboarding, feature flags, and possibly AI usage limits. |
| DEC-002 | Organization scale targets | What is the expected maximum size of an organization in the initial product (users, warehouses, products, transactions)? | Scale expectations affect process design, reporting usefulness, and later technical capacity planning. | Design for typical SMB scale: tens of users, a small number of warehouses, thousands of products, and ongoing daily purchasing and sales activity. | Micro-business only; large multi-branch enterprise from day one. | Affects usability, reporting, and later performance/scalability work. |
| DEC-011 | Data retention | How long must operational, audit, and AI records be retained? | Retention affects compliance, storage, audit usefulness, and privacy. | Retain operational and audit history for the life of the organization unless a later retention policy is defined. AI conversation content may have a shorter retention policy after confirmation. | Fixed year-based retention; legal-hold exceptions; separate audit retention. | Affects audit, reporting, privacy, and later storage design. |
| DEC-012 | SUPER_ADMIN access to tenant data | May a platform SUPER_ADMIN view or change an organization's operational inventory, sales, and purchase data beyond warehouse-access rules in DEC-003? | This is a trust, privacy, and multi-tenant isolation question. DEC-003 confirms warehouse access when operating in an organization context; broader operational duty and support impersonation remain open. | SUPER_ADMIN may manage platform users, organization lifecycle, and system configuration. Access to tenant operational records is not a normal duty and, if ever allowed for support, must be exceptional, authorized, and audited. | Full operational access; support impersonation; no tenant data access at all. | Affects security, support processes, audit, and customer trust. |
| DEC-013 | Multi-organization users | May one person belong to more than one organization? | Membership rules affect login, permissions, and isolation. | A business user belongs to one organization. Platform SUPER_ADMIN is not an organization operating user. | Multi-organization membership with explicit org switching; consultant access across tenants. | Affects user management, authentication experience, and isolation rules. |
| DEC-015 | Partial sales fulfillment | Can a sales order be shipped or completed in parts? | Partial fulfillment changes reservation, deduction, and order state. | Allow partial fulfillment: remaining reserved quantity stays committed until completed or cancelled. | Complete-only fulfillment; backorders as separate orders. | Affects sales lifecycle, inventory reservation, reporting, and notifications. |
| DEC-016 | Invoicing depth | Are invoices operational sales documents, or a step toward accounts receivable accounting? | Invoice scope can expand the product into accounting. | Sales invoices are commercial records of a sale for operational history and payment recording. They are not a general ledger, accounts receivable subledger, or tax filing system. | No invoices; full AR/AP accounting; statutory invoicing. | Affects sales close, payments, reports, and out-of-scope accounting boundary. |
| DEC-017 | Payment recording depth | How completely should payments against sales and purchases be recorded? | Payments can expand into banking and accounting. | Allow recording of payment status and amounts against sales and purchase documents (for example unpaid, partially paid, paid). No bank feeds, cash management, or reconciliation. | Status only; no payments; full payment allocation and banking. | Affects customer/supplier history and keeps accounting out of scope. |
