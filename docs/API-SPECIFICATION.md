# Inventory Management AI — API Specification

**Version:** 1.0 draft for implementation planning  
**Status:** Proposed technical contract; unresolved business decisions remain open  
**Base URL:** `/api/v1`  
**Authority:** Confirmed Requirement Decisions → Business Rules → FRD → Architecture → Database Design → this document.

## 1. Contract conventions

- REST over HTTPS; JSON request and response bodies, UTF-8; UUID identifiers; ISO 8601 timestamps with timezone.
- Browser authentication: Django-managed secure HttpOnly cookie/session; CSRF protection on unsafe requests. Never put authentication secrets in localStorage.
- All tenant endpoints derive the active organization from authenticated server-side context, **not** a supplied `organization_id`. The organization-selection mechanism is contingent on DEC-013.
- Roles: `SUPER_ADMIN`, `ADMIN`, `MANAGER`, `INVENTORY_STAFF`, `SALES_STAFF`, `VIEWER`. Warehouse-restricted roles see assigned warehouses only. `SUPER_ADMIN` tenant operational access remains subject to DEC-012.
- IDs are UUIDs; money and quantities are **decimal strings** in JSON, never floating-point approximations. Currency uses the organization's base currency.
- List endpoints: `?page=1&page_size=25&search=...&ordering=-created_at`; default page size 25, maximum 100. Unsupported filters/sorts return 400. Every list is permission-filtered before pagination and counts.
- List response: `{ "count": 0, "next": null, "previous": null, "results": [] }`. Detail response: a resource object. Mutation response: resource object or an explicit operation result.
- Dates use `YYYY-MM-DD`; datetimes use RFC 3339 with offset. Use stable enums, documented per resource. Unknown enum values return 400.
- Sensitive cost fields are omitted from responses to roles without cost access, including exports and AI tool responses.
- Destructive removal of referenced transactional data is prohibited; use documented status transitions or deactivation.
- API is versioned. Breaking contract changes require versioning or a controlled migration.

### 1.1 Error envelope

```json
{
  "error": {
    "code": "INSUFFICIENT_AVAILABLE_STOCK",
    "message": "Requested quantity exceeds available stock.",
    "details": {"product_id": "<uuid>", "field": "quantity"},
    "request_id": "<correlation-id>"
  }
}
```

| Status | Meaning |
|---|---|
| 200 | Read or successful state transition |
| 201 | Resource created |
| 202 | Asynchronous operation accepted |
| 204 | Successful no-body action |
| 400 | Invalid syntax, input, or business validation |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Not found **or concealed by tenant scope** |
| 409 | Invalid state transition, duplicate business operation, stale state, or concurrent conflict |
| 412 | Optional precondition/version check failed |
| 413 | Upload exceeds permitted size |
| 415 | Unsupported content type |
| 422 | Semantically invalid structured AI/tool input, where appropriate |
| 429 | Rate limited |
| 500 | Unexpected server failure; no sensitive trace exposed |
| 503 | Required dependency temporarily unavailable |

Stable codes include `VALIDATION_ERROR`, `UNAUTHENTICATED`, `FORBIDDEN`, `RESOURCE_NOT_FOUND`, `DUPLICATE_CODE`, `PRODUCT_INACTIVE`, `WAREHOUSE_INACTIVE`, `UNAUTHORIZED_WAREHOUSE`, `INSUFFICIENT_AVAILABLE_STOCK`, `INVALID_BATCH`, `EXPIRED_BATCH`, `INVALID_STATE_TRANSITION`, `APPROVAL_REQUIRED`, `STALE_APPROVAL`, `IDEMPOTENCY_CONFLICT`, `CONCURRENT_UPDATE_CONFLICT`, `AI_TOOL_NOT_ALLOWED`, `AI_PROVIDER_UNAVAILABLE`.

### 1.2 Idempotency and concurrency

- Require `Idempotency-Key` on stock-changing execution operations (receiving confirmation, sales completion, transfer completion, adjustment posting, return posting, AI action execution). Scope keys to organization + authenticated actor + endpoint; retain results for a documented operational period, not a business-data retention promise.
- Same key + same normalized payload returns original outcome; same key + different payload returns `409 IDEMPOTENCY_CONFLICT`.
- Services use `transaction.atomic()` and deterministic `select_for_update()` lock order for affected inventory positions/batches. Re-read available stock inside the transaction.
- Optionally support `If-Match`/resource version on editable drafts and approval screens. Return 412 on mismatched version when enabled.
- Never call an LLM or external notification provider inside a stock transaction. Dispatch side effects after commit.

## 2. Authentication and organization context

| Method | Path | Purpose | Access |
|---|---|---|---|
| POST | `/auth/login` | Establish secure cookie session | Public, rate-limited |
| POST | `/auth/logout` | End session | Authenticated |
| GET | `/auth/me` | Current identity, active membership, capabilities | Authenticated |
| GET | `/organizations` | Organizations user can access | Authenticated; DEC-013 affects membership |
| POST | `/organizations` | Create organization and initial ADMIN | Qualified authenticated user, per confirmed rule |
| GET | `/organizations/current` | Current organization configuration | Active member |
| PATCH | `/organizations/current` | Update allowed settings | ADMIN |
| GET | `/memberships` | List scoped members | ADMIN/MANAGER per permission matrix |
| POST | `/memberships` | Invite/create membership | ADMIN |
| PATCH | `/memberships/{id}` | Change role/activation | ADMIN; guard privilege escalation |
| GET | `/memberships/{id}/warehouses` | Assigned warehouse access | Authorized admin or self |
| PUT | `/memberships/{id}/warehouses` | Replace assigned warehouses | ADMIN |

**Login request:** `{"email":"user@example.com","password":"..."}`. Return safe user metadata; authentication state is established by cookie. CSRF token retrieval/rotation must follow the chosen Django integration and be documented during implementation. Never return a reusable password or authentication token in a response body.

**Organization creation:** `{"name":"Acme","code":"ACME","base_currency":"INR"}`. Creation assigns initial ADMIN to the qualified creator. Changing base currency after transactions exist needs an explicit guarded policy; do not silently convert historical amounts.

## 3. Catalog and warehouses

| Method | Path | Purpose | Typical permission |
|---|---|---|---|
| GET, POST | `/products` | List/create products | Authorized read / ADMIN or MANAGER create |
| GET, PATCH | `/products/{id}` | Detail/edit | Authorized read / authorized catalog editor |
| POST | `/products/{id}/activate` | Activate valid product | Catalog editor |
| POST | `/products/{id}/deactivate` | Prevent new purchase/sale use | Catalog editor |
| GET, POST | `/categories` | List/create categories | Authorized read / catalog editor |
| GET, PATCH | `/categories/{id}` | Detail/edit | Authorized read / catalog editor |
| GET, POST | `/brands` | List/create brands | Authorized read / catalog editor |
| GET, PATCH | `/brands/{id}` | Detail/edit | Authorized read / catalog editor |
| GET, POST | `/units` | List/create units | Authorized read / ADMIN/MANAGER |
| GET, POST | `/unit-conversions` | Explicit unit conversions | Authorized read / ADMIN/MANAGER |
| GET, POST | `/warehouses` | Scoped list/create | Assigned or all as authorized / ADMIN/MANAGER |
| GET, PATCH | `/warehouses/{id}` | Detail/edit | Scoped read / ADMIN/MANAGER |
| POST | `/warehouses/{id}/deactivate` | Deactivate safely | ADMIN/MANAGER |

**Create product example:**

```json
{
  "sku":"ITEM-001", "name":"Widget", "unit_id":"<uuid>",
  "category_id":null, "brand_id":null,
  "reorder_point":"10", "reorder_quantity":"20",
  "default_cost":"12.50", "default_sale_price":"18.00", "tax_rate":"0"
}
```

Product `status` follows `DRAFT → ACTIVE → INACTIVE → ARCHIVED` with valid controlled transitions; do not allow arbitrary status PATCH. `sku` is unique within organization. Each separately tracked item is a separate SKU; no variant matrix. Conversion requests include `from_unit_id`, `to_unit_id`, and positive decimal `conversion_factor`; prevent ambiguous/conflicting conversion definitions. Warehouse queries and responses always honor assigned-warehouse scope.

## 4. Inventory and stock operations

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET | `/inventory` | Scoped warehouse/product inventory positions | Authorized warehouse reader |
| GET | `/inventory/{position_id}` | Inventory position detail | Authorized warehouse reader |
| GET | `/inventory/movements` | Immutable movement history | Authorized warehouse reader |
| GET | `/inventory/batches` | Batch/lot and expiry status | Authorized warehouse reader |
| GET | `/inventory/low-stock` | User-maintained reorder threshold comparison | Authorized warehouse reader |
| GET, POST | `/inventory/adjustments` | List/create adjustment drafts | Authorized warehouse reader / authorized inventory writer |
| GET, PATCH | `/inventory/adjustments/{id}` | Inspect/edit draft | Scoped authorized actor |
| POST | `/inventory/adjustments/{id}/post` | Validate and atomically apply adjustment | Authorized inventory writer; approval if applicable |
| POST | `/inventory/opening-stock/imports` | Upload/prepare opening stock | Authorized inventory writer |
| GET | `/inventory/opening-stock/imports/{id}` | Import validation/result | Scoped authorized actor |
| POST | `/inventory/opening-stock/imports/{id}/commit` | Commit validated opening stock | Authorized inventory writer |

**Inventory position response:**

```json
{
  "id":"<uuid>", "product_id":"<uuid>", "warehouse_id":"<uuid>",
  "sellable_quantity":"100", "reserved_quantity":"20",
  "available_quantity":"80", "damaged_quantity":"2", "expired_quantity":"3",
  "physical_quantity":"105", "updated_at":"2026-10-05T00:00:00Z"
}
```

`available_quantity = sellable_quantity - reserved_quantity`; `physical_quantity = sellable_quantity + damaged_quantity + expired_quantity`. Both derived values are read-only. All quantities are nonnegative and reserved ≤ sellable. Inventory position/batch totals must reconcile. Stock mutation requests identify product, warehouse, optional batch, quantity, and business reason/reference. Never expose an endpoint to PATCH inventory balances directly.

**Adjustment draft:** `{"warehouse_id":"<uuid>","product_id":"<uuid>","batch_id":null,"adjustment_type":"OUT","quantity":"2","reason":"Count correction"}`. Posting locks relevant rows, validates scope/stock/batch, updates balances, writes a recognized movement, and writes an audit event atomically.

## 5. Suppliers, purchasing and receiving

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET, POST | `/suppliers` | List/create suppliers | Authorized read / purchasing editor |
| GET, PATCH | `/suppliers/{id}` | Detail/edit | Authorized read / purchasing editor |
| GET, POST | `/purchase-orders` | List/create draft POs | Authorized read / purchasing editor |
| GET, PATCH | `/purchase-orders/{id}` | Detail/edit draft | Authorized actor |
| POST | `/purchase-orders/{id}/submit` | Submit for approval | Authorized actor |
| POST | `/purchase-orders/{id}/approve` | Approve submitted PO | ADMIN/MANAGER |
| POST | `/purchase-orders/{id}/reject` | Reject with reason | ADMIN/MANAGER |
| POST | `/purchase-orders/{id}/cancel` | Controlled cancellation | Authorized actor; state-dependent |
| GET, POST | `/goods-receipts` | List/create receipt draft for approved PO | Authorized read / receiving staff |
| GET, PATCH | `/goods-receipts/{id}` | Inspect/edit draft | Authorized receiving staff |
| POST | `/goods-receipts/{id}/post` | Atomic receiving and cost update | Authorized receiving staff |

**Create PO:**

```json
{
  "supplier_id":"<uuid>","warehouse_id":"<uuid>","expected_date":null,
  "items":[{"product_id":"<uuid>","unit_id":"<uuid>","ordered_quantity":"10","unit_cost":"25.00","tax_rate":"0"}],
  "notes":"Regular replenishment"
}
```

PO status includes `DRAFT`, `SUBMITTED`, `APPROVED`, `PARTIALLY_RECEIVED`, `RECEIVED`, `CANCELLED`. Submitted POs require ADMIN/MANAGER approval. POs do not reserve stock. All monetary totals are recalculated on the backend. A receipt references an approved PO and contains receipt items with `purchase_order_item_id`, `received_quantity`, optional batch/expiry information and authoritative cost. Receipt posting is idempotent and writes inventory movements and Weighted Average Cost updates in one transaction. Over-receipt is rejected under the proposed initial constraint unless a later confirmed rule changes it.

## 6. Customers, sales and reservations

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET, POST | `/customers` | List/create customers | Authorized read / sales editor |
| GET, PATCH | `/customers/{id}` | Detail/edit | Authorized read / sales editor |
| GET, POST | `/sales-orders` | List/create draft sale | Authorized read / sales editor |
| GET, PATCH | `/sales-orders/{id}` | Detail/edit draft | Authorized sales actor |
| POST | `/sales-orders/{id}/confirm` | Validate and reserve available stock | Authorized sales actor |
| POST | `/sales-orders/{id}/complete` | Consume reservations, move stock | Authorized sales actor |
| POST | `/sales-orders/{id}/cancel` | Controlled cancellation/release | Authorized actor; unusual override ADMIN/MANAGER |
| GET | `/inventory/reservations` | Scoped reservation list | Authorized warehouse reader |

**Create sale:**

```json
{
  "customer_id":"<uuid>", "warehouse_id":"<uuid>",
  "items":[{"product_id":"<uuid>","unit_id":"<uuid>","ordered_quantity":"3","unit_price":"40.00","tax_rate":"0"}]
}
```

Drafts do not reserve stock. Confirmation locks inventory, checks fresh available stock and creates reservations; if two sales compete for the last available unit, only one may succeed. Completion atomically reduces sellable/physical stock and releases/consumes reservation, records `SALE` movement, and audits. Negative stock and backorders are prohibited. **DEC-015 partial fulfillment is unresolved**: do not add partial completion endpoints or promise partial fulfillment until decided. Invoicing and payment endpoints are deliberately excluded pending DEC-016 and DEC-017.

## 7. Returns and transfers

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET, POST | `/sales-returns` | List/create customer return draft | Scoped authorized actor |
| GET, PATCH | `/sales-returns/{id}` | Detail/edit draft | Scoped authorized actor |
| POST | `/sales-returns/{id}/inspect` | Record item inspection classifications | Authorized inventory staff |
| POST | `/sales-returns/{id}/complete` | Post classified stock movements | Authorized inventory staff |
| GET, POST | `/purchase-returns` | List/create purchase return draft | Scoped authorized actor |
| GET, PATCH | `/purchase-returns/{id}` | Detail/edit draft | Scoped authorized actor |
| POST | `/purchase-returns/{id}/dispatch` | Confirm/dispatched stock reduction | Authorized inventory staff |
| GET, POST | `/transfers` | List/create transfer draft | Scoped authorized actor |
| GET, PATCH | `/transfers/{id}` | Detail/edit draft | Scoped authorized actor |
| POST | `/transfers/{id}/submit` | Request approval | Authorized actor |
| POST | `/transfers/{id}/approve` | Approve | ADMIN/MANAGER |
| POST | `/transfers/{id}/reject` | Reject | ADMIN/MANAGER |
| POST | `/transfers/{id}/complete` | Execute transfer out/in atomically | Authorized inventory actor after approval |

Sales returns require inspection before any returned goods become sellable. Inspection outcomes: `SELLABLE`, `DAMAGED`, `EXPIRED`, `REJECTED`. Purchase return drafts do not change stock; dispatch/confirmation does. Transfers must use source and destination warehouses within the same organization; source authorization, approved status, available quantity, consistent row-lock ordering and dual movement creation are mandatory. Approved transfers may reserve stock per confirmed workflow; no double-consumption on completion.

## 8. Approvals, reports, notifications and audit

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET | `/approvals` | Scoped approval queue | Authorized approver/requester |
| GET | `/approvals/{id}` | Approval detail and current-state preview | Authorized actor |
| POST | `/approvals/{id}/approve` | Human decision; revalidate on execution | Authorized approver |
| POST | `/approvals/{id}/reject` | Reject with reason | Authorized approver |
| GET | `/reports/inventory` | Inventory report | Scoped authorized reader |
| GET | `/reports/movements` | Stock movement report | Scoped authorized reader |
| GET | `/reports/sales` | Sales report | Scoped authorized reader |
| GET | `/reports/purchasing` | Purchasing report | Scoped authorized reader |
| GET | `/reports/warehouse-distribution` | Warehouse inventory distribution | Scoped authorized reader |
| POST | `/report-jobs` | Queue large report/export | Authorized report reader |
| GET | `/report-jobs/{id}` | Job state and authorized download reference | Job owner/authorized manager |
| GET | `/notifications` | Current user's notifications | Authenticated |
| POST | `/notifications/{id}/read` | Mark notification read | Owner |
| GET | `/audit-events` | Scoped audit search | Explicit audit permission |

**Approval request:** `{"reason":"Reviewed current stock and supplier terms"}`; rejection requires a nonempty reason. Generic approval endpoints must not allow bypassing resource-specific business requirements. Every approval execution rechecks organization, warehouse, actor permission, target state and relevant inventory values. Prefer one canonical approval execution pathway per resource, avoiding duplicate execution via both generic and resource-specific routes.

Reports accept documented filters such as `warehouse_id`, `product_id`, `date_from`, `date_to`, `status`, subject to permission. Never expose hidden cost columns through export. Warehouse distribution is not warehouse-capacity utilization. Report jobs return 202 with a job identifier and polling URL. Audit records are append-only through the application; no public audit PATCH/DELETE.

## 9. AI Copilot and controlled AI actions

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET, POST | `/ai/conversations` | List/create scoped conversations | Authenticated |
| GET | `/ai/conversations/{id}` | Conversation/messages | Owner/explicit authorized scope |
| POST | `/ai/conversations/{id}/messages` | Send user prompt; orchestrated tools only | Authorized user; rate-limited |
| GET | `/ai/recommendations` | Scoped recommendation list | Authorized user |
| GET | `/ai/recommendations/{id}` | Recommendation details, freshness | Authorized user |
| POST | `/ai/recommendations/{id}/accept` | Create draft normal workflow object | Authorized user |
| GET | `/ai/actions` | Scoped proposed actions | Authorized actor |
| GET | `/ai/actions/{id}` | Proposed payload and approval state | Authorized actor |
| POST | `/ai/actions/{id}/submit` | Submit proposed high-impact action for approval | Authorized actor |
| POST | `/ai/actions/{id}/approve` | Human approval and revalidation | Authorized human approver |
| POST | `/ai/actions/{id}/reject` | Reject proposal | Authorized human approver |
| GET | `/ai/tool-executions` | Audit-safe execution history | Explicitly authorized user |

**Send message request:** `{"message":"Which items are below reorder point?"}`.

**Illustrative response:**

```json
{
  "message_id":"<uuid>",
  "answer":"Three accessible products are below their reorder point.",
  "classification":"FACT",
  "data_as_of":"2026-10-05T00:00:00Z",
  "tool_executions":[{"id":"<uuid>","tool_name":"get_low_stock_items","status":"COMPLETED"}],
  "proposed_action_ids":[]
}
```

AI classifications: `FACT`, `CALCULATION`, `PREDICTION`, `RECOMMENDATION`, `ACTION` as applicable. Operational answers should show data freshness; predictions/recommendations must not masquerade as facts. AI tools are **internal registry functions**, not arbitrary publicly invokable HTTP endpoints. Read tools may execute when authorized. Write-oriented tools may prepare drafts; high-impact execution always requires human approval, fresh state and the same service validations as human workflows. The LLM has no direct SQL, ORM, or unrestricted database access. Tool inputs use validated schemas; all retrieved names, notes, uploads and supplier text are untrusted data, not instructions. AI tool outputs are permission-filtered, including cost restrictions. Log tool calls and approvals without leaking secrets or unnecessarily retaining sensitive prompts.

## 10. Filtering, sorting and search

- Products: `search`, `sku`, `category_id`, `brand_id`, `status`, `ordering`.
- Inventory: `warehouse_id`, `product_id`, `low_stock`, `ordering`; warehouse scope enforced.
- Batches: `warehouse_id`, `product_id`, `expiry_before`, `status`.
- Movements: `warehouse_id`, `product_id`, `movement_type`, `date_from`, `date_to`.
- POs: `supplier_id`, `warehouse_id`, `status`, `date_from`, `date_to`.
- Sales: `customer_id`, `warehouse_id`, `status`, `date_from`, `date_to`.
- Transfers: `source_warehouse_id`, `destination_warehouse_id`, `status`.
- Approvals: `status`, `action_type`, `requested_by` subject to scope.
- Audit: `action`, `entity_type`, `entity_id`, `date_from`, `date_to` subject to audit permission.

All searches must be bounded, parameterized and indexed appropriately. Unsupported sort fields return validation errors. Never use client-provided organization filters to widen scope.

## 11. Representative end-to-end API flows

### Purchase receipt

1. `POST /purchase-orders` creates draft.
2. `POST /purchase-orders/{id}/submit` requests approval.
3. Authorized manager/admin approves through the canonical approval flow.
4. `POST /goods-receipts` creates receipt draft against approved PO.
5. `POST /goods-receipts/{id}/post` with `Idempotency-Key` validates receipt and atomically updates stock, cost, PO received totals, movements and audit.

### Sale with competing stock demand

1. Two clients create drafts against the same inventory.
2. Each calls `POST /sales-orders/{id}/confirm`.
3. Service locks the same inventory position; first valid transaction reserves the stock.
4. Second transaction re-reads stock and receives `409` or documented validation error when insufficient.
5. Completion consumes reservations once, posts `SALE` movement and audit.

### AI-assisted reorder

1. User sends Copilot request.
2. Orchestrator calls allowlisted read tools using the user's tenant/warehouse/cost scope.
3. Backend returns deterministic stock/reorder data with freshness.
4. AI explains recommendation; user accepts to create a draft PO.
5. Normal PO submission/approval workflow applies. AI cannot approve its own high-impact action.

## 12. Security and privacy contract

- All resource lookups enforce tenant scope and warehouse scope before data is returned.
- Mask cross-tenant IDs using a consistent 404/403 policy; never leak existence via detailed errors.
- Verify CSRF for cookie-authenticated unsafe requests; set strict CORS and trusted-origin configuration.
- Rate-limit login, AI messages, exports and uploads. Use object storage with authorized, short-lived file download access.
- Validate file content type, extension, size and tenant ownership; imported records pass through normal services.
- Do not expose sensitive auth tokens, internal prompts, arbitrary SQL, unrestricted tool invocation or raw provider traces.
- Correlation/request IDs connect API, service, task and audit logs; logs exclude secrets.

## 13. Open business decisions — contract gates

| Decision | API design intentionally left open |
|---|---|
| DEC-001 Commercial pricing | No SaaS billing/subscription API yet |
| DEC-002 Organization scale | No tenant-size limits or quotas hard-coded |
| DEC-011 Retention | No final deletion/export retention promises |
| DEC-012 SUPER_ADMIN tenant data | No assumption of unrestricted operational-data endpoints |
| DEC-013 Multi-org users | Active organization switching and membership semantics pending |
| DEC-015 Partial fulfillment | No partial shipment/fulfillment API until approved |
| DEC-016 Invoicing depth | No final invoice generation/lifecycle contract |
| DEC-017 Payment depth | No final payment recording/reconciliation contract |

**Implementation gate:** resolve relevant decisions before implementing affected endpoints. Other independently specified modules may proceed.

## 14. Contract testing and acceptance

- Generate/maintain an OpenAPI 3.x document from DRF endpoint schemas; validate representative examples in CI.
- Test success, validation, authentication, role/warehouse authorization, cross-tenant isolation, hidden cost fields, invalid transitions, duplicate requests and idempotency.
- Test two concurrent stock-consuming requests; only valid consumption succeeds and stock never goes negative.
- Test receipt and transfer rollback on failure, movement/audit atomicity and batch reconciliation.
- Test AI tool authorization, prompt-injection-resistant tool boundary, stale proposal revalidation and mandatory human approval.
- Test report/export scope and no cross-tenant leakage.
- Finalize exact field nullability, enum transitions, permissions matrix and endpoint response schemas during implementation review against the authoritative FRD and Business Rules.

## 15. Next artifacts

1. `UI-SPECIFICATION.md`: information architecture, page inventory, roles, user journeys, forms, tables, empty/error/loading states and approval UX.
2. `AI-SPECIFICATION.md`: tool schemas, orchestration, recommendation methods, approval gates, provider adapter and evaluation strategy.
3. Implementation planning: milestone breakdown and traceability to BRD/FRD/business rules.

**Core rule:** The API is a transport boundary. All authorization, stock integrity, approvals and authoritative calculations remain enforced by application services and PostgreSQL transactions.
