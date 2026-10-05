# IMPLEMENTATION PLAN
## Inventory Management AI — Production-Oriented Build Plan

**Status:** Implementation planning  
**Purpose:** Bridge the approved BRD, FRD, business rules, architecture, database, API, UI, and AI specifications into incremental development.

## 1. Implementation Principles

1. Build the transactional business foundation before advanced AI.
2. Backend is authoritative for authorization, stock, calculations, approvals, and business rules.
3. PostgreSQL constraints, transactions, and row locks are part of data integrity.
4. Inventory mutations must pass through application services.
5. AI may use approved tools but never direct database access or arbitrary SQL.
6. High-impact AI actions require explicit human approval and revalidation.
7. Build vertical slices rather than generating the whole system at once.
8. Keep the system a modular monolith initially.
9. Do not invent behavior for unresolved requirement decisions.

## 2. Source Authority

Implementation decisions follow this precedence:

1. Confirmed requirement decisions
2. Business rules
3. FRD
4. Architecture
5. Database design
6. API specification
7. UI specification
8. AI specification
9. This implementation plan
10. Implementation convenience

The following decisions remain unresolved and must not be silently invented:

- DEC-001 commercial pricing model
- DEC-002 organization scale
- DEC-011 data retention
- DEC-012 SUPER_ADMIN operational-data access
- DEC-013 multi-organization users
- DEC-015 partial sales fulfillment
- DEC-016 invoicing depth
- DEC-017 payment recording depth

## 3. Repository Structure

Recommended monorepo:

```text
inventory-management-ai/
├── AGENTS.md
├── README.md
├── .env.example
├── docker-compose.yml
├── docs/
├── backend/
│   ├── manage.py
│   ├── requirements/
│   ├── config/
│   ├── apps/
│   │   ├── organizations/
│   │   ├── users/
│   │   ├── catalog/
│   │   ├── warehouses/
│   │   ├── inventory/
│   │   ├── suppliers/
│   │   ├── purchasing/
│   │   ├── customers/
│   │   ├── sales/
│   │   ├── returns/
│   │   ├── transfers/
│   │   ├── approvals/
│   │   ├── notifications/
│   │   ├── audit/
│   │   ├── reports/
│   │   └── ai/
│   └── common/
├── frontend/
│   └── src/
│       ├── app/
│       ├── components/
│       ├── features/
│       ├── hooks/
│       ├── lib/
│       └── types/
├── infrastructure/
└── .github/workflows/
```

## 4. Phase 0 — Documentation Freeze

Before implementation:

- place approved specifications under `docs/`
- preserve unresolved decisions
- create a traceability matrix
- document local setup
- document environment variables
- verify terminology across documents

Do not start feature coding until the repository rules are available to Cursor.

## 5. Phase 1 — Development Infrastructure

Local services:

```text
Next.js
   |
Django / DRF
   |
PostgreSQL
   |
Redis → Celery
```

Set up:

- Docker Compose
- PostgreSQL
- Redis
- Django
- Next.js
- backend and frontend Dockerfiles
- `.env.example`
- basic health endpoint
- development README
- initial GitHub Actions workflow

Never commit real credentials.

## 6. Phase 2 — Backend Foundation

Install and configure:

- Django
- Django REST Framework
- PostgreSQL driver
- environment configuration
- CORS
- Celery
- Redis
- pytest / pytest-django
- structured logging
- API exception handling
- pagination
- authentication

Use settings separated for development, testing, staging, and production.

Backend layering:

```text
API View
  ↓
Serializer
  ↓
Application Service
  ↓
Domain Rules
  ↓
Query/Data Access
  ↓
Django ORM
  ↓
PostgreSQL
```

Complex business logic must not live inside views or serializers.

## 7. Phase 3 — Database Foundation

Use:

- UUID primary keys
- timezone-aware timestamps
- Decimal/Numeric quantities
- Decimal/Numeric money
- foreign keys
- unique constraints
- check constraints
- indexes
- Django migrations

Build models in dependency order:

1. organizations
2. users
3. memberships
4. warehouse access
5. categories
6. brands
7. units
8. products
9. warehouses
10. inventory positions
11. inventory batches
12. batch positions
13. inventory movements
14. suppliers
15. purchase orders/items
16. goods receipts
17. customers
18. sales orders/items
19. reservations
20. returns
21. transfers
22. approvals
23. notifications
24. audit
25. reports
26. AI entities

Do not create one giant migration for the complete system.

## 8. Phase 4 — Authentication and Multi-Tenancy

Implement:

- user registration/creation
- login
- logout
- secure session/authentication
- password reset
- organization creation
- organization membership
- role assignment
- account activation/deactivation

Use secure HttpOnly cookies rather than storing authentication tokens in localStorage.

Organization context must be derived from authenticated membership.

Never trust a browser-supplied `organization_id` as authorization.

Roles:

- SUPER_ADMIN
- ADMIN
- MANAGER
- INVENTORY_STAFF
- SALES_STAFF
- VIEWER

## 9. Phase 5 — Warehouse Authorization

Implement reusable warehouse permission checks.

Rules:

- SUPER_ADMIN/ADMIN/MANAGER: organization warehouses
- INVENTORY_STAFF/SALES_STAFF/VIEWER: assigned warehouses

Every warehouse-scoped query must enforce this on the backend.

## 10. Phase 6 — Catalog and Warehouses

Implement:

- categories
- brands
- units
- explicit unit conversions
- products
- SKU
- lifecycle
- reorder point
- commercial fields
- basic tax fields
- warehouses

Product lifecycle:

```text
DRAFT → ACTIVE → INACTIVE → ARCHIVED
```

Inactive products remain available in history/reports but cannot enter new sales or purchases.

## 11. Phase 7 — Inventory Core

This is the highest-risk business module.

Inventory model:

```text
Physical Stock
├── Sellable Stock
│   ├── Available Stock
│   └── Reserved Stock
├── Damaged Stock
└── Expired Stock
```

Invariants:

```text
Available = Sellable - Reserved
Physical = Sellable + Damaged + Expired
Sellable = Available + Reserved
```

All quantities must remain non-negative.

Available quantity should be derived rather than independently stored.

Every confirmed stock change must create an inventory movement.

## 12. Inventory Transaction Design

All inventory-changing operations must use an application service.

Use:

```python
transaction.atomic()
```

with row locking such as:

```python
select_for_update()
```

For operations involving multiple inventory rows, lock them in deterministic order to reduce deadlock risk.

The backend must prevent negative stock inside the transaction, not through a frontend pre-check.

## 13. Inventory Movements

Supported movement types:

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

Movement records should retain:

- product
- warehouse
- quantity
- unit
- movement type
- source transaction
- user
- timestamp
- unit cost where applicable

Movement history is authoritative and append-oriented.

## 14. Batch and Cost Management

Implement:

- batch/lot number
- expiry date
- batch quantities
- batch status
- warehouse/product association

Serial tracking remains future scope.

Inventory valuation uses Weighted Average Cost.

Cost calculations remain deterministic backend logic. AI may explain cost data but does not own valuation.

## 15. Phase 8 — Adjustments and Opening Stock

Opening stock:

```text
Authorized Entry/Import
→ Inventory Change
→ Opening Movement
→ Audit
```

Adjustments require:

- authorization
- reason
- validation
- inventory movement
- audit

Draft adjustments do not change inventory.

## 16. Phase 9 — Purchasing

Implement:

- suppliers
- supplier codes
- purchase orders
- PO items
- approval
- receiving
- purchase returns

Submitted purchase orders require ADMIN or MANAGER approval.

Do not invent monetary approval thresholds.

Receiving should atomically:

1. validate PO
2. validate organization/warehouse
3. validate quantities
4. update inventory
5. update weighted average cost
6. create movements
7. update receipt/PO state
8. create audit event

Purchase returns only affect inventory at the confirmed/dispatched stage defined by the business rules.

## 17. Phase 10 — Sales

Implement:

- customers
- customer codes
- sales orders
- sales items
- reservations
- completion
- cancellation

Normal sales require no approval.

Confirmed sales reserve stock.

Draft sales do not reserve stock.

Reservation must guarantee:

```text
Reserved ≤ Sellable
```

Sales completion decreases the appropriate stock and creates a movement.

Partial fulfillment remains dependent on DEC-015.

## 18. Phase 11 — Customer Returns

Customer returns require inspection.

Results:

```text
acceptable → sellable
damaged    → damaged stock
expired    → expired stock
```

Do not automatically return every customer return to sellable inventory.

## 19. Phase 12 — Transfers

Implement:

- transfer draft
- source/destination warehouse
- transfer items
- approval
- reservation where applicable
- execution
- transfer movements
- audit

Transfers require MANAGER or ADMIN approval.

Execution must be transactional.

## 20. Phase 13 — Reusable Approval Framework

Create a generic approval entity/service supporting:

- source object
- action type
- requester
- reviewer
- status
- decision
- comments
- timestamps

Potential states:

```text
PENDING
APPROVED
REJECTED
STALE
CANCELLED
```

Approval execution must always revalidate current data.

## 21. Approval Revalidation

Before executing an approved action:

1. authenticate
2. verify organization
3. verify permissions
4. reload current data
5. verify state
6. verify quantities
7. re-run business rules
8. execute transaction
9. audit result

This same pattern will be used for AI actions.

## 22. Phase 14 — Notifications and Audit

Notifications initially cover:

- low stock
- expiry
- approval requests
- approval decisions
- important failures

Audit important actions including:

- security events
- membership/role changes
- inventory changes
- approvals
- overrides
- transfers
- AI recommendations
- AI approvals
- AI execution

## 23. Phase 15 — Reports

Initial reports:

- inventory summary
- low stock
- stock movements
- sales summary
- purchasing summary
- supplier performance
- warehouse inventory distribution
- expiring batches
- authorized inventory valuation

Cost visibility must respect role permissions.

## 24. Phase 16 — Object Storage and Imports

Use an S3-compatible abstraction for:

- imports
- report exports
- attachments
- documents

Validate:

- file type
- size
- filename
- authorization
- organization scope

Never allow arbitrary filesystem access.

## 25. Phase 17 — Frontend Foundation

Use:

- Next.js
- TypeScript
- Tailwind CSS
- shadcn/ui
- React Hook Form
- Zod
- TanStack Query

Feature structure:

```text
features/
├── auth/
├── dashboard/
├── inventory/
├── products/
├── warehouses/
├── suppliers/
├── purchasing/
├── customers/
├── sales/
├── transfers/
├── approvals/
├── reports/
├── notifications/
├── audit/
└── ai/
```

## 26. Frontend Build Order

1. authentication
2. application shell
3. dashboard
4. products
5. warehouses
6. inventory
7. movements
8. suppliers
9. purchasing
10. receiving
11. customers
12. sales
13. returns
14. transfers
15. approvals
16. reports
17. notifications
18. audit
19. AI Copilot

Every screen should support loading, empty, success, error, retry, pagination/filtering, and unauthorized states as applicable.

## 27. Frontend API Layer

Create one centralized API client.

TanStack Query manages:

- server state
- caching
- invalidation
- loading
- errors

Zod provides early user-facing validation but is not the business authority.

## 28. Phase 18 — AI Foundation

Do not build advanced AI before the transactional system works.

Architecture:

```text
User
 ↓
AI Copilot
 ↓
AI Orchestrator
 ↓
Context + Policy
 ↓
Approved Tool Registry
 ↓
Application Services
 ↓
PostgreSQL
```

The LLM connects through a provider adapter.

AI must never:

- execute arbitrary SQL
- access PostgreSQL directly
- bypass authorization
- bypass business rules
- silently change inventory
- expose unauthorized cost data
- execute high-impact writes without approval

## 29. AI Context

Runtime context includes:

- user_id
- organization_id
- role
- warehouse_scope
- permissions
- current warehouse
- conversation_id

The LLM cannot choose or override these values.

## 30. AI Tool Registry

Each tool defines:

- name
- version
- description
- input schema
- output schema
- risk level
- permissions
- roles
- organization scope
- warehouse scope
- handler

Risk levels:

```text
READ
ANALYSIS
DRAFT
HIGH_IMPACT_WRITE
```

## 31. Initial AI Read Tools

Implement:

```text
get_inventory_summary
get_low_stock_items
get_stock_movements
get_inventory_by_product
get_inventory_by_warehouse
get_expiring_batches
get_sales_summary
get_sales_demand
get_purchase_pipeline
get_supplier_performance
get_product_details
get_recent_business_activity
```

All tools enforce the same authorization as normal APIs.

## 32. AI Response Types

Distinguish:

```text
FACT
CALCULATION
PREDICTION
RECOMMENDATION
ACTION
```

Numerical calculations stay deterministic in backend services.

AI should not present predictions or recommendations as confirmed facts.

## 33. AI Freshness

Current-data responses should communicate data freshness when relevant.

Example:

```text
Based on data through 10:30 AM
```

Do not imply real-time accuracy when data is delayed.

## 34. AI Recommendations

Implement in this order:

1. demand analysis
2. reorder recommendations
3. supplier recommendations
4. anomaly detection
5. daily briefing

Reorder analysis may use:

- available stock
- reserved stock
- reorder point
- sales history
- demand
- supplier lead time
- purchase pipeline
- trends

AI explains recommendations; deterministic backend logic owns authoritative numbers.

## 35. AI Draft Actions

Initial draft tools:

```text
create_purchase_order_draft
create_transfer_draft
create_inventory_adjustment_draft
```

These create normal application drafts.

They do not silently execute high-impact operations.

## 36. AI Action Lifecycle

Use:

```text
PROPOSED
→ DRAFT
→ PENDING_APPROVAL
→ APPROVED
→ REVALIDATED
→ EXECUTING
→ EXECUTED
```

Terminal states:

```text
REJECTED
STALE
FAILED
CANCELLED
```

Immediately before execution, reload current data and re-run all authorization and business rules.

## 37. AI Security

Treat user-controlled or retrieved content as untrusted data.

Prompt injection defenses must cover:

- product descriptions
- supplier notes
- CSV/import content
- uploaded documents
- other business text

Untrusted content must never elevate instructions or permissions.

## 38. AI Observability

Track, subject to privacy/security rules:

- request ID
- conversation ID
- user
- organization
- model
- tool calls
- latency
- token usage where available
- errors
- approval
- execution outcome

Never log secrets.

## 39. Background Jobs

Use Celery for:

- scheduled reports
- daily briefing
- low-stock analysis
- expiry checks
- anomaly analysis
- notifications
- large imports
- large exports

Tasks must be idempotent and retry-safe.

Redis is not the system of record.

## 40. Testing Pyramid

### Unit
- calculations
- business rules
- permissions
- inventory services
- recommendation logic

### Integration
- database transactions
- API endpoints
- tenant isolation
- warehouse access
- approvals

### End-to-end
Critical workflows from login through final state verification.

## 41. Critical Inventory Tests

At minimum:

1. cannot consume more than available stock
2. reservation cannot exceed sellable stock
3. damaged stock cannot be sold
4. expired stock cannot be sold
5. confirmed sale reserves stock
6. draft sale does not reserve
7. sale completion changes stock correctly
8. cancellation restores reservation correctly
9. concurrent sales cannot create negative stock
10. cross-organization access fails
11. unauthorized warehouse access fails
12. every confirmed stock mutation creates a movement

## 42. Critical AI Tests

Test:

- cross-tenant request
- warehouse scope violation
- unauthorized cost access
- prompt injection
- malicious product description
- malformed tool input
- invalid tool selection
- stale recommendation
- rejected approval
- duplicate execution
- concurrent execution
- failed execution
- audit creation

## 43. CI/CD

GitHub Actions should run:

```text
lint
→ type checks
→ backend tests
→ frontend tests
→ build
→ security checks
```

Protect `main` and require passing checks before merge.

## 44. Security Hardening

Before production:

- HTTPS
- secure cookies
- CSRF protection
- restricted CORS
- secure headers
- rate limiting
- object-level authorization
- file validation
- secret management
- dependency scanning
- audit logging
- safe error responses
- database backups

Never expose credentials, API keys, or internal stack traces.

## 45. Performance

After correctness, optimize:

- indexes
- N+1 queries
- pagination
- expensive aggregations
- unnecessary AI calls
- database connection use
- Celery workload

Important indexes include organization, warehouse/product, movement timestamps, expiry, status, approval, and audit timestamps.

## 46. Staging and Production

Staging should resemble production and use:

- containers
- managed PostgreSQL
- Redis
- object storage
- HTTPS
- environment-specific secrets
- migrations
- CI/CD
- synthetic/sanitized data

Production should use managed infrastructure where practical, automated backups, monitoring, health checks, and recovery procedures.

## 47. Deployment Sequence

```text
Build
→ Automated Tests
→ Security Checks
→ Versioned Images
→ Staging Deploy
→ Smoke Tests
→ Production Approval
→ Database Migration
→ Application Deploy
→ Health Checks
→ Critical Workflow Verification
→ Monitoring
```

Review migration SQL and test migrations on staging first.

## 48. Observability

Monitor:

- API latency
- error rates
- database performance
- Celery failures
- queue depth
- AI latency/errors
- storage failures
- authentication failures

Use structured logs with fields such as request ID, user ID, organization ID, operation, status, and duration.

## 49. Definition of Done

A feature is complete only when:

- requirement is mapped
- backend service exists
- authorization exists
- validation exists
- database integrity exists where appropriate
- API exists
- UI exists where required
- tests exist
- audit exists where required
- errors are handled
- documentation is updated
- CI passes

## 50. Milestones

### M1 — Foundation
Repository, Docker, PostgreSQL, Redis, Django, Next.js, CI.

### M2 — Identity
Authentication, organizations, memberships, roles, warehouse permissions.

### M3 — Catalog/Warehouses
Products, categories, brands, units, conversions, warehouses.

### M4 — Inventory Core
Positions, movements, opening stock, adjustments, batches, expiry, concurrency, weighted average cost.

### M5 — Purchasing
Suppliers, POs, approval, receiving, purchase returns.

### M6 — Sales
Customers, sales, reservations, completion, cancellations, customer returns.

### M7 — Transfers/Approvals
Transfers, reusable approval engine, revalidation, audit.

### M8 — Reports
Dashboards, reports, KPIs, notifications, audit UI.

### M9 — AI Read Intelligence
AI orchestrator, provider adapter, tool registry, Copilot, inventory/sales/purchasing/supplier queries.

### M10 — AI Recommendations
Demand, reorder, supplier recommendations, anomalies, daily briefing.

### M11 — AI Controlled Actions
Draft PO/transfer/adjustment, approval, revalidation, execution, audit.

### M12 — Production Readiness
Staging, production, backups, monitoring, security, performance, recovery, final E2E tests.

## 51. First Vertical Slice

Do not start by implementing every model.

First prove:

```text
Organization
→ User/Membership
→ Role
→ Warehouse
→ Product
→ Inventory Position
→ Opening Stock
→ Inventory Movement
→ Dashboard Inventory View
```

This validates the core tenant → warehouse → product → inventory relationship.

## 52. Second Vertical Slice

Then:

```text
Product
→ Inventory
→ Customer
→ Sales Order
→ Reservation
→ Sale Completion
→ Inventory Movement
→ Audit
```

This validates stock consumption.

## 53. Third Vertical Slice

Then:

```text
Supplier
→ Purchase Order
→ Approval
→ Goods Receipt
→ Inventory Increase
→ Weighted Average Cost
→ Movement
→ Audit
```

This validates replenishment.

## 54. Fourth Vertical Slice

Then:

```text
Inventory
→ Low Stock Detection
→ AI Read Tool
→ AI Recommendation
→ Purchase Order Draft
→ Human Approval
→ Revalidation
```

This validates the controlled AI architecture without uncontrolled autonomous writes.

## 55. Cursor Workflow

Cursor must read `AGENTS.md` before implementation.

For each task, give Cursor:

- relevant requirement
- business rules
- architecture constraints
- database entities
- API expectations
- acceptance criteria
- allowed files

Recommended prompt pattern:

```text
Read AGENTS.md first.

Implement only [specific feature].

Requirements:
...

Business rules:
...

Constraints:
- Do not bypass application services.
- Enforce organization and warehouse authorization.
- Use transactions where required.
- Do not change unrelated modules.

Tests required:
...

Before editing:
1. inspect existing implementation
2. identify relevant files
3. explain proposed changes

Then implement.
Run relevant tests.
Report files changed and test results.
```

Do not ask Cursor to "build the whole system."

## 56. Cursor Review Rules

For every generated change:

- inspect diff
- inspect migrations
- inspect authorization
- inspect transaction boundaries
- run tests
- reject unrelated refactors
- verify no secrets were introduced

## 57. Immediate Next Task

The first implementation task is **repository and development-environment bootstrap**.

Deliver:

1. monorepo directories
2. Django backend
3. Next.js frontend
4. PostgreSQL Docker service
5. Redis Docker service
6. backend Dockerfile
7. frontend Dockerfile
8. Compose configuration
9. `.env.example`
10. health endpoint
11. basic frontend page
12. Django/PostgreSQL connection
13. initial CI workflow
14. README setup instructions

## 58. First Milestone Acceptance Criteria

The foundation passes only when:

- a clean clone can start the project
- environment variables are documented
- PostgreSQL starts
- Redis starts
- Django starts
- Django connects to PostgreSQL
- Next.js starts
- frontend can reach backend
- health endpoint succeeds
- migrations run
- CI passes
- no secrets are committed
- README explains setup
- containers can restart without manual database repair

## 59. Final Architecture Principle

The central rule for the entire project is:

> **The application owns truth; AI helps users understand and act on that truth.**

AI may:

- recommend
- explain
- analyze
- create drafts
- request actions

But these remain deterministic application responsibilities:

```text
Authorization
Validation
Business Rules
Transactions
Inventory State
Approvals
Execution
Audit
```

This principle must remain intact throughout implementation.
