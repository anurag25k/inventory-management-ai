# Functional Requirements Document

**Product:** Inventory Management System with AI Inventory Intelligence  
**Document type:** Functional Requirements Document (FRD)  
**Phase:** Functional Requirements  
**Status:** Draft for review  
**Source documents:** `docs/BRD.md`, `docs/REQUIREMENT-DECISIONS.md`  
**Documentation revision:** 2026-10-04

---

## 1. Purpose

This document translates the approved business requirements into observable system behavior.

The FRD defines:

- actors and permissions;
- functional capabilities;
- inputs and validation;
- business rules;
- workflows and state transitions;
- success and failure behavior;
- audit expectations;
- notification behavior;
- AI behavior and safety boundaries;
- functional acceptance criteria;
- traceability back to BRD requirements.

This document does **not** prescribe a programming language, framework, database schema, API technology, hosting provider, or UI implementation.

Those decisions belong to later architecture and technical-design phases.

---

## 2. Source and Decision Authority

### 2.1 Requirement authority

The BRD defines what the business needs and why.

The confirmed decisions document defines product decisions that override earlier recommended defaults or conflicting wording.

### 2.2 Confirmed decisions

The following decisions are treated as confirmed for this FRD:

- DEC-003 — Warehouse permission model
- DEC-004 — Product variants
- DEC-005 — Unit conversion
- DEC-006 — Inventory valuation
- DEC-007 — Organization currency
- DEC-008 — Tax handling
- DEC-009 — Purchase order approval
- DEC-010 — AI action approval
- DEC-014 — Damaged and expired stock / inventory terminology
- DEC-018 — Batch and expiry tracking
- DEC-019 — Sales approval
- DEC-020 — Stock transfer approval
- DEC-021 — Negative stock
- DEC-022 — Backorders
- DEC-023 — Cost visibility
- DEC-024 — AI data freshness
- DEC-025 — Localization
- DEC-026 — Opening stock
- DEC-027 — Supplier and customer uniqueness
- DEC-028 — Reserved stock sources
- DEC-029 — Purchase returns
- DEC-030 — Customer returns
- DEC-031 — Reorder point ownership
- DEC-032 — Organization self-registration
- DEC-033 — Inactive products
- DEC-034 — Concurrent competing sales
- DEC-035 — AI recommendation acceptance

### 2.3 Unresolved decisions

The following remain open and must not be silently decided by implementation:

- DEC-001 — Commercial pricing model
- DEC-002 — Organization scale targets
- DEC-011 — Data retention
- DEC-012 — SUPER_ADMIN access to tenant operational data
- DEC-013 — Multi-organization users
- DEC-015 — Partial sales fulfillment
- DEC-016 — Invoicing depth
- DEC-017 — Payment recording depth

Where one of these decisions affects behavior, this FRD records the dependency and does not invent a final policy.

---

## 3. Actors

| Actor | Functional role |
|---|---|
| SUPER_ADMIN | Platform administration and organization lifecycle support; not a normal tenant operator. |
| ADMIN | Organization administration and high-impact operational control. |
| MANAGER | Operational/commercial management and approvals. |
| INVENTORY_STAFF | Receiving, warehouse operations, transfers, and authorized stock tasks. |
| SALES_STAFF | Sales and customer-facing inventory operations. |
| VIEWER | Read-only access to permitted information. |
| System | Automated calculations, validations, notifications, alerts, audit events, and other deterministic processes. |
| AI Copilot | Authorized analysis, explanation, prediction, recommendation, and draft-action generation. AI is not an unrestricted operator. |

---

## 4. Global Functional Rules

### FR-GEN-001 — Authentication

The system shall require authentication before a user can access non-public operational capabilities.

### FR-GEN-002 — Organization context

Every business operation shall execute within an authenticated organization membership context.

A client-provided organization identifier shall not be sufficient to obtain another organization's data.

### FR-GEN-003 — Authorization

Every operational action shall be authorized using the authenticated user's role, organization membership, and applicable warehouse restrictions.

### FR-GEN-004 — Backend enforcement

Security and permission rules shall be enforced by the backend/business layer. Frontend visibility is not a security boundary.

### FR-GEN-005 — Named users

Operational actions shall be attributable to named users. Deactivated users shall not be able to perform new operational actions.

### FR-GEN-006 — Warehouse scope

For warehouse-scoped roles, every inventory, transaction, report, notification, and AI result involving a warehouse shall be filtered to warehouses assigned to the user.

### FR-GEN-007 — Organization isolation

Catalog, inventory, partners, documents, reports, notifications, AI context, files, and audit information shall remain isolated by organization.

### FR-GEN-008 — Sensitive information

Inventory cost, credentials, and personal contact information shall be treated as sensitive information and exposed only to authorized users.

### FR-GEN-009 — Auditability

Important business actions shall create auditable history containing the acting identity, organization, action, affected entity, time, and relevant context.

### FR-GEN-010 — No silent stock edits

No stock quantity shall be changed by directly overwriting the quantity without a recognized business event and traceable history.

### FR-GEN-011 — Transactional business operations

Receiving, sale completion, transfer, return, and other multi-step stock operations shall not leave the business in a partially updated state.

### FR-GEN-012 — Deterministic calculations

Numeric operational figures shall be calculated from recorded business data whenever possible. AI may explain calculated results but shall not replace deterministic arithmetic.

### FR-GEN-013 — AI does not block core operations

The system shall continue to support core purchasing, receiving, sales, and stock-adjustment recording when AI services are unavailable.

---

# 5. Organization and Tenant Management

## 5.1 Organization creation

### FR-ORG-001

A qualified user shall be able to create an organization during onboarding.

### Inputs

- Organization identity
- Contact/business information as required
- Base currency
- Locale/timezone or equivalent organization settings

### Rules

1. The creator becomes the initial ADMIN.
2. Organization data is isolated from all other organizations.
3. SUPER_ADMIN may also create organizations.
4. No complex organization approval workflow is required in the initial product.
5. Base currency is one currency per organization.
6. Multi-currency inventory valuation is not supported initially.

### Success

A new active organization exists with its initial ADMIN and organization settings.

### Failure

The system shall reject incomplete or unauthorized organization creation.

---

## 5.2 Organization settings

### FR-ORG-002

ADMIN shall be able to maintain organization settings within permitted scope.

Settings include:

- organization profile;
- base currency;
- locale/timezone;
- operational defaults.

### FR-ORG-003

Organization settings shall not permit a user to alter another organization's configuration.

---

# 6. User Management

## 6.1 User creation

### FR-USR-001

ADMIN shall be able to create or invite named users for the organization.

### Inputs

- Identity/contact information
- Role
- Optional warehouse assignments where applicable
- Active/inactive status

### Rules

1. Users require organization membership.
2. Users cannot grant themselves higher privileges.
3. A tenant user cannot assign SUPER_ADMIN from within the tenant.
4. Warehouse assignments apply to INVENTORY_STAFF, SALES_STAFF, and VIEWER.
5. ADMIN and MANAGER have access to all organization warehouses.
6. The initial membership model is one business organization per user unless DEC-013 is later resolved differently.

### FR-USR-002

ADMIN shall be able to deactivate users.

A deactivated user:

- cannot perform new operational actions;
- remains attributable in historical records;
- does not have their audit history erased.

### FR-USR-003

The system shall prevent unsafe user-management actions, including attempts to leave an organization without a valid administrator where that would violate the organization's administration requirements.

---

# 7. Roles and Permission Behavior

## 7.1 SUPER_ADMIN

### FR-ROLE-001

SUPER_ADMIN shall be treated as a platform role.

It may:

- manage platform organization lifecycle;
- support platform-level configuration;
- access all warehouses when explicitly operating within an organization context, subject to the unresolved DEC-012 boundary.

It shall not automatically become a normal tenant operator.

### FR-ROLE-002

Any exceptional tenant-data access permitted under a future resolution of DEC-012 shall be separately controlled and audited.

---

## 7.2 ADMIN

### FR-ROLE-003

ADMIN shall be able to:

- manage organization settings;
- manage users and roles;
- manage master data;
- oversee warehouses;
- approve purchase orders;
- approve warehouse transfers;
- review AI action approvals;
- view inventory costs;
- review audit history;
- perform other authorized high-impact operations.

ADMIN is restricted to their organization.

---

## 7.3 MANAGER

### FR-ROLE-004

MANAGER shall be able to:

- oversee inventory;
- approve purchase orders;
- approve warehouse transfers;
- approve high-impact AI actions;
- manage/perform authorized adjustments;
- review reports and KPIs;
- view inventory costs;
- oversee sales and purchasing.

MANAGER is restricted to their organization.

---

## 7.4 INVENTORY_STAFF

### FR-ROLE-005

INVENTORY_STAFF shall be able to perform permitted warehouse operations in assigned warehouses, including:

- receive goods;
- record receiving discrepancies;
- record damage/expiry;
- participate in authorized adjustments;
- draft/execute permitted transfer steps;
- inspect returned goods.

They shall not:

- manage organization users;
- approve purchase orders;
- view inventory costs by default;
- access unassigned warehouses.

---

## 7.5 SALES_STAFF

### FR-ROLE-006

SALES_STAFF shall be able to:

- maintain customer records required for sales;
- create sales;
- confirm sales where authorized;
- complete sales;
- initiate customer returns;
- view availability in assigned warehouses.

They shall not:

- freely adjust stock to enable a sale;
- approve purchase orders;
- access unassigned warehouses;
- view inventory cost by default.

---

## 7.6 VIEWER

### FR-ROLE-007

VIEWER shall have read-oriented access only within assigned warehouses.

VIEWER shall not:

- create/change products;
- change stock;
- create/change operational documents;
- approve actions;
- view inventory cost by default.

---

# 8. Product Catalog

## 8.1 Product creation

### FR-PROD-001

An authorized user shall be able to create a product.

Required/standard information includes:

- SKU;
- product name;
- stocking unit;
- status;
- optional description;
- optional category;
- optional brand;
- optional business metadata such as barcode/notes.

### Rules

1. SKU is unique within the organization.
2. Each inventory item is represented by a unique SKU/product.
3. There is no variant matrix in the initial release.
4. Products requiring separate stock tracking are separate SKUs.
5. Product belongs to exactly one organization.

---

## 8.2 Product lifecycle

### FR-PROD-002

Product status shall support at least:

- Draft
- Active
- Inactive
- Archived

### Rules

- Draft products cannot be used on live purchasing or sales documents.
- Active products may be purchased, received, sold, transferred, and reported.
- Inactive products cannot be added to new purchase or sales documents.
- Inactive products remain visible in history, inventory history, reports, and audit records.
- Archived products are removed from ordinary operational search while remaining historically referenced.
- Products with historical transactions shall not be casually deleted.

---

## 8.3 Product images/documents

### FR-PROD-003

Authorized users may attach product images/documents where supported.

Attachments are identification/business records and shall not be treated as executable content.

---

## 8.4 Reorder settings

### FR-PROD-004

Authorized users shall be able to maintain low-stock thresholds and reorder levels.

AI may recommend changes but shall not silently overwrite user-maintained values.

---

# 9. Categories and Brands

## 9.1 Categories

### FR-CAT-001

Authorized users shall be able to create and maintain organization-scoped product categories.

### Rules

- Category names are unique within the applicable organizational hierarchy.
- Simple parent/child hierarchy may be supported.
- Categories can be activated/deactivated.
- Categories in use cannot be casually deleted.
- Deleting an in-use category requires reassignment or another controlled process.

## 9.2 Brands

### FR-BRN-001

Authorized users shall be able to create and maintain organization-scoped brands.

### Rules

- Brand names are unique within the organization.
- Brands may be active/inactive.
- In-use brands cannot be casually deleted.
- Brand is optional for products.

---

# 10. Units of Measurement

### FR-UOM-001

The organization shall be able to define units used for stocking, purchasing, and selling.

### FR-UOM-002

Each product shall have a stocking unit used as the inventory quantity unit.

### FR-UOM-003

The system shall support explicit deterministic conversion factors.

Example:

`1 Box = 12 Pieces`

### Rules

- Conversions shall not rely on AI.
- Inventory quantities are maintained in the product's stocking unit.
- Units in use cannot be casually deleted.
- No complex universal UOM engine is required initially.

---

# 11. Warehouse Management

### FR-WH-001

ADMIN/MANAGER shall be able to create and maintain warehouses within their organization.

Warehouse information includes:

- name;
- code;
- status;
- address/contact information as required.

### FR-WH-002

Warehouse codes shall be unique within the organization.

### FR-WH-003

Inactive warehouses shall not accept ordinary new stock movements except controlled wind-down operations.

### FR-WH-004

Each inventory position shall be associated with a warehouse.

### FR-WH-005

Warehouse permissions shall be enforced as follows:

| Role | Warehouse access |
|---|---|
| SUPER_ADMIN | All organization warehouses when operating in organization context |
| ADMIN | All organization warehouses |
| MANAGER | All organization warehouses |
| INVENTORY_STAFF | Assigned warehouses only |
| SALES_STAFF | Assigned warehouses only |
| VIEWER | Assigned warehouses only |

### FR-WH-006

Warehouse restrictions shall apply to:

- inventory;
- sales;
- purchases/receiving;
- transfers;
- reports;
- notifications;
- AI answers;
- AI drafts.

### FR-WH-007

A default warehouse may be configured for convenience but shall not bypass permissions or stock validation.

### FR-WH-008

Opening stock shall be established per product and warehouse by an authorized entry/import.

---

# 12. Inventory

## 12.1 Stock terminology

The following definitions are authoritative.

### FR-INV-001

**Physical Stock** is the total physical quantity recorded for a product/warehouse, including sellable and non-sellable inventory.

### FR-INV-002

**Sellable Stock** is physical inventory currently fit for normal sale.

### FR-INV-003

**Reserved Stock** is sellable inventory committed to confirmed transactions according to reservation rules.

Reserved stock is part of sellable stock, not an additional physical quantity.

### FR-INV-004

**Available Stock** is sellable stock not reserved.

`Available Stock = Sellable Stock - Reserved Stock`

Available stock is not a manually edited independent quantity.

### FR-INV-005

**Damaged Stock** is physical inventory not currently sellable because it is damaged.

### FR-INV-006

**Expired Stock** is physical inventory not currently sellable because it has expired.

### FR-INV-007

The conceptual relationship is:

```text
Physical Stock
├── Sellable Stock
│   ├── Available Stock
│   └── Reserved Stock
├── Damaged Stock
└── Expired Stock
```

Therefore:

```text
Physical Stock = Sellable Stock + Damaged Stock + Expired Stock
Sellable Stock = Available Stock + Reserved Stock
```

---

## 12.2 Inventory visibility

### FR-INV-008

Authorized users shall be able to view inventory by:

- product;
- warehouse;
- physical stock;
- sellable stock;
- reserved stock;
- available stock;
- damaged stock;
- expired stock;
- recent movements;
- batch/expiry where applicable;
- inbound purchase quantity.

Results shall be filtered by warehouse permission.

---

## 12.3 Negative stock

### FR-INV-009

The system shall never allow a confirmed operation to create negative available or sellable stock.

### FR-INV-010

Stock availability shall be revalidated within the final business transaction before a competing sale/consumption operation is committed.

---

## 12.4 Inventory movement ledger

### FR-INV-011

Every stock-changing operation shall create a traceable inventory movement.

Recognized movement categories include:

- Opening
- Purchase
- Sale
- Purchase Return
- Sales Return
- Adjustment In
- Adjustment Out
- Transfer Out
- Transfer In
- Damage
- Expired

### FR-INV-012

Inventory movements shall record sufficient context to answer:

> Why did the stock quantity change?

### FR-INV-013

Inventory history shall not be editable as ordinary business data.

---

## 12.5 Opening stock

### FR-INV-014

Authorized users shall be able to enter/import opening stock.

Validation shall include:

- valid organization;
- valid warehouse;
- valid product;
- valid quantity;
- appropriate stock classification;
- permission;
- audit information.

Opening stock shall create inventory movements.

---

## 12.6 Adjustments

### FR-INV-015

Authorized users shall be able to create controlled inventory adjustments.

Each adjustment shall contain:

- affected product/warehouse;
- quantity;
- direction/type;
- reason;
- actor;
- audit context.

An adjustment shall not silently overwrite stock.

### FR-INV-016

Adjustments that would create negative stock shall be rejected.

---

## 12.7 Damage and expiry

### FR-INV-017

The system shall support moving stock from sellable to damaged or expired classifications through recognized business events.

Damaged and expired stock shall not be included in available stock.

---

## 12.8 Batch and expiry

### FR-INV-018

The initial product shall support batch/lot tracking and expiry dates where relevant.

Serial-number tracking is out of initial scope.

Batch/expiry information shall be usable in receiving, inventory, sales, returns, reports, and AI analysis.

---

## 12.9 Inventory valuation

### FR-INV-019

Inventory valuation shall use Weighted Average Cost.

### FR-INV-020

Valuation shall be calculated from recorded inventory cost information.

AI shall not invent inventory value.

Cost visibility shall respect DEC-023.

---

# 13. Suppliers

### FR-SUP-001

Authorized users shall be able to create and maintain organization-scoped supplier records.

Supplier data may include:

- supplier code;
- identity;
- status;
- contacts;
- addresses;
- notes;
- supplier-product relationships;
- supplier-specific prices;
- historical performance.

### FR-SUP-002

Supplier codes shall be unique within an organization.

### FR-SUP-003

Inactive suppliers cannot be selected for new purchase orders.

### FR-SUP-004

Supplier history shall include purchases, receipts, returns, and related operational information.

### FR-SUP-005

Supplier performance calculations shall use available history only.

Missing data shall not be fabricated.

---

# 14. Purchasing

## 14.1 Purchase order creation

### FR-PUR-001

Authorized procurement users shall be able to create purchase orders.

A PO shall contain:

- supplier;
- expected warehouse;
- product lines;
- quantities;
- commercial prices;
- optional tax fields;
- status.

### FR-PUR-002

PO quantities shall be expressed in a defined unit and converted to stocking units when an explicit conversion applies.

### FR-PUR-003

Draft POs shall not affect inventory.

---

## 14.2 Purchase order lifecycle

Initial states:

```text
Draft
  ↓
Pending Approval
  ↓
Approved
  ↓
Partially Received
  ↓
Received

or

Cancelled
```

### FR-PUR-004

Submitting a PO shall move it to Pending Approval.

### FR-PUR-005

ADMIN or MANAGER shall approve submitted POs.

### FR-PUR-006

The initial product shall not use configurable approval matrices or monetary approval thresholds.

### FR-PUR-007

Unapproved POs shall not proceed as approved purchasing intake.

### FR-PUR-008

Partial receiving shall be supported.

### FR-PUR-009

A cancelled PO shall not be receivable.

### FR-PUR-010

Commercial totals shall be deterministic calculations from PO lines and optional recorded tax fields.

There is no full tax engine.

---

# 15. Goods Receiving

### FR-REC-001

Goods shall normally be received against an approved/receivable purchase order.

### FR-REC-002

Authorized receiving users shall record received quantity for each line.

### FR-REC-003

Partial receipts shall be supported.

### FR-REC-004

Receiving shall support recording:

- accepted quantity;
- short quantity;
- damaged quantity;
- relevant batch/lot;
- expiry.

### FR-REC-005

Accepted good quantity shall increase physical and sellable stock.

### FR-REC-006

Damaged quantity shall not become sellable or available stock.

### FR-REC-007

Receiving shall create receipt history and inventory movements.

### FR-REC-008

INVENTORY_STAFF may receive only into assigned warehouses.

ADMIN/MANAGER may receive into any organization warehouse.

### FR-REC-009

Over-receiving is an exception and is not assumed to be freely permitted.

---

# 16. Customers

### FR-CUS-001

Authorized users shall be able to create and maintain customer records required for sales and returns.

### FR-CUS-002

Customer codes shall be unique within the organization.

### FR-CUS-003

Inactive customers shall not be selectable for new sales.

### FR-CUS-004

Customer history shall retain sales, returns, and limited payment status where payment recording is enabled.

### FR-CUS-005

This capability shall remain an inventory-commerce customer record, not a full CRM.

---

# 17. Sales

## 17.1 Sales creation

### FR-SAL-001

Authorized sales users shall be able to create sales orders.

A sales order shall include:

- customer;
- fulfillment warehouse;
- product lines;
- quantity;
- selling price;
- status.

### FR-SAL-002

Draft sales shall not reserve stock.

---

## 17.2 Sales lifecycle

Confirmed initial states:

```text
Draft
  ↓
Confirmed
  ↓
Completed

or

Cancelled
```

### FR-SAL-003

Confirmation shall require sufficient available stock.

### FR-SAL-004

Backorders shall not be supported.

### FR-SAL-005

On confirmation:

- reserved stock increases;
- available stock decreases;
- physical stock does not change;
- sellable stock does not change.

### FR-SAL-006

On completion/fulfillment:

- physical stock decreases;
- sellable stock decreases;
- corresponding reservation is released;
- a Sale movement is recorded.

### FR-SAL-007

Normal sales do not require manual approval.

### FR-SAL-008

Cancellations, exceptional overrides, and other configured high-impact exceptions require MANAGER or ADMIN authorization.

### FR-SAL-009

Sales confirmation shall respect assigned warehouse access.

---

## 17.3 Concurrent sales

### FR-SAL-010

The system shall prevent race-condition overselling.

When competing users attempt to consume the same available quantity:

1. Availability is checked.
2. Relevant inventory is protected during the transaction.
3. Availability is revalidated.
4. Only sufficient-stock transactions succeed.
5. Competing transactions fail gracefully when stock is no longer available.

---

## 17.4 Partial fulfillment

### FR-SAL-011

Partial fulfillment remains dependent on unresolved DEC-015.

Until DEC-015 is confirmed, implementation shall not silently choose a final partial-fulfillment policy.

---

# 18. Returns

## 18.1 Customer returns

### FR-RET-001

Authorized users shall be able to create customer returns containing:

- original sale reference where available;
- product;
- quantity;
- reason;
- warehouse;
- inspection/result.

### FR-RET-002

Returned goods shall not automatically become sellable.

Inspection determines whether goods become:

- sellable;
- damaged/non-sellable;
- expired/non-sellable where applicable.

### FR-RET-003

Return quantities shall respect the eligible quantity rule defined by the BRD assumptions.

### FR-RET-004

Customer returns shall create auditable inventory movements.

---

## 18.2 Purchase returns

### FR-RET-005

Authorized users shall be able to create supplier/purchase returns linked to supplier and relevant receipt/purchase history.

### FR-RET-006

Draft purchase returns shall not affect inventory.

### FR-RET-007

Stock impact shall occur when the purchase return is confirmed/dispatched according to the defined workflow.

### FR-RET-008

The system shall prevent uncontrolled return quantities beyond eligible received quantities.

---

# 19. Warehouse Transfers

## 19.1 Transfer creation

### FR-TRF-001

Authorized users shall be able to create a transfer between warehouses.

A transfer shall include:

- source warehouse;
- destination warehouse;
- product lines;
- quantities;
- status;
- actor/history.

### FR-TRF-002

A transfer cannot use the same warehouse as both source and destination.

### FR-TRF-003

Source and destination access shall be checked independently.

---

## 19.2 Transfer lifecycle

Initial lifecycle:

```text
Draft
  ↓
Pending Approval
  ↓
Approved
  ↓
Dispatched
  ↓
Received

or

Cancelled
```

### FR-TRF-004

MANAGER or ADMIN shall approve transfers before final confirmation.

### FR-TRF-005

Dispatch shall decrease source inventory according to the transfer rules.

### FR-TRF-006

Receipt shall increase destination inventory.

### FR-TRF-007

In-transit stock shall remain visible and traceable.

### FR-TRF-008

Insufficient source available stock shall prevent dispatch.

### FR-TRF-009

Partial transfer behavior remains subject to the existing BRD assumption and future confirmation if changed.

---

# 20. Reporting and KPIs

### FR-RPT-001

Authorized users shall be able to view operational reports appropriate to their organization and warehouse permissions.

Reports may cover:

- inventory;
- inventory value;
- movement;
- stock aging;
- dead stock;
- stockouts;
- purchases;
- sales;
- returns;
- supplier performance;
- warehouse distribution;
- approvals;
- AI recommendations.

### FR-RPT-002

Reports shall not mix organizations.

### FR-RPT-003

Reports shall not expose cost information to roles prohibited by DEC-023.

### FR-RPT-004

KPI calculations shall be deterministic over operational data.

AI may explain KPI results but shall not invent them.

### FR-RPT-005

The system shall support the following business KPI meanings:

- Inventory turnover
- Stockout rate
- Overstock rate
- Dead stock value
- Inventory accuracy
- Order fulfillment rate
- Supplier performance
- Purchase cycle time
- Sales performance
- Warehouse Inventory Distribution
- Average inventory value
- Stock aging
- AI recommendation acceptance rate

---

# 21. Notifications

### FR-NOT-001

The system shall support business notifications for relevant events, including:

- low stock;
- stockout risk;
- purchase approvals;
- receipts;
- transfers;
- AI alerts;
- important operational events.

### FR-NOT-002

Notifications shall respect organization and warehouse permissions.

### FR-NOT-003

Low-stock detection shall use the configured business stock measure and thresholds.

The BRD establishes available stock as the basis for low-stock interpretation.

### FR-NOT-004

Notification behavior shall avoid unnecessary alert volume.

Notification frequency/channel details beyond the initial business requirement remain subject to implementation design and existing assumptions.

---

# 22. Audit and Activity Tracking

### FR-AUD-001

The system shall record important business actions.

Audit context shall include, where applicable:

- actor;
- organization;
- action;
- entity;
- timestamp;
- before/after context where appropriate;
- reason;
- approval/rejection;
- AI involvement.

### FR-AUD-002

Inventory movements shall be separately traceable as business history.

### FR-AUD-003

AI-generated recommendations, drafts, approvals, rejections, and executions shall be auditable.

### FR-AUD-004

Business users shall not be able to casually edit or delete audit history.

### FR-AUD-005

Audit visibility shall respect organization and role permissions.

---

# 23. AI Inventory Copilot

## 23.1 Natural-language questions

### FR-AI-001

Authorized users shall be able to ask natural-language inventory questions.

Examples:

- Which products are low in stock?
- Which products may stock out soon?
- How much stock exists across permitted warehouses?
- Which warehouse has the most inventory?
- Which products are not moving?
- What should be reordered this week?
- Which supplier is best for this product?

### FR-AI-002

The copilot shall answer only from information the user is authorized to see.

### FR-AI-003

The copilot shall respect:

- organization isolation;
- role permissions;
- warehouse restrictions;
- cost visibility.

### FR-AI-004

The copilot shall not invent:

- products;
- quantities;
- orders;
- suppliers;
- prices;
- movements;
- financial values.

### FR-AI-005

If required information is unavailable, the copilot shall say so rather than fabricate it.

---

# 24. AI Statement Types

### FR-AI-006

AI outputs shall distinguish:

| Type | Meaning |
|---|---|
| FACT | Recorded business value |
| CALCULATION | Deterministic result from recorded data |
| PREDICTION | Uncertain future estimate |
| RECOMMENDATION | Advice based on available evidence |
| ACTION | Proposed or executed operational change |

### FR-AI-007

Predictions and recommendations shall not be presented as facts.

### FR-AI-008

Numeric operational values such as stock, revenue, inventory value, reorder arithmetic, and tax/invoice totals shall be calculated from business data whenever possible.

---

# 25. AI Demand and Stock Analysis

### FR-AI-010

The system shall support historical demand analysis when sufficient sales/movement history exists.

### FR-AI-011

Historical calculations shall be based on recorded transactions rather than fabricated missing history.

### FR-AI-012

Stockout risk may be presented as a prediction/recommendation using available:

- historical sales;
- current available stock;
- inbound purchase quantities;
- thresholds;
- other relevant authorized data.

### FR-AI-013

The system shall communicate when data history is sparse or insufficient for a robust prediction.

---

# 26. AI Reorder Recommendations

### FR-AI-014

AI may recommend:

- products to reorder;
- suggested quantities;
- suggested timing;
- supporting reasoning.

### FR-AI-015

Recommendations should use available supporting information such as:

- available stock;
- reserved demand;
- reorder level;
- recent sales;
- inbound purchase quantities;
- supplier lead-time history.

### FR-AI-016

Numeric suggested quantities should be grounded in application calculations where possible.

### FR-AI-017

AI shall not silently overwrite reorder levels.

### FR-AI-018

Accepting a reorder recommendation shall not execute a purchase automatically.

The result becomes a draft action and follows normal validation and approval.

---

# 27. AI Supplier Recommendations

### FR-AI-019

AI may compare organization suppliers using available:

- price;
- historical performance;
- lead time;
- reliability;
- product availability/history.

### FR-AI-020

Missing supplier information shall not be invented.

### FR-AI-021

"Best supplier" shall be presented as a recommendation rather than an objective fact.

### FR-AI-022

Supplier recommendations shall remain organization- and permission-scoped.

---

# 28. AI Anomaly Detection

### FR-AI-023

The system may identify unusual patterns including:

- unexpected stock changes;
- unusual sales;
- unusual accumulation;
- possible data-entry errors;
- abnormal product movement.

### FR-AI-024

An anomaly shall be presented as a signal requiring investigation, not proof of fraud or error.

### FR-AI-025

Users shall be able to trace an anomaly to underlying movements/documents where permitted.

### FR-AI-026

Anomaly detection shall not modify stock.

---

# 29. AI Daily Briefing

### FR-AI-027

Authorized managers may receive a briefing containing relevant:

- low-stock products;
- stockout risks;
- unusual activity;
- sales trends;
- purchase activity;
- pending approvals/receipts/transfers.

### FR-AI-028

Briefing items shall be traceable to authorized records.

### FR-AI-029

Briefings shall distinguish facts/calculations from predictions/recommendations.

### FR-AI-030

Briefings shall respect recipient permissions.

---

# 30. AI-Generated Reports and Summaries

### FR-AI-031

AI may summarize existing reports and operational records in natural language.

### FR-AI-032

Summaries shall not introduce transactions or quantities absent from source records.

### FR-AI-033

Users should be able to access the underlying report or record supporting a summary.

---

# 31. AI-Assisted Actions

### FR-AI-034

AI may prepare drafts for explicitly supported operational actions, including:

- purchase orders;
- inventory adjustments;
- stock transfers;
- reorder setting changes;
- other approved draft operations.

### FR-AI-035

AI-generated drafts shall use the same business validation rules as equivalent human-created drafts.

Examples:

- product exists;
- warehouse exists;
- quantity is valid;
- user is authorized;
- stock policy is satisfied.

### FR-AI-036

AI shall never bypass:

- permissions;
- approvals;
- inventory ledger rules;
- organization isolation;
- business validation.

### FR-AI-037

AI-originated drafts shall remain identifiable as AI-originated in operational history.

---

# 32. AI Human Approval

### FR-AI-038

Any AI-generated action that can materially change business data or operational state shall require explicit human approval.

Examples:

- creating/modifying purchase orders;
- changing reorder settings;
- inventory adjustments;
- confirming transfers;
- other material writes.

### FR-AI-039

The required sequence shall be:

```text
AI analyzes authorized data
        ↓
AI produces recommendation
        ↓
User reviews
        ↓
User accepts/edits
        ↓
System creates draft action
        ↓
Normal validation + authorization
        ↓
Required human approval
        ↓
Current state revalidated
        ↓
Normal action execution
        ↓
Audit
```

### FR-AI-040

AI cannot approve its own action.

### FR-AI-041

Approval shall be performed by an authorized human using their own credentials.

### FR-AI-042

Before execution, the system shall revalidate:

- current stock;
- current document state;
- permissions;
- relevant business constraints;
- freshness/staleness.

### FR-AI-043

If validation fails after a recommendation was accepted, execution shall not proceed until the user resolves the current state.

### FR-AI-044

A rejected AI action shall not create an operational stock/order effect.

The rejected proposal may remain in history for audit.

### FR-AI-045

Read-only AI operations may execute automatically when authorized.

Write operations are subject to stricter controls.

---

# 33. AI Security and Prompt/Content Trust

### FR-AI-046

Stored business text such as:

- product descriptions;
- notes;
- supplier names;
- customer-entered text;

shall be treated as untrusted content.

Such content shall not override:

- system instructions;
- permissions;
- organization boundaries;
- approval requirements;
- inventory rules.

### FR-AI-047

The AI shall not operate as an unrestricted database administrator.

### FR-AI-048

The AI shall not perform arbitrary data exploration outside approved business capabilities.

---

# 34. AI Data Freshness and Confidence

### FR-AI-049

Where AI results depend on non-current information, the system shall communicate relevant data freshness.

### FR-AI-050

Stale or cached data shall not be presented as real-time data.

### FR-AI-051

Uncertain or low-confidence outputs shall be labelled appropriately.

### FR-AI-052

Missing, conflicting, or insufficient data shall result in a safe limitation/refusal rather than fabricated certainty.

---

# 35. End-to-End Functional Workflows

## WF-001 — Organization onboarding

1. Qualified user starts onboarding.
2. Organization identity is captured.
3. Organization is created.
4. Initial ADMIN is established.
5. Organization settings are configured.
6. System confirms isolated organization context.

Failure cases:

- incomplete organization data;
- unauthorized creator;
- duplicate/conflicting identity where applicable.

---

## WF-002 — User creation

1. ADMIN creates/invites user.
2. User profile is created.
3. Role is assigned.
4. Warehouse assignments are made when applicable.
5. User is activated.
6. User authenticates.
7. Effective permissions are applied.

---

## WF-003 — Product creation

1. Authorized user creates product.
2. SKU uniqueness is checked.
3. Stocking unit is selected.
4. Optional category/brand/metadata is provided.
5. Product begins as Draft where required.
6. Product becomes Active after required data is valid.
7. Product becomes available for operational use.

---

## WF-004 — Warehouse creation

1. ADMIN/MANAGER creates warehouse.
2. Warehouse code uniqueness is checked.
3. Warehouse is activated.
4. Operational users may be assigned.
5. Inventory can subsequently be recorded there.

---

## WF-005 — Opening inventory

1. Authorized user selects warehouse/product.
2. Quantity/classification is entered.
3. Validation occurs.
4. Opening movement is created.
5. Audit record is created.
6. Inventory becomes visible.

---

## WF-006 — Purchase order creation

1. Authorized user creates Draft PO.
2. Supplier is selected.
3. Warehouse is selected.
4. Product lines and quantities are added.
5. Deterministic totals are calculated.
6. PO is submitted.
7. Status becomes Pending Approval.

No inventory increase occurs.

---

## WF-007 — Purchase approval

1. ADMIN/MANAGER opens submitted PO.
2. Supplier, warehouse, quantities, prices and other relevant information are reviewed.
3. Approver approves/rejects according to supported workflow.
4. Approval/rejection is audited.
5. Approved PO becomes receivable.

AI-created POs follow exactly the same approval path.

---

## WF-008 — Goods receiving

1. Receiver selects an approved PO.
2. Warehouse permission is validated.
3. Received quantities are entered.
4. Short/damaged quantities are classified.
5. Batch/expiry is recorded where relevant.
6. Receipt is confirmed.
7. PO status is updated.
8. Inventory movements are created.

---

## WF-009 — Inventory after receiving

Accepted good quantity:

- increases Physical Stock;
- increases Sellable Stock;
- increases Available Stock;
- does not change Reserved Stock.

Damaged receipt:

- increases the appropriate non-sellable classification;
- does not increase Available Stock.

---

## WF-010 — Sales order creation

1. Authorized sales user selects customer.
2. Authorized warehouse is selected.
3. Product lines are entered.
4. Draft is saved.
5. No stock reservation occurs.

---

## WF-011 — Sales reservation

1. User requests confirmation.
2. Customer/product/warehouse permissions and statuses are checked.
3. Available Stock is calculated.
4. Required inventory is protected for the transaction.
5. Availability is revalidated.
6. If sufficient, order becomes Confirmed.
7. Reserved Stock increases.
8. Available Stock decreases.
9. Audit/history is created.

If insufficient stock exists, confirmation fails.

---

## WF-012 — Sale completion

1. Fulfillment is initiated.
2. Fulfilled quantity is validated against reservation.
3. Physical Stock decreases.
4. Sellable Stock decreases.
5. Reservation is released.
6. Sale movement is created.
7. Sales history is updated.

Partial fulfillment remains subject to DEC-015.

---

## WF-013 — Inventory deduction

Sale completion creates a Sale movement and keeps:

`Available Stock = Sellable Stock - Reserved Stock`

consistent after the reservation release and physical/sellable reduction.

---

## WF-014 — Customer return

1. Return is created.
2. Original transaction/eligibility is checked where applicable.
3. Returned goods are received.
4. Goods are inspected.
5. Good goods may return to Sellable Stock.
6. Damaged goods remain non-sellable.
7. Expired goods remain non-sellable where applicable.
8. Return and inventory history are recorded.

---

## WF-015 — Supplier return

1. Purchase return is created.
2. Eligible received quantity is validated.
3. Draft does not change inventory.
4. Return is confirmed/dispatched.
5. Inventory decreases according to returned classification.
6. Movement and audit history are recorded.

---

## WF-016 — Stock adjustment

1. Authorized user proposes adjustment.
2. Reason is captured.
3. Permission and quantity rules are checked.
4. Approval is obtained where required.
5. Inventory movement is recorded.
6. Audit history is recorded.

AI-generated adjustments follow the AI approval workflow.

---

## WF-017 — Warehouse transfer

1. User creates transfer.
2. Source/destination permissions are checked.
3. Product and quantity are validated.
4. Transfer enters approval workflow.
5. MANAGER/ADMIN approves.
6. Source is dispatched.
7. Source inventory is reduced.
8. In-transit state is visible.
9. Destination receives.
10. Destination inventory increases.
11. Paired transfer history is retained.

---

## WF-018 — Low-stock detection

1. Inventory event/review occurs.
2. System evaluates available stock against threshold.
3. Low-stock condition is identified.
4. Relevant notification is created.
5. Notification is delivered only to authorized recipients.

---

## WF-019 — Reorder recommendation

1. Authorized user requests reorder advice or a briefing triggers it.
2. AI gathers permitted stock, reservation, threshold, sales, inbound, and supplier information.
3. Deterministic quantities/calculations are performed where possible.
4. AI produces a labelled recommendation.
5. Supporting reasons and data freshness are presented.
6. No purchase is executed automatically.

---

## WF-020 — AI recommendation

1. User asks a natural-language question.
2. User identity/organization/warehouse permissions are established.
3. Approved business information is retrieved.
4. Deterministic calculations are performed.
5. AI interprets the results.
6. Response distinguishes statement types.
7. Missing data is disclosed.
8. Result is auditable where required.

---

## WF-021 — AI action approval

1. AI produces recommendation.
2. User reviews.
3. User accepts/edits.
4. System creates draft action.
5. Normal authorization and validation run.
6. Human approver reviews if required.
7. Current state is revalidated.
8. Action executes through normal application rules.
9. Audit records recommendation, draft, approval and execution.

---

## WF-022 — Audit trail

For important operations:

1. Actor is identified.
2. Organization context is recorded.
3. Action/entity is recorded.
4. Timestamp is recorded.
5. Relevant context is recorded.
6. AI involvement is recorded when applicable.
7. Approval/rejection is recorded where applicable.

---

# 36. State and Transition Rules

## 36.1 Purchase orders

| Current | Action | Result |
|---|---|---|
| Draft | Submit | Pending Approval |
| Pending Approval | Approve | Approved |
| Approved | Partial receipt | Partially Received |
| Partially Received | Complete receipt | Received |
| Draft/Pending/Approved | Cancel | Cancelled |
| Cancelled | Receive | Reject |
| Unapproved | Receive as approved | Reject |

## 36.2 Sales orders

| Current | Action | Result |
|---|---|---|
| Draft | Confirm with sufficient available stock | Confirmed |
| Draft | Confirm without sufficient stock | Reject |
| Confirmed | Fulfill | Completed, subject to DEC-015 |
| Confirmed | Cancel with authorization | Cancelled and reservation released |
| Cancelled | Fulfill | Reject |
| Draft | Query reservation | None |

## 36.3 Transfers

| Current | Action | Result |
|---|---|---|
| Draft | Submit | Pending Approval |
| Pending Approval | Approve | Approved |
| Approved | Dispatch | Dispatched |
| Dispatched | Receive | Received |
| Draft/Pending/Approved | Cancel where allowed | Cancelled |
| Unapproved | Finalize/execute | Reject |

---

# 37. Validation Requirements

The following validations apply across modules.

### FR-VAL-001

All referenced entities must belong to the authenticated organization.

### FR-VAL-002

All warehouse references must be accessible to the acting user.

### FR-VAL-003

Inactive entities shall not be used where the business rules prohibit their use.

### FR-VAL-004

Quantities must be valid positive operational quantities unless a specific adjustment/return direction explicitly supports the operation.

### FR-VAL-005

Stock-consuming operations must validate available stock.

### FR-VAL-006

Business state transitions must reject invalid transitions.

### FR-VAL-007

Approval actions require the appropriate role.

### FR-VAL-008

AI actions must pass the same validation as equivalent human actions.

### FR-VAL-009

Deterministic calculations must be reproducible from recorded source values.

### FR-VAL-010

Cross-organization references shall fail rather than resolve to another tenant.

---

# 38. Error and Failure Behavior

### FR-ERR-001

The system shall reject unauthorized operations.

### FR-ERR-002

The system shall reject operations that violate business state.

### FR-ERR-003

The system shall reject insufficient-stock operations.

### FR-ERR-004

The system shall reject cross-organization access attempts.

### FR-ERR-005

The system shall reject unapproved high-impact operations.

### FR-ERR-006

Concurrent conflicts shall fail gracefully and explain that the current state changed where appropriate.

### FR-ERR-007

AI shall provide a safe limitation/refusal when required data is unavailable or the request exceeds authorization.

### FR-ERR-008

Errors shall not expose sensitive information from another organization.

---

# 39. Non-Functional Functional Implications

The following business expectations must be reflected in later technical design.

### FR-NFR-001 — Performance

Common inventory queries and operational actions should remain responsive at expected SMB scale.

Exact performance targets are a later technical decision.

### FR-NFR-002 — Reliability

Core inventory recording must remain usable even when AI services are unavailable.

### FR-NFR-003 — Security

Authentication, authorization, organization isolation, and warehouse restrictions are mandatory.

### FR-NFR-004 — Auditability

Important operations must be reconstructable.

### FR-NFR-005 — Data integrity

Concurrent operations must not create contradictory stock.

### FR-NFR-006 — Observability

System operators must be able to identify failed jobs, missing notifications, and AI unavailability.

### FR-NFR-007 — Backup/recovery

Operational and audit records must be recoverable.

Exact RPO/RTO remains a later technical decision.

---

# 40. Traceability Matrix

| BRD area | FRD coverage |
|---|---|
| BR-001 to BR-005 | Global rules; inventory; AI; audit |
| BR-010 to BR-022 | Product, warehouse, inventory, purchasing, sales, reporting, AI |
| BR-030 to BR-037 | Inventory, purchasing, sales, transfers, reporting, AI |
| BR-040 to BR-049 | Cross-cutting functional rules and reporting |
| BR-050 to BR-060 | End-to-end workflows, AI, audit, security |
| BR-070 to BR-089 | Sections 5–32 |
| BR-090 to BR-103 | Scope constraints reflected throughout |
| ORG-001 to ORG-009 | Section 5 |
| USR-001 to USR-010 | Section 6 |
| PERM-001 to PERM-004 | Sections 4 and 7 |
| ROLE-001 to ROLE-006 | Section 7 |
| PROD-001 to PROD-015 | Section 8 |
| CAT-001 to CAT-006 | Section 9 |
| BRN-001 to BRN-006 | Section 9 |
| UOM-001 to UOM-006 | Section 10 |
| WH-001 to WH-010 | Section 11 |
| INV-001 to INV-047 | Section 12 |
| SUP-001 to SUP-010 | Section 13 |
| PUR-001 to PUR-015 | Section 14 |
| REC-001 to REC-011 | Section 15 |
| CUS-001 to CUS-008 | Section 16 |
| SAL-001 to SAL-012 | Section 17 |
| RET requirements | Section 18 |
| TRF requirements | Section 19 |
| RPT/KPI requirements | Section 20 |
| NOT requirements | Section 21 |
| AUD requirements | Section 22 |
| AI-001 to AI-060 | Sections 23–34 |
| SEC requirements | Sections 4, 5–7, 11, 20, 22, 23–34 |
| INT requirements | Sections 12, 14–19, 37 |
| KPI requirements | Section 20 |
| WF-001 to WF-022 | Section 35 |

---

# 41. Acceptance Criteria Summary

The initial functional release shall not be considered complete unless the following are demonstrably true:

1. Organizations are isolated.
2. Users receive only permitted roles and warehouse access.
3. Products use unique organization-scoped SKUs.
4. Inactive products cannot enter new sales/purchases.
5. Warehouse access is enforced server-side.
6. Physical, sellable, reserved, available, damaged, and expired stock have consistent meanings.
7. Available stock equals sellable minus reserved.
8. Negative stock is prevented.
9. Every stock change has a traceable movement.
10. Purchase orders follow the required approval path.
11. Receiving updates stock only when the receipt is confirmed.
12. Damaged receipt quantities are not made sellable.
13. Confirmed sales reserve stock.
14. Completed sales deduct stock.
15. Competing sales cannot oversell inventory.
16. Backorders are not accepted.
17. Returns require appropriate inventory classification/inspection.
18. Transfers follow approval and source/destination rules.
19. Cost visibility follows role restrictions.
20. Reports/KPIs respect organization and warehouse permissions.
21. AI answers respect exactly the same authorization boundary.
22. AI does not invent missing operational data.
23. AI distinguishes fact, calculation, prediction, recommendation, and action.
24. AI high-impact writes require human approval.
25. AI actions are revalidated immediately before execution.
26. AI-generated drafts remain auditable.
27. Core inventory operations work without AI availability.
28. Audit history can reconstruct important operations.
29. Open product decisions are not silently resolved by implementation.

---

# 42. Open-Decision Dependencies Before Technical Design

The following decisions should be resolved before final database/API design wherever they materially affect the model:

| Decision | Functional areas affected |
|---|---|
| DEC-001 | Product packaging, feature limits, commercial configuration |
| DEC-002 | Capacity/performance assumptions |
| DEC-011 | Audit/data lifecycle and retention |
| DEC-012 | SUPER_ADMIN/support access |
| DEC-013 | Authentication and organization membership |
| DEC-015 | Sales states, reservations, fulfillment |
| DEC-016 | Invoice behavior |
| DEC-017 | Payment records and document lifecycle |

No implementation should silently turn the recommended defaults for these decisions into confirmed product requirements.

---

# 43. Out-of-Scope Enforcement

The FRD shall not expand the initial product into:

- full accounting/general ledger;
- payroll;
- manufacturing/MRP;
- full logistics/fleet/last-mile;
- public e-commerce;
- full CRM;
- banking integrations;
- complex tax engine;
- unrestricted AI database access;
- full barcode/mobile warehouse operations;
- serial-number tracking;
- backorders;
- multi-language product;
- multi-organization business-user membership.

These remain outside the initial functional baseline unless separately approved.

---

# 44. Next Design Artifacts

After this FRD is reviewed and accepted, the next artifacts should be created in this order:

1. `docs/BUSINESS-RULES.md`
2. `docs/ARCHITECTURE.md`
3. `docs/DATABASE-DESIGN.md`
4. `docs/API-SPECIFICATION.md`
5. `docs/UI-SPECIFICATION.md`
6. Technical implementation plan
7. Application implementation

Application code should not be generated merely from the BRD. The FRD and business rules should be treated as the functional baseline for implementation.

---

# 45. Definition of Done for the FRD

The FRD is ready for technical design when:

- confirmed decisions are reflected consistently;
- unresolved decisions are clearly marked;
- all major BRD functional areas have FRD coverage;
- workflows have explicit success/failure behavior;
- stock terminology is unambiguous;
- permissions are testable;
- AI behavior is bounded and testable;
- approval/revalidation behavior is explicit;
- audit expectations are explicit;
- no technical implementation has been invented unnecessarily;
- traceability to the BRD is available.

