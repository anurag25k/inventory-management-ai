# Inventory Management AI — Engineering Guidelines

## 1. Project Overview

This is a production-oriented, multi-tenant Inventory Management

System with AI-powered inventory intelligence and controlled

agentic workflows.

The application is intended to support real businesses and must

be designed for security, reliability, maintainability,

scalability, and auditability.

The system includes:

- Organizations

- Users

- Roles and permissions

- Warehouses

- Products

- Categories

- Brands

- Units

- Suppliers

- Customers

- Purchasing

- Goods receiving

- Inventory

- Stock adjustments

- Stock transfers

- Sales

- Invoices

- Returns

- Payments

- Reports

- Notifications

- Audit logs

- AI Inventory Copilot

- AI-powered forecasting

- AI recommendations

- AI-assisted purchasing

- AI anomaly detection

- Human-approved AI actions

# 2. Technology Stack

## Frontend

- Next.js

- TypeScript

- Tailwind CSS

- shadcn/ui

- React Hook Form

- Zod

- TanStack Query

## Backend

- Python

- Django

- Django REST Framework

## Database

- PostgreSQL

## Background Processing

- Celery

- Redis

## Storage

- S3-compatible object storage

## Infrastructure

- Docker

- Docker Compose for local development

- GitHub Actions for CI/CD

## AI

- Provider-agnostic LLM integration

- Tool-based AI agent architecture

- Human-in-the-loop approval for high-impact actions

# 3. Core Engineering Principles

The application must be built as a production-quality system.

Prioritize:

1. Correctness

2. Security

3. Data integrity

4. Maintainability

5. Testability

6. Performance

7. Scalability

8. Good user experience

Do not prioritize speed of implementation over correctness.

# 4. Development Workflow

Never attempt to build the entire application in one step.

Work incrementally.

The expected development order is:

1. Requirements

2. Functional requirements

3. Architecture

4. Database design

5. Project foundation

6. Authentication

7. Multi-tenancy

8. Authorization

9. Product/catalog

10. Inventory engine

11. Purchasing

12. Sales

13. Returns

14. Reporting

15. Notifications

16. Audit logging

17. AI foundation

18. AI Copilot

19. AI forecasting

20. AI recommendations

21. AI actions

22. Testing

23. Security hardening

24. Performance optimization

25. Observability

26. CI/CD

27. Production deployment

Only implement the phase requested by the current task.

Do not implement future features unless explicitly requested.

# 5. Documentation Rules

Before implementing a feature, inspect the relevant documentation

inside `/docs`.

Important documents will include:

- [BRD.md](http://BRD.md)

- [FRD.md](http://FRD.md)

- [BUSINESS-RULES.md](http://BUSINESS-RULES.md)

- [ARCHITECTURE.md](http://ARCHITECTURE.md)

- [DATABASE-DESIGN.md](http://DATABASE-DESIGN.md)

- [UI-SPECIFICATION.md](http://UI-SPECIFICATION.md)

- [SECURITY-ARCHITECTURE.md](http://SECURITY-ARCHITECTURE.md)

- [AI-ARCHITECTURE.md](http://AI-ARCHITECTURE.md)

If a requirement is ambiguous:

1. Check the existing documentation.

2. Check existing implementation.

3. Do not invent important business behavior.

4. Identify the ambiguity clearly.

Do not silently make major product decisions.

# 6. Repository Structure

The authoritative structure is:

```text
inventory-management-ai/
├── backend/
├── frontend/
├── infrastructure/
├── docs/
├── AGENTS.md
├── README.md
├── .gitignore
└── docker-compose.yml
```

The frontend should remain separated from the backend.

The backend should be modularized by business domain.

# 7. Frontend Architecture

Use Next.js with TypeScript.

Keep business logic out of React components.

Prefer:

components/

features/

hooks/

lib/

types/

Organize feature-specific code under feature modules.

Reusable UI components should live in the shared UI/component layer.

Use:

- React Hook Form for complex forms

- Zod for frontend validation

- TanStack Query for server state where appropriate

Do not duplicate API logic across components.

Create reusable API/data-access functions.

# 8. Backend Architecture

Use Django and Django REST Framework.

Separate:

- API layer

- serializers

- services/application logic

- domain/business logic

- data access

- infrastructure

Do not put large amounts of business logic inside:

- views

- serializers

- URL handlers

Business-critical workflows should be implemented through

reusable service-layer functions.

For example:

inventory_[service.py](http://service.py)

sales_[service.py](http://service.py)

purchase_[service.py](http://service.py)

The same business service should be usable by:

- REST APIs

- background jobs

- AI tools

Do not implement the same business rule independently in

multiple places.

# 9. Database Rules

Use PostgreSQL.

All schema changes must use migrations.

Never modify the production database manually.

Use:

- foreign keys

- unique constraints

- check constraints

- indexes

- appropriate database-level validation

Use database constraints where correctness requires them.

Do not rely exclusively on frontend validation.

# 10. Multi-Tenancy

This application is multi-tenant.

Organizations must be isolated from each other.

Tenant-owned records should contain an organization relationship

where appropriate.

Never trust an organization ID provided by the frontend.

The authenticated user's organization membership must determine

the organization context.

Every tenant-scoped query must enforce organization isolation.

A user from Organization A must never be able to:

- read Organization B data

- modify Organization B data

- delete Organization B data

- infer sensitive Organization B information

Write automated tests for organization isolation.

# 11. Authentication

Authentication must be handled securely.

Never store authentication credentials or secrets in frontend

source code.

Never store sensitive authentication tokens in localStorage.

Use secure session/cookie mechanisms appropriate for the chosen

architecture.

Protected API endpoints must verify authentication.

Authentication and authorization are separate concerns.

# 12. Authorization and RBAC

Initial roles:

- SUPER_ADMIN

- ADMIN

- MANAGER

- INVENTORY_STAFF

- SALES_STAFF

- VIEWER

Permissions should be granular.

Examples:

[products.read](http://products.read)

products.create

products.update

products.delete

[inventory.read](http://inventory.read)

inventory.adjust

inventory.transfer

[purchases.read](http://purchases.read)

purchases.create

purchases.approve

[sales.read](http://sales.read)

sales.create

sales.cancel

[reports.read](http://reports.read)

users.manage

settings.manage

Always enforce authorization on the backend.

Frontend permission checks are for user experience only and

must never be treated as the security boundary.

# 13. Inventory Rules

Inventory is business-critical.

Never modify inventory quantities directly from frontend code.

Every stock-changing operation must go through the backend

inventory service.

Every stock mutation must:

1. Authenticate the user.

2. Verify authorization.

3. Validate the input.

4. Verify the organization.

5. Lock relevant records when necessary.

6. Perform the change transactionally.

7. Create an inventory movement.

8. Create an audit record when appropriate.

Inventory movement types may include:

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

Inventory must be traceable.

The system should be able to answer:

"Why did the stock quantity change?"

Never silently modify stock.

# 14. Transactions

Use database transactions for multi-step business operations.

Examples:

Purchase receiving:

validate

→ lock inventory

→ update inventory

→ create movement

→ update receipt

→ create audit record

→ commit

Sales:

validate

→ lock inventory

→ verify availability

→ reserve/deduct stock

→ create sale

→ create movement

→ create invoice

→ audit

→ commit

If any required step fails, roll back the transaction.

Never leave partially completed business transactions.

# 15. Concurrency

Inventory operations must account for concurrent users.

Example:

If two users attempt to sell the final five units

simultaneously, the system must not allow stock to become

negative accidentally.

Use appropriate PostgreSQL/Django locking and transactions.

Write concurrency tests for important inventory workflows.

# 16. Validation

Validate input at multiple appropriate layers.

Frontend:

- Zod

- form validation

Backend:

- Django validation

- DRF validation

- business-rule validation

Database:

- constraints where appropriate

Never trust client-provided values for:

- organization

- permissions

- prices

- inventory quantities

- user identity

- ownership

# 17. API Design

Build predictable REST APIs.

Use:

- appropriate HTTP methods

- consistent status codes

- consistent error responses

- pagination

- filtering

- sorting

- validation errors

Do not expose internal implementation details.

Do not return unnecessary sensitive information.

# 18. Error Handling

Never silently swallow errors.

Errors should:

- be logged appropriately

- return safe user-facing messages

- preserve useful debugging information in server logs

- avoid leaking secrets or internal stack traces

Frontend must provide:

- loading states

- error states

- empty states

- success feedback

Do not leave users with a blank screen when an operation fails.

# 19. Audit Logging

Important actions must be auditable.

Audit events may include:

- login

- user creation

- role changes

- product changes

- inventory adjustments

- stock transfers

- purchase changes

- sales changes

- returns

- payment changes

- AI actions

Audit logs should capture appropriate information such as:

- organization

- actor

- action

- entity type

- entity ID

- timestamp

- old values

- new values

Normal users must not be able to modify audit history.

# 20. AI Architecture

AI is an assistant and controlled agent, not a replacement

for the application's business logic.

The architecture should be:

User

↓

AI Orchestrator

↓

Approved Tool Registry

↓

Django Services

↓

PostgreSQL

AI must NEVER directly access PostgreSQL.

AI must NEVER execute arbitrary SQL.

AI must NEVER bypass Django authorization.

AI must NEVER bypass business rules.

AI must NEVER modify database records directly.

AI may interact with the system only through explicitly

approved tools.

# 21. AI Tool Rules

Every AI tool must:

1. Authenticate the user.

2. Determine organization context server-side.

3. Check user permissions.

4. Validate parameters.

5. Call existing application services.

6. Return structured results.

7. Log important actions.

Example read tools:

get_product()

search_products()

get_inventory()

get_low_stock_products()

get_stock_movements()

get_sales_summary()

get_purchase_summary()

get_supplier_performance()

get_profitability()

Do not create a generic:

execute_sql()

AI tools must be narrowly scoped.

# 22. AI Read vs Write Operations

Read operations may generally be executed automatically

when authorized.

Write operations require stricter controls.

Examples:

Read:

- inventory analysis

- sales analysis

- supplier analysis

- stock forecasting

Write:

- stock adjustment

- purchase order creation

- cancelling an order

- changing product prices

- bulk operations

High-impact AI actions must require explicit human approval.

# 23. AI Human-in-the-Loop

For high-impact actions use:

AI proposes

↓

User reviews

↓

User explicitly approves

↓

Backend re-validates

↓

Normal application service executes

↓

Audit log

Never execute a high-impact AI action solely because the

LLM requested it.

Always re-check current database state before execution.

AI recommendations can become stale between proposal and approval.

# 24. AI Numerical Accuracy

Do not ask the LLM to perform critical numerical calculations

when deterministic backend calculations are possible.

For example:

Do NOT rely on the LLM to calculate:

- stock quantities

- revenue

- profit

- inventory value

- reorder quantity

- tax

- invoice totals

Calculate these in backend/application code.

Use the AI to interpret structured results and explain them.

# 25. AI Hallucination Prevention

AI responses must distinguish between:

FACT

CALCULATION

PREDICTION

RECOMMENDATION

Never invent:

- products

- quantities

- orders

- suppliers

- prices

- sales

- inventory movements

- financial values

If required data is unavailable, say so.

Do not fabricate missing information.

# 26. Prompt Injection Protection

Treat external/user-provided content as untrusted.

Do not allow product descriptions, supplier names,

customer notes, uploaded documents, or other database content

to override system instructions or tool authorization.

AI tools must independently enforce authorization.

Never rely on the LLM to decide whether a user has permission

to perform an action.

# 27. Background Jobs

Use Celery + Redis for asynchronous work.

Background jobs should be:

- idempotent where possible

- retryable

- observable

- safe against duplicate execution

Examples:

- email notifications

- low-stock scanning

- daily reports

- AI briefings

- anomaly detection

Never create duplicate business transactions because a job

was retried.

# 28. File Uploads

Treat uploaded files as untrusted.

Validate:

- file type

- file size

- filename

- content where appropriate

Do not allow uploaded files to execute as application code.

Use object storage rather than storing large files directly

inside PostgreSQL.

# 29. Security

Never commit:

- passwords

- API keys

- tokens

- private certificates

- production credentials

- .env files containing secrets

Use environment variables or an appropriate secret manager.

Never expose backend secrets to the browser.

Use secure CORS configuration.

Use appropriate CSRF protection.

Validate uploaded files.

Protect sensitive endpoints against abuse.

# 30. Testing Requirements

Every important business feature must have tests.

Backend tests should include:

- unit tests

- service tests

- API tests

- authorization tests

- organization isolation tests

- transaction tests

- concurrency tests

Frontend tests should cover important:

- forms

- components

- interactions

End-to-end tests should cover critical user workflows.

# 31. Required Critical E2E Workflows

At minimum test:

1. Register

2. Login

3. Create organization

4. Create warehouse

5. Create product

6. Create supplier

7. Create purchase order

8. Receive goods

9. Verify inventory

10. Create customer

11. Create sale

12. Verify inventory deduction

13. Process return

14. Transfer inventory

15. Generate report

16. Ask AI Copilot a question

17. Generate AI reorder recommendation

18. Create AI draft purchase order

19. Approve AI action

20. Verify audit log

# 32. Code Quality

Prefer simple, readable code.

Avoid:

- unnecessary abstractions

- giant files

- giant functions

- duplicated logic

- magic numbers

- magic strings

- hidden side effects

Use clear names.

Prefer explicit behavior over clever code.

Keep functions focused.

# 33. Dependency Management

Do not install a new dependency just because it is convenient.

Before adding a dependency:

1. Check whether the project already has an equivalent.

2. Check whether the functionality can be implemented simply.

3. Consider maintenance and security.

4. Use a stable, actively maintained package when appropriate.

Document important dependency decisions.

# 34. Database Changes

Before changing the database:

1. Inspect the current schema.

2. Check [DATABASE-DESIGN.md](http://DATABASE-DESIGN.md).

3. Check existing migrations.

4. Avoid duplicate tables/columns.

5. Create a proper migration.

6. Test migration forward.

7. Test migration behavior where relevant.

Never delete production data as part of a normal development task.

# 35. Git Workflow

Use small logical commits.

Examples:

feat: add product management

feat: implement inventory ledger

feat: add purchase receiving

feat: add AI inventory copilot

fix: prevent negative inventory

test: add concurrent stock sale tests

chore: improve CI pipeline

Do not combine unrelated features into one commit.

# 36. Before Making Changes

Before implementing a task:

1. Read [AGENTS.md](http://AGENTS.md).

2. Read relevant documentation.

3. Inspect existing implementation.

4. Identify affected modules.

5. Identify database impact.

6. Identify security impact.

7. Identify testing requirements.

8. State a concise implementation plan.

9. Implement the smallest coherent change.

# 37. After Making Changes

After implementing a task:

1. Review changed files.

2. Remove unused code.

3. Run formatting.

4. Run lint.

5. Run type checking.

6. Run backend tests.

7. Run frontend tests.

8. Run relevant integration tests.

9. Run build.

10. Review database migrations.

11. Review authorization.

12. Review organization isolation.

13. Review error handling.

Do not claim success if checks fail.

# 38. Definition of Done

A feature is NOT complete merely because it works in the browser.

A feature is complete when:

- functionality is implemented

- requirements are satisfied

- validation exists

- authorization exists

- tenant isolation is maintained

- errors are handled

- loading states exist

- empty states exist

- tests exist

- migrations exist where necessary

- audit logging exists where appropriate

- documentation is updated

- lint passes

- type checks pass

- tests pass

- build passes

# 39. Important Rule for Cursor

Do not make assumptions simply to keep moving.

If a decision materially affects:

- business behavior

- financial calculations

- inventory behavior

- security

- database structure

- user permissions

- AI actions

stop and identify the decision rather than silently inventing

a requirement.

However, do not ask unnecessary questions for trivial

implementation details.

Use the existing documentation as the source of truth.

# 40. Final Principle

Build this application as if another engineering team will

maintain it five years from now.

Correctness is more important than speed.

Security is more important than convenience.

Inventory integrity is more important than UI shortcuts.

AI should enhance the application, never bypass its rules.