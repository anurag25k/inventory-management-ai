# Business Requirements Document

**Product:** Inventory Management System with AI Inventory Intelligence  
**Document type:** Business Requirements (BRD)  
**Phase:** Business Requirements  
**Status:** Draft for review  
**Audience:** Business owners, product stakeholders, and later design teams  

**Documentation revision:** 2026-10-04 — confirmed decisions synchronized; stale terminology and decision summaries corrected.  

This document describes **what** the business needs and **why**. It does not prescribe technologies, data models, APIs, or implementation design.

Related document: [REQUIREMENT-DECISIONS.md](./REQUIREMENT-DECISIONS.md)

---

## Requirement identifier conventions

Stable identifiers used in this document:

| Prefix | Domain |
|--------|--------|
| BR- | Cross-cutting business objectives and success |
| ORG- | Organizations / tenants |
| USR- | Users and membership |
| ROLE- | Roles and responsibilities |
| PERM- | Permission principles |
| PROD- | Products |
| CAT- | Categories |
| BRN- | Brands |
| UOM- | Units of measurement |
| WH- | Warehouses |
| INV- | Inventory |
| SUP- | Suppliers |
| PUR- | Purchasing |
| REC- | Goods receiving |
| CUS- | Customers |
| SAL- | Sales |
| PAY- | Payment recording (limited) |
| RET- | Returns |
| TRF- | Stock transfers |
| RPT- | Reporting |
| NOT- | Notifications |
| AUD- | Audit |
| AI- | AI intelligence |
| SEC- | Security and isolation |
| INT- | Data and business integrity |
| KPI- | Business KPIs |
| NFR- | Non-functional business expectations |
| WF- | End-to-end workflows |
| ASM- | Assumptions |
| CON- | Constraints |
| RSK- | Risks |
| DEP- | Dependencies |
| FUT- | Future scope |
| DEC- | Business decisions (open or confirmed) |

All IDs in this document are unique.

---

## 1. Executive Summary

### What the product is

The product is a production-oriented Inventory Management System for organizations that buy, store, and sell physical goods. It is multi-tenant: many independent businesses can use the same product while remaining strictly isolated from one another.

The system is the operational system of record for catalog, warehouse stock, purchasing, receiving, sales, returns, transfers, adjustments, and the history of why stock changed. It also provides an AI Inventory Copilot that answers authorized questions, explains inventory conditions, and can draft recommended actions. High-impact AI actions require human approval and then follow the same business controls as actions started by a person.

### Who it serves

It serves small and medium-sized businesses (SMBs) and the people who run inventory operations: owners, administrators, inventory and warehouse staff, procurement staff, sales staff, and managers who need trustworthy visibility.

### What problem it solves

Many SMBs still run inventory on spreadsheets, disconnected tools, or informal knowledge held by a few people. That leads to stockouts, overstock, dead inventory, weak purchasing, poor warehouse visibility, limited reporting, and little auditability. Managers cannot reliably ask “what do we have, where is it, why did it change, and what should we do next?”

### Why it is valuable

The product gives one controlled place to:

- Know physical, sellable, reserved, available, damaged, and expired stock by product and warehouse.
- Buy, receive, sell, transfer, adjust, and return goods with a traceable history.
- Assign people clear roles instead of shared, unrestricted access.
- See operational reports and alerts in time to act.
- Use AI as a permission-aware assistant, not as an unsupervised operator.

### What makes it different

BR-001. The product is designed as a serious operational system, not a simple catalog of create-read-update-delete screens.

BR-002. Inventory quantity is a controlled business asset. Stock does not change without a business reason and a traceable history.

BR-003. Multiple organizations are supported with strict data isolation.

BR-004. AI is a first-class capability, but it is bounded: it may read authorized information, explain, recommend, and draft. It may not silently change inventory, bypass approvals, or invent facts.

BR-005. Human approval is required for high-impact AI actions. AI output must distinguish fact, calculation, prediction, recommendation, and action.

---

## 2. Business Problem

SMB inventory operations commonly fail in the following ways.

BR-010. **Poor inventory visibility.** Owners and staff cannot quickly see physical, sellable, reserved, and available stock in each warehouse, including damaged and expired goods.

BR-011. **Manual stock tracking.** Spreadsheets, notebooks, and memory are used as the stock record. They go stale as soon as two people act at once.

BR-012. **Stockouts.** Goods are unavailable when a customer wants them because nobody saw the decline in time or because available stock was overstated.

BR-013. **Overstocking.** Capital is tied up in goods that are not needed, often because purchasing is based on habit rather than demand and current stock.

BR-014. **Dead inventory.** Slow-moving or obsolete goods sit unexamined, occupying space and distorting inventory value.

BR-015. **Poor warehouse visibility.** Multi-location businesses cannot see which warehouse holds stock, which location is short, and which location is overstocked.

BR-016. **Lack of supplier insight.** Buying decisions ignore lead time, reliability, price history, and past receipt quality because that history is not usable.

BR-017. **Manual purchasing decisions.** Purchase orders are built from memory, stock walks, or urgent shortages rather than reorder levels and demand.

BR-018. **Limited reporting.** Management cannot get consistent inventory, purchase, sales, aging, and stockout views without assembling data by hand.

BR-019. **Lack of auditability.** When stock is wrong, the business cannot answer who changed it, when, and why.

BR-020. **Difficulty using inventory through natural language.** Managers want to ask operational questions in ordinary language and receive answers grounded in the actual authorized records, not guesses.

BR-021. **Uncontrolled access.** Shared logins and unclear responsibilities make it possible for the wrong person to change prices, stock, or orders.

BR-022. **Disconnected processes.** Purchasing, receiving, sales, and stock adjustments happen in separate places, so inventory, orders, and reports disagree.

---

## 3. Business Opportunity

A controlled inventory system can improve the business in the following ways.

BR-030. **Inventory accuracy.** Physical, sellable, reserved, and available stock become consistent with receiving, sales, transfers, returns, and adjustments, with damaged and expired quantities kept out of available-to-promise stock.

BR-031. **Operational efficiency.** Staff spend less time reconciling spreadsheets and more time receiving, picking, selling, and investigating exceptions.

BR-032. **Purchasing decisions.** Buyers can use stock, demand, supplier history, and reorder guidance instead of urgent guesswork.

BR-033. **Sales fulfillment.** Sales staff can promise only what is available, reserve it, and complete sales without accidental oversell.

BR-034. **Warehouse management.** Each warehouse has its own stock picture, transfers are visible in transit, and location shortages are obvious.

BR-035. **Management visibility.** Owners and managers can see inventory value, movement, risk, and pending approvals without waiting for a manual report.

BR-036. **Decision-making.** People and AI can explain what is happening using the same operational facts, then propose actions that still require the right approval.

BR-037. **Trust and accountability.** Audit history makes inventory a governed record rather than an informal estimate.

---

## 4. Product Vision

The long-term vision is to be the trusted operational system of record for SMB inventory: accurate stock, controlled people and permissions, complete movement history, and an AI copilot that helps the business see risk and act—without ever becoming an unsupervised actor.

The product should remain an inventory operations platform. It should not become a full accounting, manufacturing, e-commerce, or logistics suite. Adjacent capabilities such as commercial invoices and payment recording exist only to support inventory commerce, not to replace finance systems.

Over time, the product may deepen forecasting, warehouse operations, scanning, mobile use, and integrations (see Section 51). Those expansions must preserve isolation, integrity, explainable AI, and human control of high-impact actions.

---

## 5. Business Objectives

Objectives below are directional and measurable. Exact numeric targets may be set per customer after go-live because baseline quality differs by business.

| ID | Objective | Why it matters |
|----|-----------|----------------|
| BR-040 | Improve inventory accuracy so recorded stock matches physical stock within an organization-defined tolerance. | Accuracy is the foundation of selling, buying, and AI advice. |
| BR-041 | Reduce stockouts on active, in-demand products. | Stockouts lose sales and damage customer trust. |
| BR-042 | Reduce excess and dead inventory. | Excess inventory consumes cash and space. |
| BR-043 | Improve purchasing efficiency (cycle time, fewer emergency buys, better-supported quantities). | Purchasing should be planned, approved, and received against. |
| BR-044 | Improve warehouse visibility across all active locations. | Multi-warehouse businesses cannot manage what they cannot see. |
| BR-045 | Reduce manual reporting effort for recurring inventory, sales, and purchase questions. | Management time should go to decisions, not data assembly. |
| BR-046 | Provide trustworthy AI-assisted decision support that users can explain and audit. | AI is valuable only if it is grounded, permission-aware, and controlled. |
| BR-047 | Make every stock change traceable to a business event and an actor. | Traceability is required to correct errors and satisfy accountability. |
| BR-048 | Enforce role-based responsibilities so sensitive actions are limited to the right people. | Unrestricted access is an operational and security failure. |
| BR-049 | Support multiple independent organizations without cross-organization visibility. | Isolation is a core market and trust requirement. |

---

## 6. Success Criteria

Success is not “screens exist.” Success is adoption of controlled inventory operations.

| ID | Success indicator | Business meaning |
|----|-------------------|------------------|
| BR-050 | Organizations complete onboarding: organization, users, warehouses, products, and opening stock. | The system can become the stock record. |
| BR-051 | Day-to-day receiving, sales, transfers, and adjustments are recorded in the system rather than outside it. | The ledger stays current. |
| BR-052 | Available stock is used at sale confirmation so oversell is prevented under the chosen stock policy. | Fulfillment promises are honest. |
| BR-053 | Purchase orders follow draft → approval → receive rather than informal buying with after-the-fact notes. | Procurement is controlled. |
| BR-054 | Managers obtain inventory, low-stock, movement, and valuation views without building spreadsheets. | Reporting replaces manual assembly. |
| BR-055 | Audit history can explain important stock, order, approval, and AI actions. | Accountability is real. |
| BR-056 | Users ask the AI copilot operational questions and receive answers consistent with authorized data, labelled by type (fact, calculation, prediction, recommendation). | AI is used and trusted. |
| BR-057 | High-impact AI drafts are approved or rejected by authorized people; rejected drafts do not execute. | Human control is proven. |
| BR-058 | No user of Organization A can access Organization B operational data. | Multi-tenancy holds. |
| BR-059 | Roles cannot perform actions outside their responsibilities (for example, VIEWER cannot adjust stock). | Authorization holds. |
| BR-060 | Inventory accuracy and stockout/overstock KPIs are reviewable after a meaningful operating period. | Outcomes can be managed. |

---

## 7. Target Users

The initial market is SMB organizations that:

- Hold physical inventory in one or more warehouses or stock locations.
- Purchase from suppliers and sell to customers.
- Need more control than spreadsheets, but do not need a full ERP, MRP, or accounting suite.
- Want optional AI assistance with explicit human control.

Typical sectors include wholesale, distribution, retail back-office, spare parts, and light trading businesses. Highly regulated pharmacy-grade serialization, bonded warehousing, and manufacturing production control are not the initial target.

---

## 8. Stakeholders

| Stakeholder | Interest |
|-------------|----------|
| Business owner | Cash tied in stock, stockouts, profitability, trust in numbers, and control of who can change them. |
| Organization administrator | Users, roles, organization settings, and safe onboarding. |
| Inventory manager | Accuracy, movements, adjustments, aging, dead stock, and warehouse balance. |
| Warehouse staff | Receiving, put-away visibility, transfers, counts, damage, and expiry handling. |
| Sales staff | Availability, reservations, order completion, customer history, returns. |
| Procurement staff | Suppliers, purchase orders, approvals, receiving discrepancies, purchase returns. |
| Management | Reports, KPIs, briefings, pending approvals, and exception alerts. |
| System / platform administrator (SUPER_ADMIN) | Platform health, organization lifecycle, and support without casual browsing of tenant operations. |
| Customers (external) | Reliable fulfillment; they do not use the system as operators. |
| Suppliers (external) | Accurate orders and returns; they do not use the system as operators in the initial product. |

---

## 9. User Personas

### 9.1 Priya — Business owner / managing director

- **Responsibilities:** Overall performance, cash, customer reputation, and who is trusted with control.
- **Goals:** Know whether stock is healthy; avoid surprises; approve important buys; understand AI advice without becoming an operator of every transaction.
- **Pain points:** Spreadsheet arguments; discovering stockouts from customers; not knowing which warehouse has goods.
- **Typical tasks:** Review daily briefing, low-stock, inventory value, pending approvals, and exception alerts.
- **Information needs:** KPIs, risks, pending actions, and explanations that separate facts from recommendations.

### 9.2 Daniel — Organization administrator

- **Responsibilities:** Users, roles, organization profile, and operational configuration such as warehouses and units.
- **Goals:** The right people have the right access; leavers are deactivated; settings stay consistent.
- **Pain points:** Shared passwords; leftover access; unclear ownership of master data.
- **Typical tasks:** Invite or create users, assign roles, deactivate accounts, maintain organization settings.
- **Information needs:** User list, role assignments, active/inactive status, recent admin activity.

### 9.3 Amina — Inventory manager

- **Responsibilities:** Stock integrity across warehouses, adjustments, transfers, aging, and inventory policy (reorder levels).
- **Goals:** Traceable stock, timely transfers, few unexplained variances.
- **Pain points:** Silent quantity edits; receiving that never matches POs; dead stock nobody owns.
- **Typical tasks:** Review movements, approve or perform adjustments, plan transfers, investigate anomalies.
- **Information needs:** Physical / sellable / reserved / available / damaged / expired, movements, aging, dead stock, adjustment reasons.

### 9.4 Luis — Warehouse / inventory staff

- **Responsibilities:** Receive goods, note damage and shorts, pick for sales or transfers, record obvious damage or expiry.
- **Goals:** Fast, accurate receiving and dispatch without accidentally changing another warehouse’s stock.
- **Pain points:** Paper PO copies; no visibility of expected receipts; unclear where stock should go.
- **Typical tasks:** Receive against PO, record discrepancies, dispatch/receive transfers, flag damaged goods.
- **Information needs:** Expected receipts, source/destination transfer lines, product identity, quantities, warehouse assignment.

### 9.5 Sofia — Sales staff

- **Responsibilities:** Create and complete sales, honor availability, handle customer returns at a commercial level.
- **Goals:** Promise only what can be sold; complete orders quickly; see customer history.
- **Pain points:** Selling stock another colleague just sold; no reservation; unclear return restock rules.
- **Typical tasks:** Check availability, create sales, complete fulfillment, initiate customer returns.
- **Information needs:** Available stock by warehouse, customer history, order status, selling prices (not necessarily cost).

### 9.6 Kenji — Procurement staff

- **Responsibilities:** Supplier records, purchase orders, chasing receipts, purchase returns.
- **Goals:** Right supplier, right quantity, timely approval and receipt.
- **Pain points:** Reordering from memory; missing partial receipts; weak supplier history.
- **Typical tasks:** Create POs, submit for approval, monitor receiving, raise supplier returns.
- **Information needs:** Supplier performance, prices, lead-time history, on-hand and inbound, reorder suggestions.

### 9.7 Maya — Viewer / analyst (read-only)

- **Responsibilities:** Observe operations and reports without changing them.
- **Goals:** Understand performance without risk of accidental edits.
- **Pain points:** Being given admin access “just to look.”
- **Typical tasks:** Read reports, export or review summaries if permitted, read AI explanations.
- **Information needs:** Reports and dashboards appropriate to granted visibility (cost visibility is not assumed).

---

## 10. Scope

The initial product includes the following business capabilities.

| ID | Capability |
|----|------------|
| BR-070 | Multi-organization tenancy with organization-level isolation. |
| BR-071 | User management, roles, and permissioned access. |
| BR-072 | Product catalog: products, SKUs, categories, brands, units, status, and supporting metadata. |
| BR-073 | Product images or documents where they help identify goods. |
| BR-074 | Multiple warehouses and warehouse-specific inventory. |
| BR-075 | Inventory quantities: physical, sellable, reserved, available, damaged, and expired stock; opening stock; movements; adjustments; transfers; history. |
| BR-076 | Low-stock and reorder thresholds; stock aging and dead-stock views. |
| BR-077 | Inventory valuation as a management view using Weighted Average Cost (DEC-006). |
| BR-078 | Supplier profiles, contacts, product/price information, performance, and history. |
| BR-079 | Purchase orders, approval, partial and full receiving, purchase history, and purchase returns. |
| BR-080 | Customer profiles, contacts, and sales-related history. |
| BR-081 | Sales orders, availability checks, reservation, completion, inventory deduction, and sales history. |
| BR-082 | Commercial sales invoices as operational documents (not a general ledger). |
| BR-083 | Limited payment recording against sales and purchases (status and amounts), not banking. |
| BR-084 | Customer and supplier returns with reasons, quantities, inventory impact, and history. |
| BR-085 | Warehouse transfers with dispatch, receipt, and history. |
| BR-086 | Operational reports with filtering and time-based analysis. |
| BR-087 | Business notifications for stock, orders, approvals, transfers, AI alerts, and important system events. |
| BR-088 | Audit history for important business and AI actions. |
| BR-089 | AI Inventory Copilot, demand/stock analysis, reorder and supplier recommendations, anomaly detection, briefings, summaries, and draft actions with human approval. |

---

## 11. Out of Scope

The following are **out of scope** for the initial product unless later explicitly approved.

| ID | Exclusion | Rationale |
|----|-----------|-----------|
| BR-090 | Full accounting system and general ledger | This is an inventory operations product. |
| BR-091 | Payroll | Unrelated to inventory operations. |
| BR-092 | Manufacturing, MRP, and production planning | Different operational domain. |
| BR-093 | Full logistics, fleet, last-mile, or carrier management | Transfers are warehouse-to-warehouse stock movements, not a TMS. |
| BR-094 | E-commerce storefront and public shopping carts | Sales are operational orders, not a store. |
| BR-095 | CRM beyond inventory-related customer records | Customers exist to support sales, returns, and history. |
| BR-096 | Banking system and direct bank integrations | Payments are recorded, not settled through banks. |
| BR-097 | Complex tax determination and statutory tax/accounting compliance | Optional tax amounts may be recorded; compliance engines are excluded. |
| BR-098 | Cryptocurrency and payment settlement systems | Out of domain. |
| BR-099 | Accounts payable/receivable accounting, cash management, and bank reconciliation | Would turn the product into finance software. |
| BR-100 | Shop-floor production, bill of materials explosion, and work orders | Manufacturing. |
| BR-101 | External supplier portal and customer self-service portal | Future consideration. |
| BR-102 | Full barcode/hardware operations and mobile-first warehouse app | See future scope. |
| BR-103 | Unrestricted AI database access or AI execution of arbitrary operational commands | Conflicts with trust and control. |

PAY-001. Payment recording associated with sales or purchases **may** be included.

PAY-002. Payment recording must not expand into a full accounting platform.

---

## 12. Organizations / Tenants

ORG-001. The product must support many independent organizations (tenants) on the same product.

ORG-002. Each organization’s operational data is private to that organization. No organization may read, change, delete, or infer another organization’s sensitive operational information.

ORG-003. Qualified users may create an organization through initial onboarding and become its initial ADMIN. SUPER_ADMIN platform administration may also create organizations. There is no complex organization approval workflow in the initial product (DEC-032). An organization maintains a business profile (name, status, contact details, and similar non-technical settings).

ORG-004. Organization settings include a single base currency (DEC-007), locale/timezone for business dates, and other inventory-related defaults. The initial product does not support multi-currency accounting or multi-currency inventory valuation.

ORG-005. Users operate inside an organization membership context. The organization in use is determined by the authenticated user’s membership, not by a value the user can freely assert to reach another tenant.

ORG-006. Organization administrators can manage that organization’s users, roles, warehouses, and catalog configuration according to their permissions.

ORG-007. An organization can be deactivated so that it is no longer used operationally, without silently destroying audit history.

ORG-008. Organization-level isolation applies to catalog, inventory, partners, documents, reports, notifications, AI context, and audit records.

ORG-009. Platform-level SUPER_ADMIN duties are distinct from organization operational duties (DEC-012).

---

## 13. User Management

USR-001. Authorized administrators can create users for their organization.

USR-002. Users have a profile with identity and contact information needed for work and notifications.

USR-003. Users can be activated and deactivated. Deactivated users cannot perform operational actions.

USR-004. Each organizational user has one or more assigned roles. Initial product assumption: one primary role per user unless later decided otherwise (ASM-004).

USR-005. Organization membership is required for business users. A person does not gain access to an organization only by knowing its name.

USR-006. Users cannot assign themselves higher privilege than they are allowed to manage.

USR-007. Credentials and session access are required to use the system. Shared generic logins are discouraged by design of named users.

USR-008. A user’s effective permissions are the intersection of authentication, organization membership, role, and any warehouse restrictions that apply.

USR-009. User deactivation does not erase that user’s historical actions from audit history.

USR-010. Initial membership model: one business user belongs to one organization (DEC-013).

---

## 14. Roles and Permissions

PERM-001. Authorization is mandatory. The interface may hide actions, but hiding is not the security boundary.

PERM-002. Least privilege: a role receives only the capabilities needed for its responsibilities.

PERM-003. Sensitive capabilities include user management, role changes, stock adjustments, purchase approval, sales cancellation of committed orders, transfers of stock, price/cost visibility, and AI action approval.

PERM-004. VIEWER is read-oriented and must not change operational stock or documents.

### 14.1 SUPER_ADMIN

ROLE-001. **SUPER_ADMIN** is a platform role, not a normal company operating role.

Responsibilities:

- Manage platform-level administration and organization lifecycle support.
- Assist with system-wide configuration that is not tenant operations.
- When operating in an organization context, access all warehouses across that organization (DEC-003). Broader tenant operational duty remains subject to DEC-012.

Limits:

- Not a substitute for an organization ADMIN.
- Must not casually operate a tenant’s purchasing, sales, or stock as if they were an organization manager.
- Any exceptional access to tenant operational data beyond confirmed warehouse-access rules, if ever permitted, is a controlled support act and must be audited (DEC-012).

### 14.2 ADMIN

ROLE-002. **ADMIN** is the organization administrator.

Responsibilities:

- Manage organization settings, users, roles, and master data administration.
- Oversee warehouses, catalog, and high-impact operational controls.
- Approve purchases and warehouse transfers, and review AI action approvals.
- Access all warehouses within the organization (DEC-003).
- View inventory cost information (DEC-023).
- Review audit history.

Limits:

- Still confined to their organization.
- Should follow inventory procedures rather than silently editing quantities.

### 14.3 MANAGER

ROLE-003. **MANAGER** is an operational and commercial manager.

Responsibilities:

- Oversee inventory health, purchasing, sales performance, and warehouse balance.
- Approve purchase orders, warehouse transfers, adjustments, and AI high-impact actions.
- Access all warehouses within the organization (DEC-003).
- View inventory cost information (DEC-023).
- Use reports, KPIs, and AI briefings to direct staff.

Limits:

- Does not replace platform SUPER_ADMIN.
- Still confined to their organization.

### 14.4 INVENTORY_STAFF

ROLE-004. **INVENTORY_STAFF** execute warehouse and stock operations.

Responsibilities:

- Receive goods against purchase orders.
- Record receiving discrepancies, damage, and expiry events according to procedure.
- Draft transfers and record dispatch/receipt of transfers when permitted.
- Participate in adjustments when authorized; they do not have unrestricted adjustment power by default.

Limits:

- No organization user administration.
- No unrestricted access to other organizations.
- Access only assigned warehouses (DEC-003). Warehouse restrictions are enforced by the backend, including inventory, transactions, reports, and other warehouse-scoped information.
- Cannot view inventory cost information by default (DEC-023).
- Do not approve purchase orders.

### 14.5 SALES_STAFF

ROLE-005. **SALES_STAFF** run sales and customer-facing inventory demand.

Responsibilities:

- Maintain customer records needed for sales.
- Create and complete sales using available stock.
- Initiate customer returns.

Limits:

- Cannot freely adjust inventory to make a sale possible.
- Cannot approve purchase orders.
- Access only assigned warehouses (DEC-003). Warehouse restrictions are enforced by the backend.
- Cannot view inventory cost information by default (DEC-023). AI tools apply the same restriction.
- Cannot access other organizations’ customers or stock.

### 14.6 VIEWER

ROLE-006. **VIEWER** is a read-only organizational role.

Responsibilities:

- View permitted operational information and reports for assigned warehouses only (DEC-003).
- Use AI copilot in read mode if permitted, with the same warehouse and cost restrictions.

Limits:

- Cannot create or change products, stock, orders, users, or approvals.
- Cannot approve AI actions that would change operations.
- Cannot view inventory cost information by default (DEC-023).

---

## 15. Product Management

PROD-001. An organization can maintain a product catalog of items it stocks, purchases, and/or sells.

PROD-002. Each product has a SKU that is unique within the organization.

PROD-003. Each product has a name, optional description, category, brand, unit of measure, status, and additional business metadata as needed (for example barcode value, notes).

PROD-004. Product status includes at least: **Draft**, **Active**, **Inactive**, **Archived**.

PROD-005. Draft products are not used on live purchase or sales documents.

PROD-006. Active products may be purchased, received, sold, transferred, and reported.

PROD-007. Inactive products cannot be added to new purchasing or sales transactions. They remain visible in historical transactions, inventory history, reports, and audit records. Historical data remains intact (DEC-033).

PROD-008. Archived products are removed from ordinary operational search while remaining historically referenced.

PROD-009. Products with historical transactions must not be casually deleted. The business retires them through inactive/archived status.

PROD-010. Selling and purchasing prices are business attributes of a product or of supplier-product relationships. They are not freely trusted from an unauthenticated source.

PROD-011. Product images and/or documents may be attached to help identification. Attachments are business records, not executable content.

PROD-012. Product lifecycle must support create, update, status change, and controlled retirement.

PROD-013. The inventory identity in the initial product is the SKU. There is no product-variant matrix. Items that need separate inventory tracking are separate products/SKUs (for example T-Shirt Blue Medium, T-Shirt Blue Large, and T-Shirt Red Medium). A parent-product/variant system is future scope (DEC-004).

PROD-014. Reorder level and low-stock threshold are maintained by authorized users. AI may recommend changes but must not silently overwrite them (DEC-031).

PROD-015. A product belongs to exactly one organization.

---

## 16. Categories

CAT-001. An organization can create and maintain product categories to group products for browsing, reporting, and AI explanation.

CAT-002. Category names must be unique within an organization at the same level of hierarchy used.

CAT-003. Hierarchical categories may be used if kept simple (parent/child). Deep taxonomy engines are not required.

CAT-004. Categories can be activated or deactivated without deleting historical product references.

CAT-005. Deleting a category that is in use must be prevented or require reassignment. Casual deletion is not allowed.

CAT-006. Categories are organization-scoped.

---

## 17. Brands

BRN-001. An organization can maintain brands to classify products.

BRN-002. Brand names are unique within an organization.

BRN-003. Brands can be active or inactive.

BRN-004. Brands in use on products cannot be casually deleted.

BRN-005. Brand is optional on a product if the organization does not use brands, but the capability must exist.

BRN-006. Brands are organization-scoped.

---

## 18. Units of Measurement

UOM-001. An organization can define units of measure used for stocking, purchasing, and selling (for example each, box, kilogram).

UOM-002. Each product has a stocking unit that is the unit of inventory quantity.

UOM-003. The initial product supports simple unit conversion using explicit, deterministic conversion factors (for example 1 Box = 12 Pieces). It does not provide a complex universal unit-of-measure engine (DEC-005).

UOM-004. Quantities on inventory records are expressed in the stocking unit.

UOM-005. Units in use cannot be casually deleted.

UOM-006. Units are organization-scoped unless a later decision introduces a shared standard list (not assumed).

---

## 19. Warehouse Management

WH-001. An organization may operate one or more warehouses (stock locations).

WH-002. Each warehouse has a profile (name, code, status, address/contact as needed).

WH-003. Warehouse status includes at least active and inactive. Inactive warehouses cannot receive new stock movements except as required to wind down remaining stock.

WH-004. Inventory is warehouse-specific. The same SKU may have different quantities in different warehouses.

WH-005. Warehouse-level access control is required (DEC-003). INVENTORY_STAFF, SALES_STAFF, and VIEWER are assigned to warehouses and may only use those warehouses. ADMIN and MANAGER access all warehouses in the organization. SUPER_ADMIN, when operating in an organization context, accesses all warehouses across that organization. These restrictions are enforced by the backend, not merely hidden in the frontend.

WH-006. Transfers are the business method for moving stock between warehouses. Editing one warehouse’s quantity to “move” goods without a transfer is not acceptable.

WH-007. Warehouses belong to one organization and are not shared across organizations.

WH-008. Opening stock is established per product per warehouse by authorized entry or import (DEC-026).

WH-009. Reporting, transactions, notifications, and AI answers that mention location must respect warehouse permissions. Operational users must not see warehouse-scoped information for warehouses to which they are not assigned.

WH-010. A default warehouse may be configured for operational convenience; it does not bypass availability or permissions.

---

## 20. Inventory Management

Inventory is a core business capability and the system of record for stock quantities.

### 20.1 Meaning of stock quantities

These definitions are confirmed (DEC-014) and are authoritative for this document.

INV-001. **Physical Stock** is the total physical quantity recorded at a warehouse/product level, including sellable and non-sellable inventory (stocking unit).

INV-002. **Reserved Stock** is sellable inventory committed for confirmed business transactions according to reservation rules (DEC-028). Reserved quantity is part of sellable quantity, not an extra physical quantity.

INV-003. **Sellable Stock** is physical inventory that is currently fit for normal sale (not damaged or expired).

INV-004. **Available Stock** is sellable inventory that is not reserved. It is the quantity that may still be promised for normal sales.

**Available Stock = Sellable Stock − Reserved Stock**

This is a business rule. Available Stock must not mean total physical inventory.

INV-005. Available stock must not be presented as a separately edited number. It is derived from sellable and reserved quantities.

INV-006. Physical, sellable, reserved, damaged, and expired quantities change only through recognized stock events (Section 21). They are not silently overwritten.

INV-007. Negative stock is not allowed. The system must prevent confirmation of transactions that would cause available or sellable stock to become negative (DEC-021).

Conceptual model:

```
Physical Stock
├── Sellable Stock
│   ├── Available Stock
│   └── Reserved Stock
├── Damaged Stock
└── Expired Stock
```

**Physical Stock = Sellable Stock + Damaged Stock + Expired Stock**  
**Sellable Stock = Available Stock + Reserved Stock**

### 20.2 Inventory views and planning attributes

INV-008. Users with permission can view inventory by product, by warehouse, and only across warehouses they are allowed to see (DEC-003).

INV-009. Opening stock may be entered or imported by authorized users. Opening-stock operations must validate the product and warehouse, validate quantities, create inventory movements, preserve audit history, and respect organization and warehouse permissions. Opening stock must not bypass inventory integrity rules (DEC-026).

INV-010. Authorized users maintain low-stock thresholds and reorder levels per product, and by warehouse where planning differs by location. AI may recommend changes but must not silently overwrite them (DEC-031).

INV-011. Low stock is interpreted against **available stock** so reserved demand remains visible. Damaged and expired stock do not count as available.

INV-012. Stock aging is the time stock has been held, used to identify slow-moving goods.

INV-013. Dead stock is stock with no meaningful movement over an organization-defined period, or otherwise identified as not expected to sell.

INV-014. Inventory valuation is a management view using **Weighted Average Cost**. The system maintains sufficient inventory cost information to calculate weighted-average cost correctly. FIFO, LIFO, and other complex accounting valuation methods are out of initial scope (DEC-006). Valuation is a calculation, not an AI guess. Cost figures are visible only to roles allowed by DEC-023.

INV-015. **Damaged Stock** is physical inventory that is not currently sellable because it is damaged. It must be identifiable, must not be available for normal sales, and must be moved with a damage-related inventory movement.

INV-016. **Expired Stock** is physical inventory that is not currently sellable because it has expired. It must be identifiable, must not be available for normal sales, and must be moved with an expiry-related inventory movement.

INV-017. Batch/lot tracking and expiry dates are supported in the initial product. Serial-number tracking is not. Batch and expiry information must be available where relevant to receiving, stock tracking, sales, returns, reporting, and AI analysis (DEC-018).

INV-018. Inventory history is the business-readable record of quantity changes and reasons, including transitions into or out of damaged or expired stock.

INV-019. Physical count / adjustment processes exist so recorded stock can be aligned to counted stock through an adjustment event, not a silent overwrite.

INV-020. Inbound purchased quantity that is not yet received is not physical stock and is not available stock. Purchase orders do not reserve sellable inventory (DEC-028).

INV-021. Staff must be able to explain, for any product/warehouse they are allowed to see: physical, sellable, reserved, available, damaged, and expired quantities; recent movements; batch/expiry where relevant; and whether goods are inbound.

---

## 21. Inventory Ledger Principle

INV-030. Every operation that changes stock must create a traceable business history.

INV-031. Unexplained or silent quantity change is not permitted.

INV-032. Inventory history must be sufficient to answer: “Why did the stock quantity change?”

Recognized movement types include:

| ID | Movement | Business meaning |
|----|----------|------------------|
| INV-033 | Opening | Initial physical/sellable (or classified) stock recorded at go-live or new warehouse/product introduction. |
| INV-034 | Purchase | Physical and sellable increase from goods received as good stock against purchasing. |
| INV-035 | Sale | Physical and sellable decrease from completed sale/fulfillment of reserved sellable stock. |
| INV-036 | Purchase return | Physical (and sellable or non-sellable, as applicable) decrease when a confirmed/dispatched supplier return is recorded. |
| INV-037 | Sales return | Physical increase when customer-returned goods are received; sellable increase only after inspection accepts them as sellable. |
| INV-038 | Adjustment in | Controlled increase to correct understated stock, classified as sellable or non-sellable as appropriate. |
| INV-039 | Adjustment out | Controlled decrease to correct overstated stock. |
| INV-040 | Transfer out | Decrease at source warehouse. |
| INV-041 | Transfer in | Increase at destination warehouse. |
| INV-042 | Damage | Quantity moved from sellable to damaged (non-sellable) stock, or otherwise identified as damaged. |
| INV-043 | Expired | Quantity moved from sellable to expired (non-sellable) stock, or otherwise identified as expired. |

INV-044. Adjustments require a reason and an authorized actor.

INV-045. Transfers produce paired business history at source and destination; in-transit stock must not vanish.

INV-046. Inventory history is auditable and is not editable as ordinary data by business users.

INV-047. Inventory movements must clearly identify transitions involving damaged or expired stock (DEC-014).

---

## 22. Supplier Management

SUP-001. An organization can maintain supplier profiles (identity, status, contacts, addresses, notes).

SUP-002. Supplier codes are unique within an organization. They are not globally unique across organizations. Cross-organization identifiers must never create data leakage or conflicts (DEC-027).

SUP-003. Suppliers can be active or inactive. Inactive suppliers cannot be used on new purchase orders.

SUP-004. Contacts can be stored for operational communication.

SUP-005. The organization can associate products with suppliers, including supplier-specific identifiers and prices when known.

SUP-006. Supplier pricing is historical and current commercial information. Missing prices must not be invented.

SUP-007. Supplier performance is derived from available history such as receipt completeness, discrepancy frequency, lead times, and fill of purchase orders. Performance is not assumed if history is missing.

SUP-008. Supplier history includes purchase orders, receipts, returns, and related comments/reasons.

SUP-009. Suppliers are organization-scoped.

SUP-010. Casual deletion of suppliers with history is not allowed.

---

## 23. Purchasing and Procurement

PUR-001. Authorized users can create purchase orders for an organization.

PUR-002. A purchase order has a supplier, expected location (warehouse), lines (product, quantity, commercial price), and status.

PUR-003. Purchase order line quantities are the quantities expected to be received, expressed in a defined unit and convertible to stocking units when conversion applies.

### 23.1 Purchase order lifecycle

High-level states for the initial product:

| State | Meaning |
|-------|---------|
| Draft | Being prepared; not submitted for approval; does not affect stock. |
| Pending approval | Submitted; waiting for an authorized approver. |
| Approved | Authorized to be sent/received against; still not stock on hand. |
| Partially received | Some, but not all, ordered quantity has been received. |
| Received | Ordered quantity has been fully received for practical completion. |
| Cancelled | Will not be fulfilled; no further receipts. |

PUR-004. Additional states must not be treated as confirmed requirements unless listed here or in assumptions.

PUR-005. Draft purchase orders do not increase stock and do not commit the organization until submitted.

PUR-006. Submitted purchase orders require approval before they proceed to the next operational stage. ADMIN and MANAGER can approve. The initial product does not use approval matrices or configurable monetary thresholds (DEC-009).

PUR-007. Only authorized roles may approve. Approvers should not be able to silently approve beyond their authority.

PUR-008. Partial receiving is required. Remaining open quantity stays receivable until received or the order is closed/cancelled according to procedure.

PUR-009. Full receiving marks the order Received when remaining quantity is zero (or remaining quantity is cancelled).

PUR-010. Cancelled orders cannot be received against.

PUR-011. Supplier selection uses supplier master data and, when available, price and performance information.

PUR-012. Purchase history must be retained.

PUR-013. Purchase returns are a distinct process (Section 27), not a silent negative receipt.

PUR-014. AI may draft a purchase order; it does not become Approved without the normal approval path (AI-assisted actions).

PUR-015. Commercial totals on a purchase order are calculations from lines and optional recorded tax rate/amount fields (DEC-008), not AI estimates. There is no jurisdictional tax engine.

---

## 24. Goods Receiving

REC-001. Goods are received against an approved (or otherwise receivable) purchase order, not as an anonymous stock increase.

REC-002. Receivers record received quantity per line.

REC-003. Partial receipts are allowed.

REC-004. Quantity verification is a business step: received quantity may differ from ordered quantity.

REC-005. Short receipts leave remaining quantity open or are explained.

REC-006. Damaged goods at receipt must be identifiable as damaged stock and must not be put into sellable or available stock as if they were good.

REC-007. Accepted good quantity updates physical and sellable stock at the receiving warehouse through a purchase movement. Batch/lot and expiry information is recorded where relevant (DEC-018).

REC-008. Receiving is not complete until the business records the receipt. Printing or intending to receive does not change stock.

REC-009. Receiving history is retained and auditable.

REC-010. INVENTORY_STAFF may only receive into assigned warehouses. ADMIN and MANAGER may receive into any warehouse in the organization (DEC-003).

REC-011. Over-receiving beyond ordered quantity is not assumed to be freely allowed. If permitted, it is an exception requiring authorization (ASM-007).

---

## 25. Customer Management

CUS-001. An organization can maintain customer profiles needed for sales and returns.

CUS-002. Customer codes are unique within an organization. They are not globally unique across organizations. Cross-organization identifiers must never create data leakage or conflicts (DEC-027).

CUS-003. Contact information and status (active/inactive) are maintained.

CUS-004. Inactive customers cannot be used on new sales.

CUS-005. Customer history includes sales, returns, and limited payment status against those documents.

CUS-006. Customers are organization-scoped.

CUS-007. This is not a full CRM (marketing campaigns, ticketing, opportunity pipelines are out of scope).

CUS-008. Casual deletion of customers with history is not allowed.

---

## 26. Sales

SAL-001. Authorized users can create sales orders for customers (or equivalent counter-sale documents) within their organization.

SAL-002. A sales order has customer, warehouse (fulfillment location), lines (product, quantity, selling price), and status.

### 26.1 Sales order lifecycle

High-level states for the initial product:

| State | Meaning |
|-------|---------|
| Draft | Being prepared; does not reserve stock. |
| Confirmed | Accepted operationally; reserves stock. |
| Completed | Goods issued/fulfilled; stock deducted for fulfilled quantity. |
| Cancelled | Will not be fulfilled; reservations released. |

SAL-003. Partial fulfillment may be allowed (DEC-015 remains open). If allowed, remaining reserved quantity stays committed until completed or cancelled. A distinct “partially fulfilled” label may be shown as a view of Completed/Confirmed progress; it is not an extra confirmed core state unless later added.

SAL-004. Draft sales do not reserve stock (DEC-028).

SAL-005. Confirmation requires sufficient **available stock** at the fulfillment warehouse. Negative available or sellable stock is not allowed (DEC-021). Backorders are not supported; if sufficient available stock does not exist, the sale cannot be confirmed (DEC-022).

SAL-006. Confirmation reserves sellable stock, increasing Reserved Stock and decreasing Available Stock without changing Physical Stock or Sellable Stock.

SAL-007. Completion/fulfillment deducts Physical Stock and Sellable Stock, releases the corresponding reservation, and records a sale movement. Batch/lot and expiry information is used where relevant (DEC-018).

SAL-008. Normal sales do not require manual approval. Cancellation of a confirmed order releases reservation without a sale movement. Cancellations, exceptional overrides, and other high-impact operational exceptions require MANAGER or ADMIN authorization. There is no complex configurable sales approval workflow in the initial product (DEC-019).

SAL-009. Sales staff must see availability for assigned warehouses before confirming.

SAL-010. When multiple users attempt to consume the same available inventory concurrently, availability must be revalidated before confirmation. Only transactions with sufficient available stock succeed. Competing transactions fail gracefully when stock is no longer available. Race-condition overselling is not allowed (DEC-034).

SAL-011. Sales history is retained.

SAL-012. Commercial invoices may be produced as operational records of a sale (DEC-016). Invoices do not create a general ledger.

SAL-013. Selling prices on documents are controlled business data. Users cannot arbitrarily overwrite protected pricing if the organization later locks prices; initially, authorized users may set line prices with audit of significant changes (ASM-008).

SAL-014. Sales do not bypass warehouse permissions. SALES_STAFF may only sell from assigned warehouses (DEC-003).

SAL-015. AI may draft a sale-related action only if later in scope; high-impact sales changes still follow permissions. Initial AI write drafts focus on purchasing, adjustments, and transfers (Section 39).

---

## 27. Returns Management

RET-001. The organization can process **customer returns** (sales returns) and **supplier returns** (purchase returns).

RET-002. Every return has a reason, quantities, source document when applicable, and status.

RET-003. Draft returns do not change stock.

RET-004. Customer returns do not automatically become sellable stock. Returned goods must go through inspection. After inspection, acceptable goods may return to sellable inventory; damaged goods remain non-sellable; expired goods remain non-sellable where applicable. All resulting inventory changes are recorded and auditable (DEC-030).

RET-005. Purchase-return stock impact occurs when the return is confirmed/dispatched according to the workflow. Draft purchase returns do not change inventory. All stock changes create traceable inventory movements (DEC-029).

RET-006. Return quantities cannot exceed the original eligible sold or received quantities in a way that creates unexplained stock. Over-returning is not allowed without an authorized exception (ASM-009).

RET-007. Returns create sales-return or purchase-return movements.

RET-008. Return history is retained and auditable.

RET-009. Credits or payment adjustments related to returns may be noted at a commercial level; they are not full accounting credit-note ledgers.

RET-010. Returns are organization-scoped and permissioned.

---

## 28. Warehouse Transfers

TRF-001. Stock moving between warehouses uses a transfer document, not two unrelated adjustments.

TRF-002. A transfer has source warehouse, destination warehouse, items, quantities, and status.

TRF-003. Source and destination must be different warehouses in the same organization.

### 28.1 Transfer lifecycle

High-level states:

| State | Meaning |
|-------|---------|
| Draft | Prepared; no stock effect. |
| Pending approval | Submitted; waiting for MANAGER or ADMIN approval (DEC-020). |
| Approved | Authorized; may reserve source sellable stock where required by the transfer workflow (DEC-028). |
| Dispatched | Left the source; in transit. |
| Received | Accepted at destination. |
| Cancelled | Will not occur; no remaining stock effect. |

TRF-004. Warehouse transfers require approval by an authorized MANAGER or ADMIN before final confirmation (DEC-020). Transfers must respect warehouse permissions and inventory availability rules. Negative available or sellable stock is not allowed.

TRF-005. Dispatch decreases source physical and sellable stock via Transfer out. If reservation was used after approval, reservation is consumed consistently (DEC-028).

TRF-006. While dispatched and not yet received, quantity is in transit: it is not available at either warehouse as ordinary sellable stock.

TRF-007. Receipt increases destination physical and sellable stock via Transfer in. Batch/lot and expiry information travels with the goods where relevant (DEC-018).

TRF-008. Partial dispatch or partial receipt is an assumption if needed (ASM-010). If not enabled, transfers are dispatched and received in full.

TRF-009. Transfer history is retained.

TRF-010. Users may only transfer from/to warehouses they are permitted to use (DEC-003).

TRF-011. AI may draft transfers; confirmation still requires human approval and normal controls (DEC-010, DEC-035).

---

## 29. Reporting

Reports are operational management views of existing business data. They do not invent transactions.

RPT-001. Inventory reports: physical, sellable, reserved, available, damaged, and expired quantities, by product and warehouse, including batch/expiry where relevant.

RPT-002. Stock movement reports: movements by type, product, warehouse, and time.

RPT-003. Purchase reports: orders, status, receipts, supplier, time.

RPT-004. Sales reports: orders, fulfillment, customer, product, time.

RPT-005. Supplier reports: volume, discrepancies, returns, performance based on available history.

RPT-006. Warehouse reports: stock position, inbound/outbound, transfers.

RPT-007. Product performance: sales and movement of products over time.

RPT-008. Inventory valuation reports using Weighted Average Cost (DEC-006), visible only to roles permitted to see cost (DEC-023).

RPT-009. Low-stock reports using thresholds.

RPT-010. Stock aging reports.

RPT-011. Dead-stock analysis.

RPT-012. Stockout analysis: products/periods where demand could not be filled or available stock reached zero.

RPT-013. Reports support filtering (organization context is implicit; warehouse, product, supplier, customer, status, date range as applicable). Warehouse-restricted roles see only assigned warehouses (DEC-003).

RPT-014. Reports support time-based analysis (periods, date ranges).

RPT-015. Users only see report data they are authorized to see, including warehouse and cost restrictions.

RPT-016. Report figures that are calculations must be produced from operational data, not estimated by AI. AI may explain a report (Section 38).

---

## 30. Notifications

NOT-001. Users receive notifications for events that require attention.

NOT-002. Low-stock alerts when a product/warehouse crosses its low-stock threshold.

NOT-003. Out-of-stock alerts when available stock reaches zero for tracked active products. Damaged and expired stock must not be treated as available.

NOT-004. Purchase order events: submitted, approved, rejected, received, cancelled, as relevant to the user.

NOT-005. Approval events for purchases, transfers, adjustments, and AI actions.

NOT-006. Transfer events: dispatched, received, cancelled, exceptions.

NOT-007. AI-generated alerts for anomalies, stockout risk, or briefing items, labelled as AI-generated and not as audited facts until verified.

NOT-008. Important system notifications (access changes, failed important actions visible to the actor).

NOT-009. Notifications respect organization isolation and permissions. A user must not be notified with another organization’s data.

NOT-010. Users should be able to understand what a notification refers to and navigate to the business record in the product experience.

---

## 31. Audit and Activity Tracking

AUD-001. Important business activities are traceable.

AUD-002. An audit record includes at least: organization, actor, action, entity type, entity identity, when it happened, and relevant before/after business context.

AUD-003. Audited activities include: sign-in and access-relevant events as appropriate; user creation and role changes; product changes; inventory adjustments and other stock-changing events; stock transfers; purchase changes and approvals; sales changes; returns; payment-status changes; important configuration changes; AI recommendations; AI-generated actions; human approval or rejection of AI actions.

AUD-004. Inventory movements are part of inventory history and are also auditable as important actions.

AUD-005. Ordinary business users cannot modify or delete audit history.

AUD-006. Audit history is organization-scoped for tenant events. Platform events for SUPER_ADMIN are distinguishable.

AUD-007. Audit must support investigation: who did what, when, and in what business context.

AUD-008. AI involvement must be visible (recommended vs approved vs executed).

---

## 32. AI Inventory Copilot

AI-001. The product includes an AI Inventory Copilot for authorized users.

AI-002. Users may ask natural-language operational questions, including:

- Which products are low in stock?
- Which products may stock out soon?
- How much stock do we have across all warehouses?
- Which warehouse has the most inventory?
- Which products are not moving?
- What should we reorder this week?
- Which supplier is best for this product?

AI-003. The copilot may only use information the user is authorized to see in the application.

AI-004. The copilot must respect organization isolation, roles, warehouse restrictions, and cost-visibility rules. AI tools apply the same permission rules as the application (DEC-003, DEC-023).

AI-005. The copilot must not invent products, quantities, orders, suppliers, prices, movements, or financial values.

AI-006. If required data is missing, the copilot must say so.

AI-007. The copilot is not a database administrator and must not run unconstrained data exploration beyond approved business capabilities.

AI-008. Answers must distinguish **FACT**, **CALCULATION**, **PREDICTION**, **RECOMMENDATION**, and **ACTION**.

AI-009. Predictions and recommendations must not be presented as facts.

---

## 33. Demand and Stock Analysis

AI-010. The system should support demand analysis using historical sales and related movement history when that history exists.

AI-011. Historical sales analysis is a calculation over recorded sales, not an AI estimate of missing history.

AI-012. Stockout risk is a prediction or recommendation based on available history, current available stock, inbound receipts, and thresholds. It is labelled as risk, not as certainty.

AI-013. Reorder recommendations may be produced from analysis (Section 34).

AI-014. Inventory trend analysis explains direction of stock and demand using available data.

AI-015. Analysis quality depends on history completeness (see dependencies and risks). The product must not pretend sparse data is a robust forecast.

---

## 34. Reorder Recommendations

AI-016. AI can recommend products to reorder, suggested quantities, timing, and reasoning.

AI-017. Recommendations must be explainable using supporting information such as current available stock, reserved demand, reorder level, recent sales, inbound purchase quantities, and supplier lead-time history when available.

AI-018. Suggested quantities that are numeric operational suggestions should be grounded in application calculations where possible. AI interprets and explains; it does not become the only source of arithmetic.

AI-019. AI must not silently overwrite user-maintained reorder points. Recommended reorder-point changes are drafts and follow the approval workflow (DEC-031, DEC-010).

AI-020. Accepting a reorder recommendation does not execute a purchase. The system creates a draft action; normal validation, authorization, and required approval follow (DEC-035).

---

## 35. Supplier Recommendations

AI-021. AI may compare suppliers using **available** business information: price, historical performance, lead time, reliability, and product availability from history.

AI-022. If a data element does not exist, AI must not assume it.

AI-023. “Best supplier” is a recommendation, not a fact.

AI-024. Supplier recommendations must stay within the organization’s suppliers and authorized data.

---

## 36. Inventory Anomaly Detection

AI-025. AI should help identify unusual patterns such as unexpected stock changes, unusual sales, unusual accumulation, possible data-entry errors, and abnormal product movement.

AI-026. Anomalies are signals for investigation, not automatic proof of fraud or error.

AI-027. Anomaly alerts must be reviewable against the underlying movements and documents.

AI-028. Anomaly detection must not itself change stock.

---

## 37. AI Daily Briefing

AI-029. Managers may receive a management-oriented briefing covering important items such as:

- Low-stock products
- Stockout risks
- Unusual inventory activity
- Sales trends
- Purchase activity
- Important pending actions (approvals, receipts, transfers)

AI-030. Briefing items must be traceable to authorized data.

AI-031. Briefings distinguish facts and calculations from predictions and recommendations.

AI-032. Briefings respect the recipient’s permissions.

---

## 38. AI-Generated Reports and Summaries

AI-033. AI may produce natural-language explanations and summaries of existing reports and records.

AI-034. Summaries must not add transactions or quantities that are not in the source data.

AI-035. Users should be able to open the underlying report or record.

---

## 39. AI-Assisted Actions

AI-036. AI may prepare drafts for purchase orders, inventory adjustments, stock transfers, and other operational actions that are explicitly allowed.

AI-037. Drafts use the same business validation as human-created drafts where applicable (products exist, warehouses exist, quantities are positive, stock policy applies).

AI-038. AI-generated actions must not bypass permissions, approvals, inventory ledger rules, or organization isolation.

AI-039. Execution happens only through normal application controls after any required approval.

AI-040. Drafts remain visible as AI-originated in history.

---

## 40. Human Approval for AI Actions

AI-041. Any AI-generated action that can materially change business data or operational state requires explicit human approval. Examples include creating or modifying purchase orders, changing reorder settings, creating inventory adjustments, confirming transfers, and other material write operations (DEC-010). AI may analyze data and generate recommendations automatically, but must not silently execute high-impact writes.

AI-042. Required business sequence (DEC-035):

1. AI analyzes authorized data.
2. AI produces a recommendation.
3. The user reviews the recommendation.
4. The user accepts or edits.
5. The system creates a draft action.
6. Normal application validation and authorization run.
7. Required approval is obtained.
8. The action executes through normal application controls.
9. The audit log records the recommendation, draft, approval, and execution.

AI recommendations remain distinguishable from confirmed business actions.

AI-043. Approval is given by an authorized human using their own credentials. The AI cannot approve itself.

AI-044. Before execution, current business state must be revalidated. Recommendations can become stale (stock may have been sold, prices changed, orders cancelled).

AI-045. Rejection leaves no operational stock or order effect beyond retaining the rejected proposal for audit.

AI-046. Read operations may run automatically when authorized. Write operations are stricter.

### 40.1 Statement types

| Type | Meaning |
|------|---------|
| FACT | A recorded business value (for example, on-hand quantity now). |
| CALCULATION | A deterministic result from recorded values (for example, available stock, totals, turnover). |
| PREDICTION | An uncertain future estimate (for example, likely days to stockout). |
| RECOMMENDATION | Advice (for example, reorder 40 units from Supplier A). |
| ACTION | A proposed or executed change to operational records. |

AI-047. The product must not treat the AI as an unrestricted operator of inventory records.

---

## AI Trust Requirements

AI-050. **Explainability.** Users can see why a recommendation or draft was produced, in business terms.

AI-051. **Traceability.** AI outputs and subsequent approvals are auditable.

AI-052. **Confidence.** Uncertain outputs are labelled; low-confidence advice is not presented as sure.

AI-053. **Data freshness.** AI responses and recommendations must communicate data freshness where relevant. AI must not present stale, cached, delayed, or historical data as real-time data. When analysis depends on non-current information, the relevant timestamp or freshness must be identified (DEC-024).

AI-054. **Permission awareness.** AI cannot be used to read or do what the user could not do in the application.

AI-055. **Human approval.** High-impact writes wait for a person.

AI-056. **Auditability.** Recommendations, drafts, approvals, rejections, and executions are recorded.

AI-057. **Revalidation.** Execution re-checks current stock, documents, and permissions.

AI-058. **Uncertain results.** Missing data, conflicts, or low confidence produce a safe refusal to invent.

AI-059. **Prompt and content distrust.** Product descriptions, notes, supplier names, and similar stored text are untrusted content and must not override permissions or system rules.

AI-060. Numeric operational figures (stock, revenue, profit, inventory value, reorder quantity arithmetic, tax/invoice totals) are business calculations whenever they can be calculated. AI explains them; it does not replace them.

---

## 41. Security Requirements

SEC-001. Users must authenticate before accessing non-public capabilities.

SEC-002. Every operational action is authorized according to role and context.

SEC-003. Role-based access is mandatory.

SEC-004. Organization isolation is mandatory.

SEC-005. Warehouse-level restrictions apply when the organization uses them.

SEC-006. Sensitive information (credentials, cost, personal contact data) is handled as sensitive business information.

SEC-007. Security-relevant events are auditable.

SEC-008. Users cannot obtain another organization’s data by manipulating identifiers, names, or exported references.

SEC-009. Deactivated users lose the ability to act.

SEC-010. AI tools, if any, are subject to the same authentication, organization context, and permissions as the user.

---

## 42. Multi-Tenancy

SEC-020. Organization data isolation is complete across catalog, stock, partners, documents, files, reports, notifications, AI context, and audit.

SEC-021. Users can only access authorized organization data.

SEC-022. Users can only perform permitted actions.

SEC-023. Warehouse restrictions are respected where applicable.

SEC-024. Identifier manipulation must not grant cross-organization access.

SEC-025. AI answers and drafts are tenant-bound.

SEC-026. Reports and KPIs never mix organizations.

---

## 43. Data Integrity

INT-001. Recorded inventory must match the sum of recognized movements from a known opening or prior position.

INT-002. Multi-step business operations (receive, sell, transfer, return) complete as one business transaction from the user’s point of view: they do not leave half-updated orders and stock.

INT-003. Stock changes are always traceable.

INT-004. There are no unexplained inventory changes.

INT-005. Order states are consistent with their effects (for example, draft does not reserve; cancelled confirmed sales release reservations; received POs reflect receipts).

INT-006. Approval states are consistent (unapproved POs are not receivable as approved stock intake).

INT-007. Concurrent users cannot both consume the same available quantity.

INT-008. Document totals, available stock, and reservations stay internally consistent.

---

## 44. Business KPIs

KPI definitions are business meanings. Formulas may be refined later, but they must remain calculations over operational data.

| ID | KPI | Business meaning |
|----|-----|------------------|
| KPI-001 | Inventory turnover | How many times inventory is sold or used relative to average inventory over a period. Higher generally means stock is moving; extremely high may signal understocking. |
| KPI-002 | Stockout rate | How often active products (or order lines) were unavailable when needed, over a period. |
| KPI-003 | Overstock rate | Share of stock or SKUs held materially above expected need or above a planning threshold. |
| KPI-004 | Dead stock value | Value of stock identified as dead or not moving over the defined period. |
| KPI-005 | Inventory accuracy | Agreement between recorded stock and physical counted stock, where counts exist. |
| KPI-006 | Order fulfillment rate | Share of sales demand fulfilled complete and on the intended basis, versus short or cancelled for stock reasons. |
| KPI-007 | Supplier performance | Composite view of timeliness, completeness, and discrepancy/return history where data exists. |
| KPI-008 | Purchase cycle time | Time from purchase draft/submit to approval and/or to completed receipt. |
| KPI-009 | Sales performance | Sales quantities and commercial amounts over a period, by product, customer, or warehouse as authorized. |
| KPI-010 | Warehouse Inventory Distribution | Business view of how inventory is distributed across warehouses, including relative stock position and inbound/outbound movement. Physical space-capacity utilization is not measured unless warehouse capacity data is explicitly introduced. |
| KPI-011 | Average inventory value | Typical value of on-hand inventory over a period using the chosen valuation method. |
| KPI-012 | Stock aging | Distribution of stock by age bands. |
| KPI-013 | AI recommendation acceptance rate | Share of AI recommendations that authorized users accept versus reject or ignore, used to judge usefulness—not to auto-execute. |

KPI-014. KPIs are organization-scoped and permission-filtered.

KPI-015. KPIs are calculations or defined operational counts. AI may explain them, not invent them.

---

## 45. Non-Functional Expectations

These are business expectations, not technology choices.

NFR-001. **Security.** Access is authenticated, authorized, and tenant-isolated.

NFR-002. **Reliability.** Core stock and order operations complete correctly or not at all.

NFR-003. **Availability.** The product is available for normal business hours and increasingly for always-on retail/wholesale operations; planned downtime should be communicated. Exact uptime targets are a later operational decision.

NFR-004. **Performance.** Common operational tasks (search products, view stock, confirm a sale, receive a PO) should feel responsive in ordinary SMB use.

NFR-005. **Scalability.** The product should support growth in organizations, users, products, and transactions without changing the business meaning of isolation and integrity.

NFR-006. **Maintainability.** Another operations team should be able to understand processes and responsibilities years later.

NFR-007. **Usability.** Staff can complete core workflows without being software specialists.

NFR-008. **Accessibility.** Interactive surfaces should be usable by people with reasonable assistive needs appropriate to a business application.

NFR-009. **Auditability.** Important actions can be reconstructed.

NFR-010. **Data integrity.** Inventory and documents remain consistent under concurrent use.

NFR-011. **Observability.** Operators can tell whether the product is healthy enough to trust (business-facing: failed jobs, missing notifications, AI unavailability).

NFR-012. **Backup and recovery.** The business expects that operational and audit records can be recovered after failure, with a recovery point and time appropriate to SMB operations (exact RPO/RTO later).

NFR-013. **AI safety.** AI unavailability must not block recording of purchases, receipts, sales, or stock adjustments.

---

## 46. End-to-End Workflows

Each workflow is described as business process only.

### WF-001 Organization onboarding

- **Objective:** Create an isolated organization ready to be configured.
- **Trigger:** A qualified user or platform administrator starts onboarding.
- **Actors:** Prospective ADMIN and/or SUPER_ADMIN.
- **Steps:** Provide organization identity; create the organization; establish the first ADMIN; set basic organization settings (currency/locale as decided).
- **Outcome:** A new organization exists with no other tenant’s data visible.
- **Exceptions:** Duplicate or incomplete identity; unauthorized creator.

### WF-002 User creation and role assignment

- **Objective:** Give a named person the right access.
- **Trigger:** ADMIN needs a new or changed user.
- **Actors:** ADMIN; the new user.
- **Steps:** Create user profile; assign role; optionally assign warehouses; activate; user authenticates.
- **Outcome:** The user can perform only permitted actions in that organization.
- **Exceptions:** Duplicate user; attempting to grant SUPER_ADMIN from inside a tenant; deactivating the last ADMIN.

### WF-003 Product creation

- **Objective:** Add an item to the catalog.
- **Trigger:** Need to stock, buy, or sell a new SKU.
- **Actors:** ADMIN, MANAGER, or other catalog-authorized role.
- **Steps:** Enter SKU, name, unit, optional category/brand, status Draft; complete data; set Active.
- **Outcome:** An active product exists for operational documents.
- **Exceptions:** Duplicate SKU; missing stocking unit; activating an incomplete product.

### WF-004 Warehouse creation

- **Objective:** Add a stock location.
- **Trigger:** New site or logical warehouse.
- **Actors:** ADMIN or MANAGER.
- **Steps:** Create warehouse profile; set active; optionally assign users.
- **Outcome:** Inventory can be held and reported at that warehouse.
- **Exceptions:** Duplicate code; creating a warehouse in the wrong organization (must be impossible).

### WF-005 Opening inventory

- **Objective:** Establish starting stock.
- **Trigger:** Go-live or first stocking of a product/warehouse.
- **Actors:** ADMIN, MANAGER, authorized inventory role.
- **Steps:** Enter opening quantities per product/warehouse; provide reason/reference; record opening movements.
- **Outcome:** Stock on hand exists and is traceable to opening.
- **Exceptions:** Opening that would create negatives; unauthorized actor; opening after movements already exist (must be controlled).

### WF-006 Purchase order creation

- **Objective:** Request goods from a supplier.
- **Trigger:** Reorder need, AI recommendation, or planned buy.
- **Actors:** Procurement-authorized user (ADMIN, MANAGER, or designated staff).
- **Steps:** Select supplier and warehouse; add lines; save Draft; submit to Pending approval.
- **Outcome:** A PO awaiting approval; no stock increase.
- **Exceptions:** Inactive supplier/product; empty lines; submitting without permission.

### WF-007 Purchase order approval

- **Objective:** Authorize a buy.
- **Trigger:** PO submitted.
- **Actors:** ADMIN or MANAGER (DEC-009).
- **Steps:** Review supplier, quantities, prices, warehouse; approve or reject.
- **Outcome:** Approved (receivable) or returned/cancelled per rejection practice.
- **Exceptions:** Approver lacks authority; document changed since submission; AI-drafted PO still follows this workflow.

### WF-008 Goods receiving

- **Objective:** Record what physically arrived.
- **Trigger:** Delivery against an approved PO.
- **Actors:** INVENTORY_STAFF, MANAGER.
- **Steps:** Select PO; record received quantities; record shorts/damage; confirm receipt.
- **Outcome:** Receipt history exists; PO becomes Partially received or Received.
- **Exceptions:** Wrong warehouse; over-receipt; receiving cancelled PO; damaged-only receipt.

### WF-009 Inventory update after receiving

- **Objective:** Increase on-hand for accepted good quantity.
- **Trigger:** Confirmed receipt of good stock.
- **Actors:** Same as receiving (system of record updates as part of the business receipt).
- **Steps:** Accepted quantity becomes purchase movement at the warehouse; receiving sellable stock increases sellable_quantity, while available stock is derived from sellable_quantity minus reserved_quantity (reserved unchanged).
- **Outcome:** Stock on hand, sellable stock, and derived available stock reflect the receipt.
- **Exceptions:** Attempt to add damaged goods as sellable; concurrent adjustment at same SKU/warehouse.

### WF-010 Sales order creation

- **Objective:** Capture customer demand.
- **Trigger:** Customer order or counter sale.
- **Actors:** SALES_STAFF, MANAGER, ADMIN.
- **Steps:** Select customer and warehouse; add lines; save Draft.
- **Outcome:** Draft sales document; no reservation.
- **Exceptions:** Inactive customer/product; sales into unauthorized warehouse.

### WF-011 Inventory reservation

- **Objective:** Commit available stock to a confirmed sale.
- **Trigger:** Confirmation of a sales order.
- **Actors:** SALES_STAFF or authorized confirmer.
- **Steps:** Check available stock; if sufficient, confirm; reserved increases; available decreases.
- **Outcome:** Stock is promised to the order.
- **Exceptions:** Insufficient available stock; concurrent confirmation; product inactive.

### WF-012 Sale completion

- **Objective:** Fulfill the order.
- **Trigger:** Goods issued to the customer.
- **Actors:** SALES_STAFF, INVENTORY_STAFF as applicable, MANAGER.
- **Steps:** Record fulfillment quantities; complete the order (or remaining quantity under DEC-015).
- **Outcome:** Order Completed (or remaining commitment updated).
- **Exceptions:** Fulfilling more than reserved; missing stock due to damage after reservation (requires exception handling).

### WF-013 Inventory deduction

- **Objective:** Reduce stock on hand for fulfilled goods.
- **Trigger:** Sale completion.
- **Actors:** Same business event as completion.
- **Steps:** Sale movement decreases on-hand; reservation for fulfilled quantity is released; available remains consistent with the formula.
- **Outcome:** Ledger explains the decrease as a sale.
- **Exceptions:** Partial fulfill; cancellation instead of fulfill.

### WF-014 Customer return

- **Objective:** Take back customer goods and update stock correctly.
- **Trigger:** Customer return request.
- **Actors:** SALES_STAFF, INVENTORY_STAFF for inspection.
- **Steps:** Create return with reason and quantities; inspect; restock sellable or record damage; confirm.
- **Outcome:** Sales-return history; stock updated per inspection (DEC-030).
- **Exceptions:** Quantity above original sale; goods not inspectable; unauthorized restock of damaged goods as sellable.

### WF-015 Supplier return

- **Objective:** Return goods to a supplier and reduce stock.
- **Trigger:** Defective, excess, or disagreed receipt.
- **Actors:** Procurement-authorized user; warehouse staff for dispatch.
- **Steps:** Create purchase return linked to supplier/receipt; confirm; dispatch; stock decreases (DEC-029).
- **Outcome:** Purchase-return movement and history.
- **Exceptions:** Returning more than received; draft treated as stock change.

### WF-016 Stock adjustment

- **Objective:** Align recorded stock to a known physical or write-off reality.
- **Trigger:** Count variance, damage, loss, found stock.
- **Actors:** MANAGER/ADMIN; INVENTORY_STAFF only if authorized.
- **Steps:** Propose adjustment with reason; approve if required; record adjustment in or out (or damage/expiry type).
- **Outcome:** Traceable quantity change; no silent overwrite.
- **Exceptions:** Adjustment that would go negative; AI draft without approval; missing reason.

### WF-017 Warehouse transfer

- **Objective:** Move stock between warehouses with in-transit visibility.
- **Trigger:** Rebalancing or fulfilling from another location.
- **Actors:** INVENTORY_STAFF draft; MANAGER/ADMIN approve (DEC-020); staff dispatch and receive.
- **Steps:** Draft lines; approve; dispatch from source; receive at destination.
- **Outcome:** Source decreased, destination increased, history paired; in-transit visible between dispatch and receipt.
- **Exceptions:** Same warehouse; insufficient source stock; receive before dispatch; cancelled in transit (requires explicit exception handling).

### WF-018 Low-stock detection

- **Objective:** Notify the business before stockouts.
- **Trigger:** Available or configured stock measure crosses the low-stock threshold after a stock event or review.
- **Actors:** System detection; inventory/procurement consumers of alerts.
- **Steps:** Evaluate thresholds; create alert; include product, warehouse, on-hand, reserved, available.
- **Outcome:** Authorized users are aware of low stock.
- **Exceptions:** Missing thresholds; inactive products; users without warehouse permission.

### WF-019 Reorder recommendation

- **Objective:** Suggest what to buy.
- **Trigger:** User request, briefing, or low-stock review.
- **Actors:** AI copilot; procurement/manager reviewer.
- **Steps:** Analyze authorized stock, demand, inbound POs, thresholds; produce labelled recommendation with reasons.
- **Outcome:** Explainable reorder advice; no PO approved yet.
- **Exceptions:** Insufficient history; missing supplier; permission-limited view.

### WF-020 AI recommendation

- **Objective:** Provide decision support without executing it.
- **Trigger:** User question or scheduled briefing.
- **Actors:** Authorized user; AI copilot.
- **Steps:** Interpret question; gather authorized facts/calculations; separate prediction/recommendation; present.
- **Outcome:** User understands data and advice types.
- **Exceptions:** Unauthorized question; missing data; attempt to jailbreak via stored notes.

### WF-021 AI action approval

- **Objective:** Convert a draft high-impact action into a real operation only with human control.
- **Trigger:** User accepts an AI draft for PO, adjustment, transfer, or similar.
- **Actors:** Authorized approver (not the AI).
- **Steps:** Review explanation and draft; approve or reject; on approve, revalidate current state; execute through normal controls; audit.
- **Outcome:** Either a normal business document/movement, or a recorded rejection.
- **Exceptions:** Stale stock; approver lacks permission; validation failure after approval; self-approval by AI.

### WF-022 Audit trail generation

- **Objective:** Preserve who did what.
- **Trigger:** Any important action in the workflows above.
- **Actors:** The acting user (or recorded system process for detections); audit record is not user-editable.
- **Steps:** Record actor, organization, action, entity, time, and context including AI involvement.
- **Outcome:** Investigators can reconstruct the event.
- **Exceptions:** Attempts to edit audit records fail; cross-tenant audit access denied.

---

## 47. Assumptions

Assumptions are not confirmed requirements. They exist so later phases do not treat them as decided fact.

| ID | Assumption |
|----|------------|
| ASM-001 | Initial customers are SMBs with one legal/operating organization per tenant. |
| ASM-002 | Goods are discrete stocked items identified primarily by SKU. |
| ASM-003 | Organizations operate primarily in a single currency. |
| ASM-004 | A user has one primary role per organization. |
| ASM-005 | External suppliers and customers do not log into the product in the initial scope. |
| ASM-006 | Email or in-product notification is sufficient; SMS/WhatsApp is not assumed. |
| ASM-007 | Over-receiving against a PO is exceptional, not a normal uncontrolled action. |
| ASM-008 | Authorized users may enter or override sales line prices; strict price lists are not assumed. |
| ASM-009 | Return quantities cannot exceed original eligible quantities. |
| ASM-010 | Transfers are full-quantity dispatch/receipt unless partial transfer is later confirmed. |
| ASM-011 | Physical cycle counts exist as a business practice; a full warehouse-management count module is not assumed beyond adjustments. |
| ASM-012 | Historical data for AI quality will be incomplete at go-live. |
| ASM-013 | Invoices are optional operational documents, not statutory e-invoicing. |
| ASM-014 | “Warehouse” includes any stock location the business treats as a warehouse. |
| ASM-015 | English is the initial product language. |
| ASM-016 | No backorders in the initial product (aligned with DEC-022). |
| ASM-017 | Purchase orders require approval for all submitted orders (aligned with DEC-009). |
| ASM-018 | Serial-number tracking is not required initially. |
| ASM-019 | The product is used by named employees, not anonymous shop-floor kiosks, in the initial scope. |
| ASM-020 | AI may be unavailable without stopping core inventory recording. |

---

## 48. Constraints

| ID | Constraint | Why it exists |
|----|------------|---------------|
| CON-001 | Multi-tenant isolation | Organizations will not use a product that leaks their stock, prices, or customers. |
| CON-002 | Inventory data integrity | Incorrect stock destroys fulfillment and purchasing. |
| CON-003 | Security and RBAC | Inventory and cost data are sensitive. |
| CON-004 | AI trust limits | AI hallucination and unauthorized writes are unacceptable in an inventory system of record. |
| CON-005 | Human approval for high-impact AI actions | The business remains accountable for stock and spending. |
| CON-006 | Traceable stock changes | Every quantity change needs a business reason. |
| CON-007 | Not a full accounting platform | Scope must remain inventory operations. |
| CON-008 | Scalability as a business constraint | The product must remain correct as organizations and volume grow. |
| CON-009 | Maintainability | Processes and rules must stay understandable. |
| CON-010 | Permission-aware AI | AI cannot be a back door around roles. |
| CON-011 | No silent oversell | Available stock is a promise boundary. |
| CON-012 | Audit immutability for business users | Accountability fails if history can be rewritten. |

---

## 49. Risks

| ID | Risk | Impact | Likelihood | Mitigation direction |
|----|------|--------|------------|----------------------|
| RSK-001 | Incorrect inventory data | Wrong sales, buys, and AI advice; lost trust. | Medium to high at go-live | Opening stock discipline, no silent edits, receiving against POs, adjustments with reasons, counts. |
| RSK-002 | AI hallucination | Users act on invented products or quantities. | Medium | Ground answers in authorized records; label types; refuse when data is missing. |
| RSK-003 | Incorrect recommendations | Overbuy, underbuy, or bad supplier choice. | Medium | Explainability, human approval, revalidation, calculations done as calculations. |
| RSK-004 | Unauthorized access | Fraud, data loss, incorrect stock. | Medium | Authentication, RBAC, least privilege, deactivation, audit. |
| RSK-005 | Data leakage between organizations | Severe trust and legal harm. | Low if isolation is treated as mandatory; high impact | Isolation as a non-negotiable rule; no identifier tricks; AI tenant binding. |
| RSK-006 | Concurrent inventory changes | Negative or double-sold stock. | Medium | Available-stock confirmation rule; only one winner for last units; complete business transactions. |
| RSK-007 | Poor adoption | Spreadsheets remain the real system; the ledger goes stale. | Medium | Usable core workflows, clear roles, fast receiving/sales, useful reports and alerts. |
| RSK-008 | Incomplete historical data | Weak forecasting and supplier advice. | High at launch | Honest AI about missing history; user-maintained reorder levels; improve over time. |
| RSK-009 | Incorrect demand predictions | Treated as facts; bad purchases. | Medium | Label predictions; keep human approval; do not auto-buy. |
| RSK-010 | Stale AI drafts executed | Adjustments or POs that no longer match stock. | Medium | Revalidation before execution; show freshness. |
| RSK-011 | Scope creep into accounting | Delayed inventory quality; confused users. | Medium | Hard out-of-scope list; limited payments. |
| RSK-012 | Incorrect damaged/expired classification | Available stock misstated if damaged or expired quantities are incorrectly treated as sellable. | Medium | Enforce the confirmed DEC-014 terminology and DEC-018 batch/expiry rules consistently across receiving, sales, returns, transfers, reporting, and AI. |
| RSK-013 | Shared logins | Audit becomes meaningless. | Medium | Named users; discourage sharing in administration practice. |
| RSK-014 | Over-privileged ADMIN | Internal abuse. | Medium | Audit; separate duties for day-to-day staff; SUPER_ADMIN ≠ tenant operator. |
| RSK-015 | Notification fatigue | Alerts ignored, including real stockouts. | Medium | Meaningful thresholds; role-targeted notices. |

---

## 50. Dependencies

| ID | Dependency | Why it matters |
|----|------------|----------------|
| DEP-001 | Accurate product catalog | All stock and documents hang on SKU identity. |
| DEP-002 | Accurate warehouse list | Location-level stock is otherwise meaningless. |
| DEP-003 | Accurate opening and ongoing inventory records | Availability, reports, and AI facts require them. |
| DEP-004 | Supplier information | Purchasing and supplier recommendations. |
| DEP-005 | Customer information | Sales and returns. |
| DEP-006 | User permissions correctly assigned | Security and correct AI answers. |
| DEP-007 | Historical sales and receipt data | Demand analysis, aging, supplier performance. |
| DEP-008 | Maintained reorder/low-stock thresholds | Alerts and reorder advice quality. |
| DEP-009 | External AI services where the product uses them | Copilot, briefings, and summaries may depend on a provider, but core inventory must not. |
| DEP-010 | Human approvers being available | POs, transfers, adjustments, and AI writes cannot complete without them. |
| DEP-011 | Organization settings (currency, timezone) | Reports and commercial amounts stay interpretable. |

---

## 51. Future Scope

The following are **not** initial requirements. They may be considered after the core product is successful.

| ID | Possible future capability |
|----|----------------------------|
| FUT-001 | Barcode scanning as a normal warehouse practice |
| FUT-002 | Mobile application for receiving, picking, and counts |
| FUT-003 | Advanced statistical or ML forecasting beyond initial trend/risk analysis |
| FUT-004 | Multi-echelon or constrained inventory optimization |
| FUT-005 | E-commerce storefront or marketplace integrations |
| FUT-006 | Accounting and general-ledger integrations |
| FUT-007 | Supplier EDI or supplier portal integrations |
| FUT-008 | Automated purchase execution below approved thresholds |
| FUT-009 | Advanced warehouse operations (bins, waves, directed put-away) |
| FUT-010 | Additional AI capabilities (scenario planning, richer briefings) |
| FUT-011 | Lot recall workflows and full serialization |
| FUT-012 | Multi-organization user membership |
| FUT-013 | Customer and supplier self-service portals |
| FUT-014 | Multi-language product |
| FUT-015 | Backorders and allocated ATP promising beyond simple available stock |
| FUT-016 | Landed cost and advanced costing |
| FUT-017 | Cycle-count programs with schedules and blind counts |

---

## 52. Open Business Decisions

Unresolved decisions are listed in full in [REQUIREMENT-DECISIONS.md](./REQUIREMENT-DECISIONS.md). Only decisions that are still marked **Open** are listed here; confirmed decisions are authoritative and are not repeated in this section.

| Decision ID | Topic |
|-------------|-------|
| DEC-001 | Commercial pricing model of the product |
| DEC-002 | Organization scale targets |
| DEC-011 | Data retention |
| DEC-012 | SUPER_ADMIN access to tenant operational data |
| DEC-013 | Multi-organization users |
| DEC-015 | Partial sales fulfillment |
| DEC-016 | Invoicing depth |
| DEC-017 | Payment recording depth |

Do not treat recommended defaults for these unresolved decisions as approved until a decision owner confirms them.

---

## Requirement quality notes

- Inventory meaning is consistent: **Available Stock = Sellable Stock − Reserved Stock**; Physical Stock includes sellable and non-sellable inventory.
- Multi-tenancy applies to all operational domains, reports, notifications, AI, and audit.
- Roles are least-privilege; VIEWER cannot change stock; AI cannot outrank roles.
- High-impact AI actions require human approval and revalidation, then normal controls.
- Payments and invoices remain commercial/operational, not a general ledger.
- Assumptions are labelled ASM-*. Open decisions are labelled DEC-* and live in both this BRD and `docs/REQUIREMENT-DECISIONS.md`.
- Future scope is labelled FUT-* and is not initial scope.
- This document does not prescribe programming languages, frameworks, databases, hosting, or interface libraries.

---

*End of Business Requirements Document. Next phase (functional requirements) should proceed only after this BRD is reviewed and approved.*
