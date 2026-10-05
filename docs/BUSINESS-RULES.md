# Business Rules

**Product:** Inventory Management System with AI Inventory Intelligence  
**Document type:** Business Rules Reference  
**Status:** Draft for technical-design review  
**Source:** `docs/BRD.md`, `docs/REQUIREMENT-DECISIONS.md`, `docs/FRD.md`  
**Revision:** 2026-10-04

---

## 1. Purpose

This document is the compact business-rule authority for implementation.

It extracts the rules that must be enforced consistently across:

- frontend;
- backend;
- database constraints where appropriate;
- service/application logic;
- reports;
- notifications;
- AI tools;
- background jobs.

A UI must not weaken a rule. An AI feature must not bypass a rule.

Where this document refers to an unresolved decision, implementation must not silently invent the missing policy.

---

# 2. Tenant and Organization Rules

### BR-ORG-001 — Organization isolation

Every business record belongs to exactly one organization.

A user must not be able to access another organization's records by changing identifiers, URLs, request parameters, filters, or AI questions.

### BR-ORG-002 — Organization context

Every authenticated business operation executes within an authorized organization context.

Client-supplied organization IDs are not trusted as the sole basis for authorization.

### BR-ORG-003 — Organization-scoped uniqueness

Business identifiers such as product SKU, supplier code, and customer code are unique within an organization unless a later requirement explicitly states otherwise.

### BR-ORG-004 — Base currency

Each organization has one base currency.

The initial system does not provide multi-currency inventory valuation.

### BR-ORG-005 — Initial language

English is the initial product language.

---

# 3. Identity, Roles, and Permissions

## 3.1 Role rules

### BR-AUTH-001

Every operational user has an authenticated identity and an organization membership.

### BR-AUTH-002

Roles:

- SUPER_ADMIN
- ADMIN
- MANAGER
- INVENTORY_STAFF
- SALES_STAFF
- VIEWER

### BR-AUTH-003

ADMIN and MANAGER have access to all warehouses within their organization.

### BR-AUTH-004

INVENTORY_STAFF, SALES_STAFF, and VIEWER may access only warehouses assigned to them.

### BR-AUTH-005

Warehouse restrictions apply to all relevant operations, not only inventory screens.

This includes:

- inventory;
- purchases/receiving;
- sales;
- transfers;
- reports;
- notifications;
- AI results.

### BR-AUTH-006

Backend authorization is authoritative. Frontend hiding is not a security mechanism.

### BR-AUTH-007

Deactivated users cannot perform new operational actions.

### BR-AUTH-008

A tenant user cannot grant themselves SUPER_ADMIN privileges.

### BR-AUTH-009

Cost information is visible by default only to:

- ADMIN
- MANAGER

INVENTORY_STAFF, SALES_STAFF, and VIEWER do not receive cost visibility by default.

### BR-AUTH-010

AI must enforce the same role, organization, warehouse, and cost permissions as ordinary application access.

---

# 4. Product Rules

### BR-PROD-001 — SKU uniqueness

SKU must be unique within the organization.

### BR-PROD-002 — No variant matrix

The initial system does not implement a parent-product/variant matrix.

If two stockable items require separate inventory tracking, they are separate SKUs/products.

### BR-PROD-003 — Product activation

A product must contain the required operational information before becoming Active.

### BR-PROD-004 — Inactive products

Inactive products:

- cannot be added to new sales;
- cannot be added to new purchase orders;
- remain visible in historical transactions;
- remain visible in inventory history;
- remain visible in reports/audit where relevant.

### BR-PROD-005 — Historical integrity

A product with historical business records must not be physically deleted in a way that breaks those records.

### BR-PROD-006 — Reorder ownership

User-maintained reorder points remain under user/business control.

AI may recommend changes but cannot silently overwrite them.

---

# 5. Category and Brand Rules

### BR-CAT-001

Categories are organization-scoped.

### BR-CAT-002

Products may optionally belong to a category.

### BR-CAT-003

Categories in active use cannot be casually deleted.

### BR-BRN-001

Brands are organization-scoped.

### BR-BRN-002

Products may optionally belong to a brand.

### BR-BRN-003

Brands in active use cannot be casually deleted.

---

# 6. Unit Rules

### BR-UOM-001

Each product has a defined stocking unit.

### BR-UOM-002

Unit conversions use explicit deterministic conversion factors.

Example:

`1 Box = 12 Pieces`

### BR-UOM-003

Inventory quantities are maintained in the product's stocking unit.

### BR-UOM-004

AI must never determine or invent a unit conversion.

### BR-UOM-005

The initial system does not implement a complex universal unit-of-measure engine.

---

# 7. Warehouse Rules

### BR-WH-001

Every inventory position belongs to one organization and one warehouse.

### BR-WH-002

Warehouse code is unique within an organization.

### BR-WH-003

Source and destination warehouses for a transfer must belong to the same organization.

### BR-WH-004

A transfer cannot use the same warehouse as both source and destination.

### BR-WH-005

Warehouse permissions must be checked for both source and destination operations.

### BR-WH-006

A default warehouse is a convenience setting only; it never overrides authorization.

---

# 8. Inventory Quantity Model

This section is authoritative.

## 8.1 Physical stock

### BR-INV-001

Physical Stock is the total physical quantity recorded for a product/warehouse.

It includes:

- sellable stock;
- damaged stock;
- expired stock.

## 8.2 Sellable stock

### BR-INV-002

Sellable Stock is physical inventory currently fit for normal sale.

## 8.3 Reserved stock

### BR-INV-003

Reserved Stock is sellable inventory committed to confirmed transactions according to reservation rules.

Reserved Stock is part of Sellable Stock.

It is not an additional physical quantity.

## 8.4 Available stock

### BR-INV-004

Available Stock is sellable stock that is not reserved.

Formula:

`Available Stock = Sellable Stock - Reserved Stock`

Available Stock is derived and must not be independently edited.

## 8.5 Damaged stock

### BR-INV-005

Damaged Stock is physical inventory that is not currently sellable because it is damaged.

Damaged Stock is not Available Stock.

## 8.6 Expired stock

### BR-INV-006

Expired Stock is physical inventory that is not currently sellable because it has expired.

Expired Stock is not Available Stock.

## 8.7 Quantity relationship

### BR-INV-007

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

# 9. Inventory Ledger Rules

### BR-LEDGER-001

Every stock-changing operation creates a recognized inventory movement.

### BR-LEDGER-002

Recognized movement types include:

- OPENING
- PURCHASE
- SALE
- PURCHASE_RETURN
- SALES_RETURN
- ADJUSTMENT_IN
- ADJUSTMENT_OUT
- TRANSFER_IN
- TRANSFER_OUT
- DAMAGE
- EXPIRED

### BR-LEDGER-003

Stock may not be changed by silently overwriting an inventory quantity.

### BR-LEDGER-004

Every movement must be traceable to a business reason/source where applicable.

### BR-LEDGER-005

Inventory history is not ordinary editable business data.

### BR-LEDGER-006

A multi-step business operation that changes both a document and stock must succeed or fail as one business transaction from the user's perspective.

---

# 10. Negative Stock and Concurrency

### BR-STOCK-001

Negative stock is not allowed.

### BR-STOCK-002

A stock-consuming operation must have sufficient Available Stock at the time of final confirmation.

### BR-STOCK-003

Availability must be checked and revalidated inside the final business transaction.

### BR-STOCK-004

Concurrent transactions must not consume the same available quantity twice.

### BR-STOCK-005

If a competing transaction has consumed the required stock first, the later transaction fails gracefully.

### BR-STOCK-006

The system must not rely only on a frontend availability check.

---

# 11. Opening Stock

### BR-OPEN-001

Opening stock may be entered/imported only by an authorized user.

### BR-OPEN-002

Opening stock must reference:

- organization;
- warehouse;
- product;
- quantity;
- stock classification where applicable;
- reason/reference;
- actor.

### BR-OPEN-003

Opening stock creates inventory movement history.

### BR-OPEN-004

Opening stock must not create negative inventory.

### BR-OPEN-005

Opening-stock operations respect warehouse permissions.

---

# 12. Inventory Adjustments

### BR-ADJ-001

An adjustment must have an identifiable business reason.

### BR-ADJ-002

An adjustment must specify the affected product and warehouse.

### BR-ADJ-003

An adjustment must create an inventory movement.

### BR-ADJ-004

An adjustment cannot create negative sellable/available stock.

### BR-ADJ-005

AI-generated adjustments are drafts until the required human approval is completed.

---

# 13. Damage and Expiry

### BR-DMG-001

Damaged goods are not sellable.

### BR-DMG-002

Expired goods are not sellable.

### BR-DMG-003

Moving sellable inventory into damaged/expired classification must create traceable inventory history.

### BR-DMG-004

Damaged/expired quantities must not increase Available Stock.

---

# 14. Batch and Expiry

### BR-BATCH-001

The initial system supports batch/lot tracking where relevant.

### BR-BATCH-002

Expiry dates may be recorded for applicable stock.

### BR-BATCH-003

Batch/expiry information must remain traceable through relevant receiving, inventory, sales, returns, reports, and AI analysis.

### BR-BATCH-004

Serial-number tracking is out of initial scope.

---

# 15. Inventory Valuation

### BR-COST-001

Inventory valuation uses Weighted Average Cost.

### BR-COST-002

Inventory value is calculated from recorded cost data.

### BR-COST-003

AI must not invent inventory costs or valuations.

### BR-COST-004

Cost visibility follows role permissions.

---

# 16. Supplier Rules

### BR-SUP-001

Supplier records are organization-scoped.

### BR-SUP-002

Supplier code is unique within an organization.

### BR-SUP-003

Inactive suppliers cannot be selected for new purchase orders.

### BR-SUP-004

Supplier recommendations use only available organization data.

### BR-SUP-005

Missing supplier information must not be invented.

---

# 17. Purchase Order Rules

### BR-PO-001

A purchase order must identify a supplier and intended warehouse.

### BR-PO-002

PO lines must reference valid products.

### BR-PO-003

PO quantities must be valid.

### BR-PO-004

Draft purchase orders do not affect inventory.

### BR-PO-005

Submitted purchase orders require approval.

### BR-PO-006

ADMIN or MANAGER may approve purchase orders.

### BR-PO-007

The initial system does not use configurable monetary approval thresholds.

### BR-PO-008

An unapproved purchase order cannot be treated as an approved stock-intake document.

### BR-PO-009

Cancelled purchase orders cannot be received.

### BR-PO-010

AI-generated purchase orders follow exactly the same approval and validation rules as human-generated purchase orders.

---

# 18. Purchase Receiving Rules

### BR-REC-001

Goods are normally received against an approved purchase order.

### BR-REC-002

Receiving must respect warehouse permissions.

### BR-REC-003

Receiving supports partial receipts.

### BR-REC-004

Receiving sellable stock increases Physical Stock and Sellable Stock (recorded as `sellable_quantity`). Available Stock is derived from `sellable_quantity - reserved_quantity` (it is not directly incremented).

Reserved Stock is unchanged by normal purchase receiving.

### BR-REC-005

Damaged received quantity increases non-sellable Damaged Stock (`damaged_quantity`); it does not increase `sellable_quantity` or derived Available Stock.

### BR-REC-006

Receiving creates receipt and movement history.

### BR-REC-007

Batch/lot and expiry are captured where applicable.

### BR-REC-008

Over-receiving is exceptional and is not an uncontrolled normal operation.

---

# 19. Customer Rules

### BR-CUST-001

Customer records are organization-scoped.

### BR-CUST-002

Customer code is unique within an organization.

### BR-CUST-003

Inactive customers cannot be selected for new sales.

### BR-CUST-004

Customer records are not a full CRM.

---

# 20. Sales Rules

### BR-SALES-001

Sales must reference an authorized warehouse.

### BR-SALES-002

Draft sales do not reserve stock.

### BR-SALES-003

Confirmed sales reserve stock.

### BR-SALES-004

On confirmation:

- Reserved Stock increases;
- Available Stock decreases;
- Physical Stock remains unchanged;
- Sellable Stock remains unchanged.

### BR-SALES-005

A sale cannot be confirmed if sufficient Available Stock does not exist.

### BR-SALES-006

Backorders are not supported.

### BR-SALES-007

On fulfillment/completion:

- Physical Stock decreases;
- Sellable Stock decreases;
- the fulfilled reservation is released;
- a SALE movement is created.

### BR-SALES-008

Normal sales do not require manual approval.

### BR-SALES-009

Exceptional high-impact sales operations such as configured cancellations/overrides require appropriate MANAGER/ADMIN authorization.

### BR-SALES-010

Concurrent sales must be handled transactionally so that only sufficient-stock operations succeed.

### BR-SALES-011

Partial fulfillment remains unresolved under DEC-015 and must not be silently defined differently by implementation.

---

# 21. Reservation Rules

### BR-RES-001

Confirmed sales are reservation sources.

### BR-RES-002

Draft sales are not reservation sources.

### BR-RES-003

Purchase orders do not reserve sellable stock.

### BR-RES-004

Approved transfers may reserve source sellable stock where required by the transfer workflow.

### BR-RES-005

Reservation must not create additional physical stock.

### BR-RES-006

Reservations must be released or consumed consistently when their source transaction is completed/cancelled.

---

# 22. Customer Return Rules

### BR-CRET-001

Draft customer returns do not change stock.

### BR-CRET-002

Returned goods require inspection before classification.

### BR-CRET-003

Accepted returned goods may return to sellable stock.

### BR-CRET-004

Damaged returns remain non-sellable.

### BR-CRET-005

Expired returns remain non-sellable where applicable.

### BR-CRET-006

Customer-return quantities cannot exceed eligible original quantities without an authorized exception.

### BR-CRET-007

Return inventory changes create SALES_RETURN movement history.

---

# 23. Purchase Return Rules

### BR-PRET-001

Draft purchase returns do not change stock.

### BR-PRET-002

Purchase-return stock impact occurs on confirmation/dispatch according to the workflow.

### BR-PRET-003

A purchase return cannot exceed eligible received quantities without an authorized exception.

### BR-PRET-004

Purchase returns create PURCHASE_RETURN movement history.

---

# 24. Transfer Rules

### BR-TRF-001

Warehouse-to-warehouse movement uses a transfer document.

### BR-TRF-002

Source and destination are different warehouses within the same organization.

### BR-TRF-003

Transfers require MANAGER or ADMIN approval before dispatch/final execution.

### BR-TRF-004

Transfer operations respect permissions for both source and destination.

### BR-TRF-005

Dispatch reduces source inventory.

### BR-TRF-006

Undelivered dispatched stock is considered in transit and is not ordinary available stock at either warehouse.

### BR-TRF-007

Receipt increases destination inventory.

### BR-TRF-008

Batch/lot and expiry information follows the transferred goods where relevant.

### BR-TRF-009

AI-generated transfers require the same human approval and business controls as human-generated transfers.

### BR-TRF-010

Partial transfer behavior follows the current BRD assumption; implementation must not silently introduce a new partial-transfer policy.

---

# 25. Tax and Commercial Amount Rules

### BR-TAX-001

The initial system supports basic optional tax fields.

### BR-TAX-002

There is no full tax engine in the initial scope.

### BR-TAX-003

Commercial totals are deterministic calculations from recorded line values and applicable tax fields.

### BR-TAX-004

AI must not replace deterministic invoice/tax arithmetic.

### BR-TAX-005

Full statutory e-invoicing/accounting behavior is outside the initial inventory scope.

---

# 26. Reporting Rules

### BR-RPT-001

Reports use recorded operational data.

### BR-RPT-002

Reports never mix organizations.

### BR-RPT-003

Warehouse-restricted users see only permitted warehouse data.

### BR-RPT-004

Cost reports are restricted by role.

### BR-RPT-005

KPI values are deterministic calculations or defined operational counts.

### BR-RPT-006

AI may explain reports but cannot invent underlying values.

### BR-RPT-007

Warehouse KPI is "Warehouse Inventory Distribution" unless warehouse capacity is explicitly introduced later.

---

# 27. Notification Rules

### BR-NOT-001

Notifications are organization-scoped.

### BR-NOT-002

Notifications respect warehouse permissions.

### BR-NOT-003

Low-stock alerts use configured thresholds.

### BR-NOT-004

Out-of-stock means Available Stock has reached zero for the applicable active tracked product/warehouse.

### BR-NOT-005

Damaged and expired stock are not counted as Available Stock.

### BR-NOT-006

AI-generated alerts are labelled as AI-generated where relevant.

### BR-NOT-007

Notification content must not expose unauthorized business information.

---

# 28. Audit Rules

### BR-AUD-001

Important business actions are auditable.

### BR-AUD-002

Audit records include, at minimum where applicable:

- organization;
- actor;
- action;
- entity;
- timestamp;
- relevant business context.

### BR-AUD-003

Stock-changing operations are traceable through inventory movements and audit history.

### BR-AUD-004

Audit history cannot be casually edited or deleted by ordinary business users.

### BR-AUD-005

AI involvement is visible in relevant history.

### BR-AUD-006

AI recommendation, draft, approval/rejection, and execution stages are distinguishable.

---

# 29. AI Rules

## 29.1 Permission boundary

### BR-AI-001

AI can only access data the requesting user can access.

### BR-AI-002

AI cannot cross organization boundaries.

### BR-AI-003

AI respects warehouse restrictions.

### BR-AI-004

AI respects cost visibility.

### BR-AI-005

AI is not a database administrator and cannot run unconstrained data exploration.

---

## 29.2 AI statement integrity

### BR-AI-006

AI outputs must distinguish:

- FACT;
- CALCULATION;
- PREDICTION;
- RECOMMENDATION;
- ACTION.

### BR-AI-007

Predictions and recommendations cannot be represented as facts.

### BR-AI-008

Missing information must be disclosed.

### BR-AI-009

AI must not invent operational values.

---

## 29.3 AI freshness

### BR-AI-010

AI must communicate relevant data freshness.

### BR-AI-011

Historical/cached/stale information must not be presented as real-time information.

### BR-AI-012

Recommendations may become stale and must be revalidated before high-impact execution.

---

## 29.4 AI recommendations

### BR-AI-013

AI may recommend reorder quantities, suppliers, stock actions, and other supported decisions.

### BR-AI-014

Numeric recommendations should be grounded in deterministic application calculations whenever possible.

### BR-AI-015

AI does not silently change business settings such as reorder points.

---

## 29.5 AI anomaly detection

### BR-AI-016

An anomaly is a signal for investigation, not proof of fraud or error.

### BR-AI-017

Anomaly detection does not change stock or documents automatically.

---

# 30. AI Action and Approval Rules

### BR-ACTION-001

AI-generated high-impact writes require explicit human approval.

### BR-ACTION-002

Examples include:

- purchase-order creation/modification;
- reorder-setting changes;
- inventory adjustments;
- stock transfers;
- other material operational writes.

### BR-ACTION-003

The AI cannot approve its own action.

### BR-ACTION-004

AI-generated actions must pass normal application authorization and business validation.

### BR-ACTION-005

Before execution, the system revalidates current:

- stock;
- document state;
- permissions;
- relevant business constraints.

### BR-ACTION-006

A stale AI proposal cannot be executed without successful revalidation.

### BR-ACTION-007

AI recommendation acceptance creates a draft/action proposal; it does not automatically execute the business operation.

### BR-ACTION-008

AI-generated actions remain identifiable in audit/history.

### BR-ACTION-009

Rejected AI actions create no operational stock/order effect.

---

# 31. AI Prompt and Content Trust

### BR-PROMPT-001

Stored business text is untrusted content.

This includes:

- product descriptions;
- supplier names;
- customer text;
- notes;
- other user-entered free text.

### BR-PROMPT-002

Stored text cannot override:

- authorization;
- organization isolation;
- business rules;
- approval requirements;
- system safety controls.

---

# 32. Data Integrity Rules

### BR-DATA-001

Inventory must remain reconcilable to recognized movements from a known opening/prior position.

### BR-DATA-002

Document and inventory state must remain consistent.

### BR-DATA-003

Approval state must match the business action permitted by that state.

### BR-DATA-004

Draft documents must not produce effects reserved for confirmed/approved documents.

### BR-DATA-005

Cancelled transactions must release/reverse applicable reservations or pending effects according to their business state.

### BR-DATA-006

Concurrent operations must preserve all inventory invariants.

---

# 33. Unresolved Decisions

The following rules must remain open until their corresponding decision is confirmed.

## BR-OPEN-001 — DEC-001

Commercial pricing model is unresolved.

No pricing-plan enforcement should be invented in the operational domain.

## BR-OPEN-002 — DEC-002

Organization scale targets are unresolved.

Do not invent hard capacity limits.

## BR-OPEN-003 — DEC-011

Data-retention period is unresolved.

Do not hard-code a business retention period in the application requirements.

## BR-OPEN-004 — DEC-012

SUPER_ADMIN access to tenant operational data is unresolved.

Do not assume unrestricted support access.

## BR-OPEN-005 — DEC-013

Multi-organization users are unresolved.

The initial FRD assumes a single primary organization membership unless this decision changes.

## BR-OPEN-006 — DEC-015

Partial sales fulfillment policy is unresolved.

Do not invent final reservation/release behavior for partial fulfillment beyond the confirmed general rules.

## BR-OPEN-007 — DEC-016

Invoicing depth is unresolved.

Do not implement a statutory/full invoicing system based only on assumptions.

## BR-OPEN-008 — DEC-017

Payment-recording depth is unresolved.

Do not implement a full payment/accounting subsystem based only on assumptions.

---

# 34. Implementation Enforcement Priorities

When rules overlap, implementation should enforce them in this order:

1. Organization isolation
2. Authentication
3. Authorization and warehouse scope
4. Business document state
5. Inventory availability/integrity
6. Approval requirements
7. Transaction/concurrency protection
8. Audit requirements
9. AI-specific restrictions
10. User-facing convenience behavior

Convenience must never override a higher-priority rule.

---

# 35. Core Invariants

The following invariants must always hold.

### INV-I-001

```text
Available Stock = Sellable Stock - Reserved Stock
```

### INV-I-002

```text
Physical Stock = Sellable Stock + Damaged Stock + Expired Stock
```

### INV-I-003

```text
Sellable Stock = Available Stock + Reserved Stock
```

### INV-I-004

```text
Available Stock >= 0
Sellable Stock >= 0
Physical Stock >= 0
Damaged Stock >= 0
Expired Stock >= 0
Reserved Stock >= 0
```

### INV-I-005

Reserved Stock cannot exceed Sellable Stock.

### INV-I-006

Damaged Stock and Expired Stock cannot be included in Available Stock.

### INV-I-007

Every confirmed stock change has a recognized movement.

### INV-I-008

No operation may bypass organization or warehouse authorization.

### INV-I-009

No AI write may bypass normal business validation.

### INV-I-010

No AI high-impact write may execute without required human approval.

### INV-I-011

A concurrent stock-consuming operation cannot make the final available quantity negative.

### INV-I-012

Reports and AI results cannot expose information outside the user's authorized data scope.

---

# 36. Rule-to-Module Reference

| Module | Primary rules |
|---|---|
| Organizations | BR-ORG |
| Authentication/RBAC | BR-AUTH |
| Products | BR-PROD |
| Categories | BR-CAT |
| Brands | BR-BRN |
| Units | BR-UOM |
| Warehouses | BR-WH |
| Inventory | BR-INV, BR-LEDGER, BR-STOCK |
| Opening Stock | BR-OPEN |
| Adjustments | BR-ADJ |
| Damage/Expiry | BR-DMG, BR-BATCH |
| Valuation | BR-COST |
| Suppliers | BR-SUP |
| Purchasing | BR-PO |
| Receiving | BR-REC |
| Customers | BR-CUST |
| Sales | BR-SALES, BR-RES |
| Customer Returns | BR-CRET |
| Purchase Returns | BR-PRET |
| Transfers | BR-TRF |
| Tax/Commercial | BR-TAX |
| Reporting | BR-RPT |
| Notifications | BR-NOT |
| Audit | BR-AUD |
| AI Copilot | BR-AI |
| AI Actions | BR-ACTION |
| AI Prompt Security | BR-PROMPT |
| Data Integrity | BR-DATA |

---

# 37. Rule Change Governance

Any change to a confirmed business rule must first be reflected in the requirements documentation.

Do not silently change business behavior through:

- database implementation;
- API implementation;
- frontend logic;
- AI prompts;
- background jobs.

A changed rule must be reviewed for its effect on:

- BRD;
- FRD;
- database model;
- API behavior;
- UI;
- tests;
- reports;
- AI tools;
- audit behavior.

---

# 38. Definition of Done

The business-rules artifact is ready for technical design when:

- every major operational invariant is explicit;
- stock terminology is unambiguous;
- role and warehouse restrictions are explicit;
- document-state effects are explicit;
- approval boundaries are explicit;
- concurrency expectations are explicit;
- AI boundaries are explicit;
- unresolved decisions remain unresolved;
- no rule contradicts the BRD/FRD;
- the rules can be converted into automated tests.

