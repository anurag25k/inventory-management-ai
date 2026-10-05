# DATABASE-DESIGN.md

# Inventory Management AI — Database Design

**Document Status:** Approved for implementation planning  
**Document Type:** Database Design Specification  
**Version:** 1.0  
**Date:** 2026-10-04

---

## 1. Purpose

This document defines the relational database architecture for the Inventory Management AI system.

It translates the approved:

1. BRD
2. Requirement Decisions
3. FRD
4. Business Rules
5. Architecture

into a PostgreSQL-oriented data model.

This document defines:

- entities
- relationships
- ownership
- organization scoping
- keys
- constraints
- indexes
- inventory state
- inventory movements
- reservations
- batches/lots
- expiry
- weighted average cost
- purchasing
- receiving
- sales
- returns
- transfers
- approvals
- notifications
- audit
- AI persistence

It does **not** define final SQL migrations yet.

---

# 2. Database Principles

## DB-001 — PostgreSQL is the source of truth

PostgreSQL is authoritative for transactional application data.

Redis, AI context, browser state, and cached responses are not authoritative.

---

## DB-002 — Organization isolation is mandatory

Tenant-owned records must be associated with an organization directly or through a controlled ownership relationship.

---

## DB-003 — Foreign keys must be used

Application-level relationships should be backed by PostgreSQL foreign keys wherever appropriate.

---

## DB-004 — Business invariants should be protected at database level where practical

Use:

- NOT NULL
- UNIQUE
- composite UNIQUE
- CHECK
- FOREIGN KEY
- indexes

Application services remain responsible for multi-step business rules.

---

## DB-005 — Inventory is transactional

Inventory mutations must occur inside database transactions.

---

## DB-006 — Available stock is derived

The business invariant is:

```text
Available Stock = Sellable Stock - Reserved Stock
```

Available stock must not become an independent conflicting source of truth.

---

## DB-007 — No negative stock

Quantities must never become negative.

This is enforced through application logic and appropriate database constraints.

---

## DB-008 — Inventory movement is auditable

Every confirmed stock-changing operation must produce a recognized inventory movement.

---

## DB-009 — AI cannot bypass the data model

AI actions must use application services and normal persistence rules.

---

# 3. Database Technology

Recommended:

```text
PostgreSQL
```

Use:

- UUID primary keys for application entities
- `timestamptz` for timestamps
- `numeric` for money
- `numeric` for quantities where fractional units are supported
- explicit timezone-aware timestamps
- PostgreSQL indexes and constraints
- Django migrations as schema authority

---

# 4. Identifier Strategy

## 4.1 Primary keys

Application/domain entities should use UUID primary keys.

Benefits:

- safer external identifiers
- less predictable than sequential IDs
- easier distributed workflows
- suitable for future integrations

Example:

```text
id UUID PRIMARY KEY
```

---

## 4.2 Human-readable codes

Operational entities should also have human-readable organization-scoped codes where useful.

Examples:

```text
Product: PROD-000123
Supplier: SUP-000045
Customer: CUST-000321
Purchase Order: PO-000102
Sales Order: SO-000211
Transfer: TRF-000031
```

The exact code-generation strategy belongs to implementation.

---

# 5. Timestamp Convention

Use:

```text
created_at timestamptz NOT NULL
updated_at timestamptz NOT NULL
```

for mutable entities.

Business-event timestamps should also use `timestamptz`.

Examples:

```text
approved_at
confirmed_at
completed_at
received_at
cancelled_at
expires_at
```

Store UTC at the database/application boundary and localize for display.

---

# 6. Organization Model

## Table: organizations

Purpose:

Represents a tenant/business organization.

### Core fields

```text
id UUID PK
name
code
base_currency
status
created_at
updated_at
```

### Relationships

```text
organization
 ├── users/memberships
 ├── products
 ├── categories
 ├── brands
 ├── units
 ├── warehouses
 ├── suppliers
 ├── customers
 ├── inventory
 ├── purchase orders
 ├── sales
 ├── transfers
 ├── returns
 ├── notifications
 ├── audit events
 └── AI records
```

### Constraints

```text
code unique
name NOT NULL
base_currency NOT NULL
status NOT NULL
```

Base currency is one per organization.

The final commercial pricing model remains unresolved under DEC-001.

---

# 7. User and Membership Model

The architecture should use an organization membership abstraction.

## Table: users

Represents a platform identity.

Conceptual fields:

```text
id UUID PK
email
password_hash / authentication-managed credentials
first_name
last_name
is_active
created_at
updated_at
```

---

## Table: organization_memberships

Represents a user's membership in an organization.

Fields:

```text
id UUID PK
organization_id FK
user_id FK
role
is_active
created_at
updated_at
```

### Constraint

```text
UNIQUE(organization_id, user_id)
```

This structure remains compatible with DEC-013 if multi-organization users are enabled later.

---

# 8. Role Model

Initial roles:

```text
SUPER_ADMIN
ADMIN
MANAGER
INVENTORY_STAFF
SALES_STAFF
VIEWER
```

A role may initially be represented as a controlled enum/choice.

If a future permission-management product is required, this can evolve into:

```text
roles
permissions
role_permissions
organization_memberships
```

No custom permission-builder requirement is assumed yet.

---

# 9. Warehouse Assignment

## Table: membership_warehouses

Associates restricted organization members with warehouses.

Fields:

```text
id UUID PK
membership_id FK
warehouse_id FK
created_at
```

Constraint:

```text
UNIQUE(membership_id, warehouse_id)
```

This supports:

```text
INVENTORY_STAFF
SALES_STAFF
VIEWER
```

warehouse restrictions.

ADMIN/MANAGER access to all organization warehouses is a business authorization rule and does not require an assignment row for every warehouse.

---

# 10. Product Catalog

## Table: products

Fields:

```text
id UUID PK
organization_id FK
sku
name
description
category_id FK nullable
brand_id FK nullable
unit_id FK
status
reorder_point
reorder_quantity nullable
default_cost nullable
default_sale_price nullable
tax_rate nullable
created_at
updated_at
```

### Product lifecycle

```text
DRAFT
ACTIVE
INACTIVE
ARCHIVED
```

Inactive products:

- remain in history
- remain in reports
- remain in inventory/history
- cannot enter new sales
- cannot enter new purchases

---

# 11. Product SKU Rule

Confirmed business rule:

> Separately tracked inventory items use separate SKUs.

No product-variant matrix is required in the initial system.

Therefore:

```text
Product
   └── SKU
```

rather than:

```text
Product
 └── Variant
      └── SKU
```

---

# 12. Product Constraints

Recommended:

```text
UNIQUE(organization_id, sku)
```

Additional checks:

```text
reorder_point >= 0
reorder_quantity >= 0
default_cost >= 0
default_sale_price >= 0
tax_rate >= 0
```

Whether all commercial fields are mandatory depends on the final workflow requirements.

---

# 13. Categories

## Table: categories

Fields:

```text
id UUID PK
organization_id FK
name
description nullable
is_active
created_at
updated_at
```

Constraint:

```text
UNIQUE(organization_id, name)
```

---

# 14. Brands

## Table: brands

Fields:

```text
id UUID PK
organization_id FK
name
description nullable
is_active
created_at
updated_at
```

Constraint:

```text
UNIQUE(organization_id, name)
```

---

# 15. Units of Measure

## Table: units

Fields:

```text
id UUID PK
organization_id FK
name
symbol
is_base_unit
created_at
updated_at
```

Example:

```text
Piece
Box
Kg
Gram
Liter
```

---

# 16. Unit Conversion

## Table: unit_conversions

Fields:

```text
id UUID PK
organization_id FK
from_unit_id FK
to_unit_id FK
conversion_factor
created_at
updated_at
```

Example:

```text
1 Box = 12 Pieces
```

represented as an explicit conversion factor.

The initial system supports simple explicit conversions.

It does not require a general unit-conversion engine.

---

# 17. Warehouse Model

## Table: warehouses

Fields:

```text
id UUID PK
organization_id FK
code
name
address_line_1 nullable
address_line_2 nullable
city nullable
state nullable
postal_code nullable
country nullable
is_active
created_at
updated_at
```

Constraint:

```text
UNIQUE(organization_id, code)
```

No warehouse capacity field is assumed because the approved requirements do not define warehouse physical capacity.

---

# 18. Inventory Position

The inventory position represents product stock at a warehouse.

## Table: inventory_positions

Fields:

```text
id UUID PK
organization_id FK
warehouse_id FK
product_id FK
sellable_quantity
reserved_quantity
damaged_quantity
expired_quantity
created_at
updated_at
```

### Core invariant

```text
available_quantity =
    sellable_quantity - reserved_quantity
```

`available_quantity` should be derived rather than independently stored.

### Constraints

```text
sellable_quantity >= 0
reserved_quantity >= 0
damaged_quantity >= 0
expired_quantity >= 0
reserved_quantity <= sellable_quantity
```

### Uniqueness

```text
UNIQUE(warehouse_id, product_id)
```

Organization consistency must also be enforced by application/domain validation.

---

# 19. Why Available Quantity Is Not Stored

Avoid:

```text
sellable_quantity
reserved_quantity
available_quantity
```

as three independently mutable values.

Instead:

```text
sellable_quantity
reserved_quantity
```

are state values and:

```text
available = sellable - reserved
```

is derived.

This prevents drift such as:

```text
sellable = 100
reserved = 20
available = 75
```

which would violate the business rule.

---

# 20. Inventory Batches / Lots

Batch/lot tracking is supported initially.

## Table: inventory_batches

Fields:

```text
id UUID PK
organization_id FK
product_id FK
warehouse_id FK
batch_number
manufactured_at nullable
expiry_date nullable
status
created_at
updated_at
```

Possible status:

```text
ACTIVE
EXPIRED
QUARANTINED
CLOSED
```

Exact batch lifecycle may be refined during implementation.

---

# 21. Batch Quantity

If batch-level quantity must be independently tracked, use a separate relation.

## Table: inventory_batch_positions

Fields:

```text
id UUID PK
organization_id FK
batch_id FK
inventory_position_id FK
sellable_quantity
reserved_quantity
damaged_quantity
expired_quantity
created_at
updated_at
```

Constraint:

```text
UNIQUE(batch_id, inventory_position_id)
```

The batch-level model must preserve the same inventory invariants.

---

# 22. Batch Expiry Rule

Expired inventory cannot be part of available sellable stock.

Conceptually:

```text
Expiry detected
      ↓
Stock classified as expired
      ↓
Expired quantity increases
      ↓
Sellable/available quantity is reduced accordingly
      ↓
Movement recorded
```

Expiry processing must use the normal inventory service.

---

# 23. Inventory Movement Ledger

## Table: inventory_movements

This is the historical stock ledger.

Fields:

```text
id UUID PK
organization_id FK
warehouse_id FK
product_id FK
batch_id FK nullable
movement_type
quantity
unit_cost nullable
reference_type nullable
reference_id nullable
occurred_at
created_by FK nullable
created_at
```

Movement types:

```text
OPENING
PURCHASE
SALE
PURCHASE_RETURN
SALES_RETURN
ADJUSTMENT_IN
ADJUSTMENT_OUT
TRANSFER_IN
TRANSFER_OUT
DAMAGE
EXPIRED
```

---

# 24. Movement Quantity Convention

Store movement quantity as a positive magnitude.

The movement type determines direction.

Example:

```text
PURCHASE +10
SALE 10
```

The movement itself records `quantity = 10`; direction is interpreted by the movement type.

This avoids ambiguous negative movement values.

If implementation instead chooses signed quantities, the convention must be globally consistent. The initial design recommends positive magnitude + movement type.

---

# 25. Movement References

Inventory movements should reference their originating business object.

Examples:

```text
SALE → sales_order.id
PURCHASE → purchase_order / receipt.id
TRANSFER_OUT → transfer.id
TRANSFER_IN → transfer.id
ADJUSTMENT_IN → adjustment.id
```

A generic:

```text
reference_type
reference_id
```

can provide flexibility, but critical workflows should also have explicit relational integrity where practical.

---

# 26. Inventory Adjustment

## Table: inventory_adjustments

Fields:

```text
id UUID PK
organization_id FK
warehouse_id FK
product_id FK
batch_id FK nullable
adjustment_type
quantity
reason
status
created_by FK
approved_by FK nullable
created_at
approved_at nullable
```

Adjustment types:

```text
IN
OUT
```

Adjustment rules:

- authorized users only
- no negative resulting stock
- movement required
- audit required

---

# 27. Opening Stock

Opening stock is a controlled inventory operation.

Possible persistence:

```text
inventory_adjustments
+
inventory_movements
```

or a dedicated opening-stock import model.

The final implementation should ensure opening stock is not silently inserted directly into inventory balances.

---

# 28. Weighted Average Cost Model

Weighted Average Cost requires historical cost data.

The database should retain enough information to calculate or audit cost changes.

Relevant fields:

```text
inventory_movements.unit_cost
inventory_positions / cost state
```

A dedicated cost-state table may be used.

## Optional table: inventory_cost_positions

```text
id UUID PK
organization_id FK
warehouse_id FK
product_id FK
average_unit_cost
cost_quantity
updated_at
```

The implementation must ensure cost state and inventory state remain transactionally consistent.

---

# 29. Cost Update Concept

For a purchase:

```text
new_average_cost =
(
    existing_quantity × existing_average_cost
    +
    received_quantity × received_unit_cost
)
/
(existing_quantity + received_quantity)
```

This calculation is deterministic backend logic.

The LLM must not calculate or persist authoritative cost.

---

# 30. Suppliers

## Table: suppliers

Fields:

```text
id UUID PK
organization_id FK
supplier_code
name
email nullable
phone nullable
address nullable
tax_identifier nullable
is_active
created_at
updated_at
```

Constraint:

```text
UNIQUE(organization_id, supplier_code)
```

Supplier names may be duplicated unless a stronger uniqueness rule is introduced later.

---

# 31. Purchase Orders

## Table: purchase_orders

Fields:

```text
id UUID PK
organization_id FK
supplier_id FK
warehouse_id FK
po_number
status
order_date
expected_date nullable
subtotal
tax_amount
total_amount
notes nullable
created_by FK
approved_by FK nullable
approved_at nullable
created_at
updated_at
```

Suggested statuses:

```text
DRAFT
SUBMITTED
APPROVED
PARTIALLY_RECEIVED
RECEIVED
CANCELLED
```

The exact partial receiving semantics should be aligned with the FRD during implementation.

---

# 32. Purchase Order Items

## Table: purchase_order_items

Fields:

```text
id UUID PK
purchase_order_id FK
product_id FK
unit_id FK
ordered_quantity
unit_cost
tax_rate nullable
line_subtotal
line_tax
line_total
received_quantity
created_at
updated_at
```

Constraints:

```text
ordered_quantity > 0
unit_cost >= 0
received_quantity >= 0
received_quantity <= ordered_quantity
```

If future business decisions permit receiving beyond ordered quantity, this constraint must be revisited explicitly.

---

# 33. Purchase Approval

Purchase order approval can use the general approval model.

The purchase order itself remains the business object.

Approval record references:

```text
purchase_order
```

and records:

```text
requested_by
approved_by
status
timestamps
reason
```

---

# 34. Receiving

Receiving should be modeled separately from the purchase order.

## Table: goods_receipts

Fields:

```text
id UUID PK
organization_id FK
purchase_order_id FK
warehouse_id FK
receipt_number
status
received_at
received_by FK
created_at
updated_at
```

---

# 35. Goods Receipt Items

## Table: goods_receipt_items

Fields:

```text
id UUID PK
goods_receipt_id FK
purchase_order_item_id FK
product_id FK
batch_id FK nullable
received_quantity
unit_cost
created_at
```

Receiving transaction:

```text
Receipt
 ↓
Validate PO
 ↓
Validate warehouse
 ↓
Update inventory
 ↓
Update cost
 ↓
Create movement
 ↓
Audit
```

All critical changes occur transactionally.

---

# 36. Customers

## Table: customers

Fields:

```text
id UUID PK
organization_id FK
customer_code
name
email nullable
phone nullable
address nullable
tax_identifier nullable
is_active
created_at
updated_at
```

Constraint:

```text
UNIQUE(organization_id, customer_code)
```

---

# 37. Sales Orders

## Table: sales_orders

Fields:

```text
id UUID PK
organization_id FK
customer_id FK nullable
warehouse_id FK
sales_order_number
status
order_date
subtotal
tax_amount
total_amount
notes nullable
created_by FK
created_at
updated_at
```

Suggested statuses:

```text
DRAFT
CONFIRMED
COMPLETED
CANCELLED
```

Exact partial fulfillment states depend on DEC-015.

---

# 38. Sales Order Items

## Table: sales_order_items

Fields:

```text
id UUID PK
sales_order_id FK
product_id FK
unit_id FK
ordered_quantity
unit_price
tax_rate nullable
line_subtotal
line_tax
line_total
fulfilled_quantity
created_at
updated_at
```

Constraints:

```text
ordered_quantity > 0
unit_price >= 0
fulfilled_quantity >= 0
```

The final relationship between ordered and fulfilled quantities depends on DEC-015.

---

# 39. Sales Reservation

Reservations must be explicitly persisted because confirmed sales reserve stock.

## Table: inventory_reservations

Fields:

```text
id UUID PK
organization_id FK
warehouse_id FK
product_id FK
batch_id FK nullable
sales_order_id FK nullable
transfer_id FK nullable
quantity
status
created_at
released_at nullable
```

Suggested statuses:

```text
ACTIVE
RELEASED
CONSUMED
CANCELLED
```

---

# 40. Reservation Rule

Reservations increase:

```text
reserved_quantity
```

but do not immediately reduce:

```text
physical_quantity
```

The invariant remains:

```text
available = sellable - reserved
```

---

# 41. Reservation Sources

Confirmed rules:

### Sales

Confirmed sales may create reservations.

### Draft sales

Draft sales do not reserve stock.

### Transfers

Approved transfers may reserve stock where required by the workflow.

### Purchase orders

Purchase orders do not reserve inventory.

---

# 42. Sale Completion

At completion:

```text
reserved_quantity decreases
sellable_quantity decreases
physical stock decreases
inventory movement created
reservation marked consumed
audit event created
```

All required changes must be coordinated transactionally.

---

# 43. Sales Returns

## Table: sales_returns

Fields:

```text
id UUID PK
organization_id FK
sales_order_id FK
warehouse_id FK
return_number
status
reason nullable
created_by FK
inspected_by FK nullable
created_at
updated_at
```

Suggested statuses:

```text
DRAFT
SUBMITTED
INSPECTED
APPROVED
COMPLETED
REJECTED
```

Exact workflow may be refined.

---

# 44. Sales Return Items

## Table: sales_return_items

Fields:

```text
id UUID PK
sales_return_id FK
sales_order_item_id FK
product_id FK
quantity
inspection_result
batch_id FK nullable
created_at
```

Inspection result:

```text
SELLABLE
DAMAGED
EXPIRED
REJECTED
```

The resulting inventory classification determines the destination stock category.

---

# 45. Purchase Returns

## Table: purchase_returns

Fields:

```text
id UUID PK
organization_id FK
purchase_order_id FK
warehouse_id FK
return_number
status
reason nullable
created_by FK
created_at
updated_at
```

Purchase-return stock reduction occurs at the confirmed/dispatched workflow, not at draft creation.

---

# 46. Purchase Return Items

## Table: purchase_return_items

Fields:

```text
id UUID PK
purchase_return_id FK
purchase_order_item_id FK nullable
product_id FK
batch_id FK nullable
quantity
unit_cost
created_at
```

---

# 47. Warehouse Transfers

## Table: transfers

Fields:

```text
id UUID PK
organization_id FK
source_warehouse_id FK
destination_warehouse_id FK
transfer_number
status
requested_by FK
approved_by FK nullable
approved_at nullable
completed_at nullable
created_at
updated_at
```

Suggested statuses:

```text
DRAFT
SUBMITTED
APPROVED
IN_TRANSIT
COMPLETED
CANCELLED
```

Transfers require manager/admin approval.

---

# 48. Transfer Items

## Table: transfer_items

Fields:

```text
id UUID PK
transfer_id FK
product_id FK
batch_id FK nullable
quantity
created_at
updated_at
```

Constraints:

```text
quantity > 0
```

---

# 49. Transfer Inventory Integrity

A completed transfer should produce:

```text
TRANSFER_OUT
```

from source and:

```text
TRANSFER_IN
```

to destination.

These must be coordinated so the system cannot accidentally:

- remove stock twice
- add stock twice
- lose stock
- create stock from nothing

---

# 50. Approval Model

## Table: approvals

General-purpose approval record.

Fields:

```text
id UUID PK
organization_id FK
action_type
resource_type
resource_id
requested_by FK
approved_by FK nullable
status
reason nullable
requested_at
approved_at nullable
rejected_at nullable
expires_at nullable
created_at
updated_at
```

Statuses:

```text
PENDING
APPROVED
REJECTED
EXPIRED
CANCELLED
```

---

# 51. Approval Resource References

A polymorphic reference:

```text
resource_type
resource_id
```

is useful for:

- purchase orders
- transfers
- AI actions
- special sales overrides

However, critical relationships should still be validated by the application service.

---

# 52. Audit Events

## Table: audit_events

Fields:

```text
id UUID PK
organization_id FK nullable
actor_user_id FK nullable
action
entity_type
entity_id
request_id nullable
metadata JSONB
created_at
```

Potential actions:

```text
LOGIN
LOGOUT
CREATE
UPDATE
DELETE
APPROVE
REJECT
STOCK_ADJUSTED
SALE_CONFIRMED
SALE_COMPLETED
PURCHASE_APPROVED
TRANSFER_APPROVED
AI_TOOL_CALLED
AI_ACTION_CREATED
AI_ACTION_APPROVED
AI_ACTION_EXECUTED
```

The exact taxonomy can evolve.

---

# 53. Audit Metadata

`metadata JSONB` may contain contextual information such as:

```json
{
  "warehouse_id": "...",
  "quantity": 10,
  "reason": "Damaged stock"
}
```

Do not store:

- passwords
- raw authentication tokens
- secret keys
- sensitive credentials

---

# 54. Notifications

## Table: notifications

Fields:

```text
id UUID PK
organization_id FK
user_id FK
notification_type
title
message
entity_type nullable
entity_id nullable
is_read
created_at
read_at nullable
```

Indexes should support:

```text
(user_id, is_read, created_at)
```

---

# 55. Report Jobs

Large reports/exports should be asynchronous.

## Table: report_jobs

Fields:

```text
id UUID PK
organization_id FK
requested_by FK
report_type
parameters JSONB
status
file_object_key nullable
error_message nullable
created_at
completed_at nullable
```

Suggested statuses:

```text
QUEUED
PROCESSING
COMPLETED
FAILED
EXPIRED
```

---

# 56. Object Storage Metadata

## Table: stored_files

Fields:

```text
id UUID PK
organization_id FK
uploaded_by FK
object_key
original_filename
content_type
size_bytes
purpose
status
created_at
```

Object contents live in object storage.

PostgreSQL stores metadata.

---

# 57. AI Conversation Model

## Table: ai_conversations

Fields:

```text
id UUID PK
organization_id FK
user_id FK
title nullable
status
created_at
updated_at
```

AI conversation data is tenant-scoped.

---

# 58. AI Messages

## Table: ai_messages

Fields:

```text
id UUID PK
conversation_id FK
role
content
created_at
```

Possible roles:

```text
USER
ASSISTANT
SYSTEM_REFERENCE
TOOL
```

Sensitive provider internals should not be stored unnecessarily.

---

# 59. AI Tool Executions

## Table: ai_tool_executions

Fields:

```text
id UUID PK
organization_id FK
conversation_id FK nullable
user_id FK
tool_name
tool_version nullable
input_json JSONB
output_json JSONB nullable
status
started_at
completed_at nullable
error_code nullable
created_at
```

This provides traceability for AI tool usage.

Sensitive data should be minimized.

---

# 60. AI Actions

## Table: ai_actions

Represents a proposed AI-assisted mutation.

Fields:

```text
id UUID PK
organization_id FK
requested_by FK
conversation_id FK nullable
action_type
resource_type
resource_id nullable
proposal_json JSONB
status
created_at
updated_at
approved_by FK nullable
approved_at nullable
executed_at nullable
rejection_reason nullable
```

Suggested statuses:

```text
DRAFT
PENDING_APPROVAL
APPROVED
REJECTED
STALE
EXECUTED
FAILED
CANCELLED
```

---

# 61. AI Action Rule

AI actions do not directly mutate inventory.

Instead:

```text
AI Action
   ↓
Approval
   ↓
Revalidation
   ↓
Application Service
   ↓
Database Transaction
```

The AI action record is a workflow object, not an alternate mutation pathway.

---

# 62. AI Recommendation Records

If recommendations need persistence independent of actions:

## Table: ai_recommendations

Fields:

```text
id UUID PK
organization_id FK
user_id FK nullable
recommendation_type
subject_type
subject_id
recommendation_json JSONB
confidence nullable
data_as_of
status
created_at
expires_at nullable
```

This supports:

- reorder recommendations
- supplier recommendations
- anomaly recommendations
- demand recommendations

---

# 63. AI Freshness

AI-derived records involving operational data should include:

```text
data_as_of
```

This allows the UI and AI layer to distinguish:

```text
current data
```

from:

```text
previously generated recommendation
```

---

# 64. AI Confidence

Where applicable:

```text
confidence numeric
```

should be treated as informational metadata, not as proof of correctness.

Confidence should not replace business validation.

---

# 65. AI Prompt / Provider Metadata

Avoid storing full provider payloads unless operationally necessary.

If required:

## Table: ai_runs

```text
id UUID PK
organization_id FK
user_id FK nullable
provider
model
request_type
latency_ms
input_tokens nullable
output_tokens nullable
status
created_at
```

Do not store secrets.

---

# 66. Product and Inventory Relationships

Conceptual relationship:

```text
Organization
    │
    ├── Product
    │     │
    │     ├── Inventory Position
    │     │       └── Warehouse
    │     │
    │     └── Inventory Batch
    │
    └── Warehouse
```

A product can exist in multiple warehouses.

A warehouse can contain multiple products.

Therefore:

```text
Product N ↔ N Warehouse
```

is resolved through:

```text
inventory_positions
```

---

# 67. Core Relationship Map

```text
Organization
│
├── Memberships
│   └── Users
│       └── Warehouse Assignments
│
├── Products
│   ├── Category
│   ├── Brand
│   └── Unit
│
├── Warehouses
│
├── Inventory Positions
│   ├── Product
│   ├── Warehouse
│   └── Batches
│
├── Suppliers
│   └── Purchase Orders
│       └── Purchase Order Items
│           └── Goods Receipts
│
├── Customers
│   └── Sales Orders
│       └── Sales Order Items
│           └── Reservations
│
├── Returns
│
├── Transfers
│
├── Approvals
│
├── Notifications
│
├── Audit Events
│
└── AI
    ├── Conversations
    ├── Messages
    ├── Tool Executions
    ├── Recommendations
    └── Actions
```

---

# 68. Organization-Scoped Uniqueness

The system should generally use:

```text
UNIQUE(organization_id, business_code)
```

rather than global uniqueness for tenant-owned identifiers.

Examples:

```text
UNIQUE(organization_id, sku)
UNIQUE(organization_id, supplier_code)
UNIQUE(organization_id, customer_code)
UNIQUE(organization_id, warehouse_code)
```

This permits independent organizations to use the same internal codes.

---

# 69. Cross-Entity Organization Integrity

A major integrity risk is:

```text
Organization A
    Product A

Organization B
    Warehouse B

Inventory row accidentally references:
    Product A + Warehouse B
```

Application services must prevent this.

Where practical, composite foreign-key patterns or database-level structures can strengthen this boundary.

At minimum:

```text
inventory.organization_id
must match
product.organization_id
and warehouse.organization_id
```

This is a critical multi-tenant invariant.

---

# 70. Soft Deletion

Transactional entities should generally not be physically deleted once referenced.

Prefer lifecycle/status fields:

```text
is_active
status
archived
```

This is especially important for:

- products
- warehouses
- suppliers
- customers
- users
- historical transaction entities

Historical records must remain queryable.

---

# 71. Product Deactivation

A product marked INACTIVE:

- remains in database
- remains in historical transactions
- remains in reports
- may remain in inventory
- cannot be selected for new sales
- cannot be selected for new purchases

This is primarily an application/domain rule.

---

# 72. Warehouse Deactivation

A warehouse should not be physically deleted if it has historical inventory or transactions.

Use:

```text
is_active = false
```

and preserve history.

The application should prevent new operations against inactive warehouses where appropriate.

---

# 73. Supplier Deactivation

Suppliers should remain available in historical purchase records.

Use:

```text
is_active = false
```

rather than destructive deletion.

---

# 74. Customer Deactivation

Customers should remain available for historical sales.

Use:

```text
is_active = false
```

where applicable.

---

# 75. Foreign Key Deletion Policy

Default approach:

```text
PROTECT / RESTRICT
```

for core transactional relationships.

Avoid cascading deletion from:

```text
organization
product
warehouse
supplier
customer
```

into historical transactions.

Cascade deletion should be limited to safe child records whose lifecycle is fully owned by the parent.

---

# 76. Index Strategy

Important indexes should support:

### Tenant filtering

```text
organization_id
```

### Inventory

```text
(organization_id, warehouse_id, product_id)
(organization_id, product_id)
```

### Movement history

```text
(organization_id, product_id, occurred_at)
(organization_id, warehouse_id, occurred_at)
```

### Expiry

```text
(organization_id, expiry_date)
```

### Sales

```text
(organization_id, warehouse_id, order_date)
```

### Purchasing

```text
(organization_id, supplier_id, order_date)
```

### Notifications

```text
(user_id, is_read, created_at)
```

### Audit

```text
(organization_id, created_at)
(organization_id, entity_type, entity_id)
```

Exact indexes should be validated using real query plans.

---

# 77. Partial Indexes

PostgreSQL partial indexes may improve common operational queries.

Examples:

```text
active products
pending approvals
unread notifications
active reservations
```

Use them only where query patterns justify them.

---

# 78. JSONB Usage

JSONB is appropriate for flexible metadata such as:

- audit metadata
- AI tool input/output
- AI proposal details
- report parameters

Do not use JSONB as a substitute for core relational modeling.

For example, avoid storing:

```text
product
warehouse
quantity
```

inside an opaque JSON document when these are core queryable business entities.

---

# 79. Money Representation

Use PostgreSQL `numeric` for monetary values.

Avoid floating-point types for money.

Recommended conceptual precision:

```text
NUMERIC(18, 4)
```

or another centrally defined precision.

The exact precision must be finalized during migration design.

---

# 80. Quantity Representation

Inventory quantities may be fractional depending on unit.

Use PostgreSQL `numeric`.

Avoid floating-point types for authoritative inventory quantities.

A centrally defined precision/scale should be chosen based on supported units.

---

# 81. Tax Representation

The initial system supports basic tax fields.

Likely fields:

```text
tax_rate
tax_amount
```

The database should not prematurely implement a full jurisdictional tax engine.

---

# 82. Sales and Purchase Totals

Where persisted:

```text
subtotal
tax_amount
total_amount
```

should be calculated by deterministic backend services.

The frontend may preview totals, but backend calculation is authoritative.

---

# 83. Derived vs Stored Data

Prefer deriving values that can safely be calculated.

Examples:

```text
available_quantity
```

should be derived from:

```text
sellable_quantity - reserved_quantity
```

Persist values when:

- calculation is expensive
- historical snapshot is required
- performance requires it
- external business semantics require it

Any duplicated value requires a consistency strategy.

---

# 84. Historical Snapshots

Transactions may store historical values that should not change if master data later changes.

Examples:

Purchase order item:

```text
unit_cost
tax_rate
```

Sales order item:

```text
unit_price
tax_rate
```

This prevents changing a product's current price from rewriting historical transactions.

---

# 85. Audit Immutability

Audit events should be append-oriented.

Normal users should not be able to modify or delete audit history.

Administrative retention/deletion policies remain dependent on DEC-011 and future governance.

---

# 86. Inventory Movement Immutability

Inventory movements should be treated as historical facts.

Corrections should normally be represented by new movements rather than editing history.

Example:

Incorrect adjustment:

```text
ADJUSTMENT_OUT 10
```

Correction:

```text
ADJUSTMENT_IN 10
```

with a reason/reference.

This creates a traceable ledger.

---

# 87. Reservation Integrity

The application must guarantee:

```text
reserved_quantity <= sellable_quantity
```

and:

```text
reserved_quantity >= 0
```

Reservation release/consumption must be idempotent.

A reservation must not be consumed twice.

---

# 88. Batch Integrity

Batch records must belong to:

```text
organization
product
warehouse
```

The application must prevent cross-organization batch references.

Batch quantities must remain consistent with their parent inventory position.

---

# 89. Expiry Processing

Expiry can be processed:

- at transaction time when detected
- by scheduled background jobs
- by explicit user action

A scheduled job may identify expired batches, but any inventory mutation must still pass through the inventory service and create the required movement/audit trail.

---

# 90. Reorder Point

Products contain a user-maintained reorder point.

Conceptual field:

```text
reorder_point
```

AI may recommend a change.

AI must not silently overwrite the value.

A future recommendation may be represented in:

```text
ai_recommendations
```

and accepted through a normal controlled workflow.

---

# 91. AI Recommendation Acceptance

Acceptance flow:

```text
AI Recommendation
      ↓
User accepts
      ↓
Create Draft / Proposed Change
      ↓
Normal validation
      ↓
Required approval if applicable
      ↓
Execution
```

The recommendation itself does not automatically mutate master data.

---

# 92. Reports and Data Warehouse Consideration

Initial architecture uses PostgreSQL operational queries.

Do not introduce a separate data warehouse initially unless reporting scale requires it.

Future evolution could include:

```text
PostgreSQL
   ↓
CDC / ETL
   ↓
Analytics Warehouse
```

without changing the transactional domain model.

---

# 93. Database Connection Management

Production Django should use appropriate PostgreSQL connection management/pooling.

Connection limits must be considered alongside:

- web workers
- Celery workers
- background jobs
- administrative processes

Do not allow worker multiplication to exhaust PostgreSQL connections.

---

# 94. Row-Level Locking

Critical inventory operations should use PostgreSQL row-level locks through Django transaction mechanisms.

Conceptually:

```text
transaction.atomic()
+
select_for_update()
```

Lock only the necessary inventory rows.

Avoid broad table locks.

---

# 95. Deadlock Avoidance

When multiple inventory rows must be locked:

- acquire locks in a deterministic order
- keep transactions short
- avoid network calls inside the transaction
- avoid LLM calls inside inventory transactions
- avoid sending emails inside critical transactions

This is especially important for:

- transfers
- multi-item sales
- bulk adjustments

---

# 96. AI and Transactions

Never keep a database transaction open while waiting for an LLM.

Bad:

```text
BEGIN
 ↓
call LLM
 ↓
wait 5 seconds
 ↓
update inventory
 ↓
COMMIT
```

Correct:

```text
LLM recommendation
 ↓
user approval
 ↓
fresh transaction
 ↓
lock
 ↓
validate
 ↓
mutate
 ↓
commit
```

---

# 97. Celery and Transactions

Background jobs that mutate business data must enter normal application services.

Example:

```text
Celery
 ↓
ExpiryService
 ↓
InventoryService
 ↓
transaction
```

Celery must not directly modify inventory rows.

---

# 98. Data Integrity Priority

When performance conflicts with correctness:

> Prefer correctness for inventory and transactional state.

Caching and asynchronous optimization must never create an authoritative stale stock decision.

---

# 99. Backup Considerations

Backup scope must include:

- PostgreSQL database
- migration history
- object-storage metadata
- required object-storage files

Redis generally does not contain authoritative business data and therefore is not the primary recovery source.

---

# 100. Migration Strategy

All schema changes must use Django migrations.

Migration rules:

- review generated SQL
- test migrations
- avoid destructive migrations without a plan
- separate data migrations from schema changes when practical
- use backward-compatible migrations for zero/minimal downtime changes where required

---

# 101. Initial Entity Inventory

The initial logical model contains these major entities:

### Identity / tenant

```text
organizations
users
organization_memberships
membership_warehouses
```

### Catalog

```text
products
categories
brands
units
unit_conversions
```

### Warehouses / inventory

```text
warehouses
inventory_positions
inventory_batches
inventory_batch_positions
inventory_movements
inventory_adjustments
inventory_cost_positions
inventory_reservations
```

### Purchasing

```text
suppliers
purchase_orders
purchase_order_items
goods_receipts
goods_receipt_items
purchase_returns
purchase_return_items
```

### Sales

```text
customers
sales_orders
sales_order_items
sales_returns
sales_return_items
```

### Transfers

```text
transfers
transfer_items
```

### Platform

```text
approvals
notifications
audit_events
stored_files
report_jobs
```

### AI

```text
ai_conversations
ai_messages
ai_tool_executions
ai_actions
ai_recommendations
ai_runs
```

---

# 102. Key Relationship Summary

```text
Organization
│
├── Membership ── User
│      │
│      └── Membership Warehouse ── Warehouse
│
├── Product ── Category
│      │     └── Brand
│      │
│      ├── Inventory Position ── Warehouse
│      │
│      └── Inventory Batch
│
├── Supplier ── Purchase Order
│                    │
│                    ├── Purchase Order Item ── Product
│                    │
│                    └── Goods Receipt
│
├── Customer ── Sales Order
│                   │
│                   ├── Sales Order Item ── Product
│                   │
│                   └── Reservation
│
├── Sales Return
├── Purchase Return
│
├── Transfer
│     └── Transfer Item
│
├── Approval
├── Notification
├── Audit Event
│
└── AI
      ├── Conversation
      │     └── Message
      ├── Tool Execution
      ├── Recommendation
      ├── Action
      └── Run
```

---

# 103. Core Database Invariants

The following must always hold.

## Inventory

```text
available = sellable - reserved
```

```text
physical = sellable + damaged + expired
```

```text
sellable = available + reserved
```

## Quantities

```text
all quantities >= 0
```

## Reservation

```text
reserved <= sellable
```

## Tenant isolation

```text
entity.organization_id
must match the authenticated organization context
```

## Product lifecycle

```text
inactive products cannot enter new sales/purchases
```

## AI

```text
AI does not directly mutate transactional data
```

## Approval

```text
high-impact AI write
→ human approval
→ revalidation
→ normal service execution
```

---

# 104. Unresolved Business Decisions

The following remain intentionally unresolved and must not be silently encoded as final business policy:

## DEC-001

Commercial pricing model.

## DEC-002

Organization scale.

## DEC-011

Data retention.

## DEC-012

SUPER_ADMIN tenant operational data access.

## DEC-013

Whether users can belong to multiple organizations.

## DEC-015

Partial sales fulfillment.

## DEC-016

Invoicing depth.

## DEC-017

Payment recording depth.

The database design is structured to minimize redesign when these decisions are finalized.

---

# 105. Schema Decisions Dependent on Open Decisions

| Decision | Database Impact |
|---|---|
| DEC-001 | commercial/billing tables if SaaS billing is introduced |
| DEC-002 | partitioning/scaling strategy at larger scale |
| DEC-011 | retention/archival/deletion policies |
| DEC-012 | SUPER_ADMIN authorization relationships |
| DEC-013 | organization membership semantics |
| DEC-015 | sales fulfillment/backorder/partial shipment state |
| DEC-016 | invoice tables and document lifecycle |
| DEC-017 | payment tables and payment status |

---

# 106. What This Design Does Not Yet Define

The following belong to subsequent artifacts:

- exact SQL DDL
- exact Django model code
- exact API payload schemas
- endpoint methods
- pagination contracts
- UI layouts
- detailed AI tool schemas
- final prompt templates
- detailed notification channels
- cloud-specific infrastructure
- production sizing numbers

---

# 107. Database Design Validation Checklist

Before implementing Django models:

- [ ] every tenant entity has organization ownership
- [ ] cross-tenant references are prevented
- [ ] all critical foreign keys are defined
- [ ] unique constraints are reviewed
- [ ] quantity constraints are reviewed
- [ ] money uses numeric
- [ ] timestamps use timezone-aware types
- [ ] inventory available quantity remains derived
- [ ] reservation rules are represented
- [ ] inventory movements are append-oriented
- [ ] weighted-average-cost data is sufficient
- [ ] batch/expiry is supported
- [ ] historical transaction values are snapshotted
- [ ] audit records are append-oriented
- [ ] AI records are tenant-scoped
- [ ] approval records support revalidation
- [ ] indexes match major access patterns
- [ ] open business decisions are not accidentally finalized

---

# 108. Next Artifact

The next specification should be:

**API-SPECIFICATION.md**

It should define:

- API conventions
- authentication endpoints
- organization endpoints
- user/membership endpoints
- catalog endpoints
- warehouse endpoints
- inventory endpoints
- inventory adjustment endpoints
- supplier endpoints
- purchase order endpoints
- receiving endpoints
- customer endpoints
- sales endpoints
- return endpoints
- transfer endpoints
- report endpoints
- notification endpoints
- approval endpoints
- AI Copilot endpoints
- AI tool execution contracts
- AI action/approval contracts
- pagination
- filtering
- sorting
- error codes
- HTTP status conventions
- idempotency
- authorization behavior
- tenant isolation
- request/response schemas

---

# 109. Final Database Principle

> **Model business facts relationally, protect invariants transactionally, preserve historical truth, and never create a second source of truth merely for convenience.**

The database must support the application architecture rather than becoming an uncontrolled collection of tables.

