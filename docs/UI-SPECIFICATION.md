# UI-SPECIFICATION.md

# Inventory Management AI — UI Specification

**Document Status:** Approved for implementation planning  
**Document Type:** User Interface Specification  
**Version:** 1.0  
**Date:** 2026-10-05

---

# 1. Purpose

This document defines the user-facing application experience for the Inventory Management AI system.

It translates the approved:

1. BRD
2. Requirement Decisions
3. FRD
4. Business Rules
5. Architecture
6. Database Design
7. API Specification

into a practical UI structure.

This document defines:

- application navigation
- screen hierarchy
- dashboards
- role-aware visibility
- page responsibilities
- tables
- forms
- filters
- detail pages
- workflow states
- approval interfaces
- inventory interfaces
- reporting interfaces
- notifications
- AI Copilot
- loading/error/empty states
- responsive behavior
- accessibility expectations

It does **not** define React/Next.js implementation code.

---

# 2. UI Principles

## UI-001 — Backend is authoritative

The UI must never be treated as the source of truth for:

- authorization
- inventory availability
- approval status
- business rules
- calculations
- AI action execution

---

## UI-002 — Show only what the user can act on

Role and warehouse permissions should shape:

- navigation
- buttons
- actions
- data visibility

However, backend authorization remains mandatory.

---

## UI-003 — Preserve operational context

Important screens should make these visible where relevant:

```text
Organization
Warehouse
Product
Status
Date/time
User
```

---

## UI-004 — Destructive/high-impact actions require clear confirmation

Examples:

- inventory adjustment
- transfer execution
- cancellation
- AI action approval
- important overrides

---

## UI-005 — Never hide important failures

Errors must be:

- visible
- understandable
- actionable
- safe

---

## UI-006 — Data freshness must be visible where important

Especially for:

- inventory
- dashboards
- AI recommendations
- anomaly detection
- reports

---

## UI-007 — AI must look assistive, not authoritative

The UI must distinguish:

```text
Fact
Calculation
Prediction
Recommendation
Proposed Action
```

---

# 3. Application Shell

The primary authenticated layout consists of:

```text
┌─────────────────────────────────────────────────────────────┐
│ Logo │ Organization │ Warehouse │ Search │ AI │ Alerts │ User│
├──────────────┬──────────────────────────────────────────────┤
│              │                                              │
│ Navigation   │              Main Content                    │
│              │                                              │
│ Dashboard    │                                              │
│ Inventory    │                                              │
│ Products     │                                              │
│ Purchasing   │                                              │
│ Sales        │                                              │
│ Suppliers    │                                              │
│ Customers    │                                              │
│ Transfers    │                                              │
│ Reports      │                                              │
│ Approvals    │                                              │
│ AI Copilot   │                                              │
│ Notifications│                                              │
│ Settings     │                                              │
│              │                                              │
└──────────────┴──────────────────────────────────────────────┘
```

---

# 4. Top Bar

The top bar contains:

## Organization selector

Shown when the user has access to more than one organization context.

Because DEC-013 remains unresolved, the UI should be designed so this control can be:

- hidden when only one organization exists
- enabled when multi-organization membership is introduced

---

## Warehouse selector

For warehouse-scoped roles:

```text
Current Warehouse
```

The selector must show only warehouses the user is authorized to access.

For ADMIN/MANAGER, an:

```text
All Warehouses
```

view may be available where the workflow supports it.

---

## Global search

Search may cover authorized:

- products
- suppliers
- customers
- purchase orders
- sales orders
- transfers

Results must respect organization and warehouse authorization.

---

## AI Copilot launcher

Persistent entry point:

```text
Ask AI
```

or an equivalent icon/button.

---

## Notifications

Shows:

- unread count
- important approvals
- low-stock notifications
- anomaly notifications
- report completion

---

## User menu

Contains:

- profile
- preferences
- organization context
- security/session options
- logout

Available settings depend on role.

---

# 5. Primary Navigation

Recommended navigation:

```text
Dashboard

Inventory
  ├── Overview
  ├── Stock
  ├── Movements
  ├── Adjustments
  └── Batches / Expiry

Catalog
  ├── Products
  ├── Categories
  ├── Brands
  └── Units

Purchasing
  ├── Purchase Orders
  ├── Receiving
  ├── Purchase Returns
  └── Suppliers

Sales
  ├── Sales Orders
  ├── Customers
  └── Sales Returns

Warehouses
  └── Transfers

Reports

Approvals

AI Copilot

Notifications

Settings
```

The actual navigation shown is role-aware.

---

# 6. Role-Aware Navigation

## SUPER_ADMIN

Platform-level navigation should focus on organization/platform administration.

Tenant operational-data access depends on unresolved DEC-012.

---

## ADMIN

Broad organization access:

- Dashboard
- Inventory
- Catalog
- Purchasing
- Sales
- Warehouses
- Reports
- Approvals
- AI
- Notifications
- Settings

---

## MANAGER

Operational management access:

- Dashboard
- Inventory
- Catalog
- Purchasing
- Sales
- Warehouses
- Reports
- Approvals
- AI
- Notifications

---

## INVENTORY_STAFF

Primary access:

- Dashboard
- Inventory
- Catalog
- Warehouses/Transfers where authorized
- relevant Purchasing/Receiving
- Notifications

Cost visibility remains restricted.

---

## SALES_STAFF

Primary access:

- Dashboard
- Products
- Inventory availability
- Sales
- Customers
- Sales Returns
- Notifications

Cost visibility remains restricted.

---

## VIEWER

Read-oriented access:

- Dashboard
- authorized inventory/catalog views
- reports allowed by role
- notifications

No mutation actions.

---

# 7. Responsive Design

The application must support:

- desktop
- tablet
- mobile

Desktop is the primary operational experience.

---

## Desktop

Use:

```text
persistent sidebar
+
full data tables
+
multi-column forms
```

---

## Tablet

Use:

```text
collapsible sidebar
+
responsive tables
+
stacked form sections
```

---

## Mobile

Use:

```text
drawer navigation
+
single-column forms
+
card/list representations
+
horizontal table scrolling where unavoidable
```

Do not remove essential information merely because the viewport is smaller.

---

# 8. Design System

Use:

- Tailwind CSS
- shadcn/ui
- consistent spacing
- consistent typography
- semantic status badges
- accessible form controls
- consistent button hierarchy

---

# 9. Button Hierarchy

Primary action:

```text
Create Product
Create Purchase Order
Confirm Sale
Approve
```

Secondary:

```text
Cancel
Back
Export
Filter
```

Destructive:

```text
Cancel Order
Reject
Deactivate
```

Destructive actions should require confirmation where consequences are material.

---

# 10. Status Presentation

Use consistent status badges.

Examples:

```text
DRAFT
ACTIVE
INACTIVE
PENDING
APPROVED
REJECTED
COMPLETED
CANCELLED
EXPIRED
FAILED
```

Status should be communicated by:

- text
- icon where helpful
- semantic visual treatment

Do not rely on color alone.

---

# 11. Authentication Screens

## 11.1 Login

Fields:

```text
Email
Password
```

Actions:

```text
Sign In
```

Supporting states:

- invalid credentials
- account inactive
- session expired
- server unavailable

---

## 11.2 Organization Creation

Qualified users may create an organization and become its initial ADMIN.

Fields may include:

```text
Organization Name
Organization Code
Base Currency
```

The exact onboarding flow should follow the finalized organization policy.

---

## 11.3 Session Expiry

When authentication expires:

```text
Session expired
Please sign in again.
```

Avoid silently losing unsaved form data where possible.

---

# 12. Dashboard

## Purpose

Provide a concise operational overview.

---

## Dashboard structure

```text
Welcome / Organization Context

[Inventory Value*] [Low Stock] [Pending Approvals] [Open POs]

Inventory Health
┌─────────────────────────────────────────────────────────────┐
│ Stock summary / trends                                      │
└─────────────────────────────────────────────────────────────┘

Low Stock
┌─────────────────────────────────────────────────────────────┐
│ Product │ Warehouse │ Available │ Reorder Point │ Status   │
└─────────────────────────────────────────────────────────────┘

Recent Activity
┌─────────────────────────────────────────────────────────────┐
│ Time │ Event │ User │ Reference                            │
└─────────────────────────────────────────────────────────────┘

AI Briefing
┌─────────────────────────────────────────────────────────────┐
│ Key observations and recommendations                        │
└─────────────────────────────────────────────────────────────┘
```

`Inventory Value` must only appear for roles allowed to see cost.

---

# 13. Dashboard KPI Cards

Potential cards:

- Available Stock
- Low Stock Items
- Pending Approvals
- Open Purchase Orders
- Sales Today
- Recent Inventory Changes
- Inventory Distribution by Warehouse

Cards must respect role and warehouse scope.

---

# 14. Dashboard Freshness

For operational metrics:

```text
Updated just now
```

or:

```text
Updated 5 minutes ago
```

For cached/aggregated values, show appropriate freshness.

---

# 15. Inventory Overview

Route:

```text
/inventory
```

Purpose:

Provide a warehouse/product inventory view.

Primary controls:

```text
Warehouse
Product
Category
Brand
Stock Status
Batch
Expiry
```

Table:

```text
Product
SKU
Warehouse
Sellable
Reserved
Available
Damaged
Expired
Reorder Point
Status
```

Cost columns appear only for authorized roles.

---

# 16. Inventory Stock Detail

Route:

```text
/inventory/[product]/[warehouse]
```

Sections:

```text
Product Summary
Current Stock
Batch Breakdown
Recent Movements
Reservations
Cost Information
```

Example:

```text
Product: ABC-001
Warehouse: Main Warehouse

Sellable       120
Reserved        20
Available      100
Damaged          3
Expired          2
```

The UI should not display a separately stored available value that conflicts with backend state.

---

# 17. Inventory Movement Screen

Route:

```text
/inventory/movements
```

Filters:

- date range
- warehouse
- product
- movement type
- batch
- user
- reference

Columns:

```text
Date
Product
Warehouse
Movement Type
Quantity
Unit Cost*
Reference
User
```

Cost is role-controlled.

---

# 18. Movement Detail

Show:

```text
Movement ID
Type
Product
Warehouse
Batch
Quantity
Cost
Reference
Created By
Created At
```

Link to originating business object where authorized.

---

# 19. Inventory Adjustment Screen

Route:

```text
/inventory/adjustments
```

Primary action:

```text
New Adjustment
```

Form:

```text
Warehouse
Product
Batch
Adjustment Type
Quantity
Reason
```

Adjustment types:

```text
IN
OUT
```

Before submission show:

```text
Current Sellable
Current Available
Requested Change
Projected Result
```

Final authority remains backend validation.

---

# 20. Adjustment Confirmation

Before committing:

```text
Review Adjustment

Warehouse: Main Warehouse
Product: ABC-001
Type: OUT
Quantity: 10
Reason: Damaged stock

[Cancel] [Confirm Adjustment]
```

If the current inventory changed since the form was opened, backend validation must reject or revalidate.

---

# 21. Batch / Expiry Screen

Route:

```text
/inventory/batches
```

Columns:

```text
Batch
Product
Warehouse
Quantity
Expiry Date
Days Until Expiry
Status
```

Filters:

```text
Warehouse
Product
Expiry Range
Status
```

Useful states:

```text
Expired
Expiring Soon
Active
```

The exact “expiring soon” threshold should be configurable rather than assumed as a business rule.

---

# 22. Products List

Route:

```text
/catalog/products
```

Table:

```text
SKU
Product Name
Category
Brand
Unit
Status
Reorder Point
Created
Actions
```

Actions depend on role.

---

# 23. Product Creation

Form sections:

### Basic Information

```text
Product Name
SKU
Description
Category
Brand
Unit
```

### Inventory Settings

```text
Reorder Point
Reorder Quantity
```

### Commercial

```text
Default Cost
Default Sale Price
Tax Rate
```

Commercial/cost fields must respect role.

---

# 24. Product Detail

Sections:

```text
Overview
Inventory
Batches
Purchase History
Sales History
Movement History
AI Insights
```

Actions:

```text
Edit
Deactivate
Archive
```

Only authorized roles can mutate.

---

# 25. Product Lifecycle UI

Show status:

```text
DRAFT
ACTIVE
INACTIVE
ARCHIVED
```

For inactive products:

```text
This product remains in historical records but cannot be used in new transactions.
```

---

# 26. Categories Screen

Features:

- list
- search
- create
- edit
- deactivate

Columns:

```text
Name
Description
Status
Product Count
Actions
```

---

# 27. Brands Screen

Similar structure:

```text
Brand
Description
Status
Product Count
Actions
```

---

# 28. Units Screen

Display:

```text
Unit
Symbol
Base Unit
Conversions
```

Provide explicit conversion management.

---

# 29. Warehouses List

Route:

```text
/warehouses
```

Columns:

```text
Code
Name
Location
Status
Inventory Count
Actions
```

Do not show unsupported capacity/utilization metrics.

---

# 30. Warehouse Detail

Sections:

```text
Overview
Inventory
Movements
Transfers
Staff Access
```

For restricted users, only authorized warehouses are visible.

---

# 31. Warehouse Staff Assignment

Admin/manager interface:

```text
User
Role
Assigned Warehouses
Status
```

Restricted roles receive explicit assignments.

---

# 32. Supplier List

Route:

```text
/purchasing/suppliers
```

Columns:

```text
Supplier Code
Supplier
Contact
Status
Purchase Orders
Actions
```

---

# 33. Supplier Detail

Sections:

```text
Overview
Purchase History
Performance
Returns
AI Recommendations
```

Potential metrics:

- order count
- purchase value
- delivery performance
- return frequency

Metrics must be backed by defined report calculations.

---

# 34. Purchase Order List

Route:

```text
/purchasing/orders
```

Filters:

- status
- supplier
- warehouse
- date range
- created by

Columns:

```text
PO Number
Supplier
Warehouse
Order Date
Expected Date
Total
Status
Actions
```

---

# 35. Purchase Order Creation

Form:

```text
Supplier
Warehouse
Expected Date
Notes

Items
----------------------------------
Product
Quantity
Unit
Unit Cost
Tax
Line Total
----------------------------------
```

Actions:

```text
Save Draft
Submit for Approval
```

Submitted purchase orders require ADMIN/MANAGER approval.

---

# 36. Purchase Order Detail

Sections:

```text
Summary
Items
Approval History
Receiving
Returns
Audit
```

Actions vary by status:

```text
Edit Draft
Submit
Approve
Reject
Receive
Cancel
```

---

# 37. Purchase Approval Screen

Route:

```text
/approvals
```

Purchase approval card:

```text
Purchase Order
Supplier
Warehouse
Total
Requested By
Submitted At

Items

[Reject] [Approve]
```

Approvers should see enough context to make an informed decision.

---

# 38. Receiving Screen

Route:

```text
/purchasing/receiving
```

Flow:

```text
Select Approved PO
 ↓
Review Items
 ↓
Enter Received Quantities
 ↓
Enter/Confirm Batch Information
 ↓
Confirm Receipt
 ↓
Inventory Updated
```

The UI must clearly warn that confirmation changes inventory.

---

# 39. Receiving Validation

Display:

```text
Ordered
Previously Received
Receiving Now
Remaining
```

Do not permit invalid quantities.

Backend remains authoritative.

---

# 40. Purchase Returns

Route:

```text
/purchasing/returns
```

List:

```text
Return Number
PO
Supplier
Warehouse
Status
Quantity
Date
```

Creating a draft must not change inventory.

Confirmed/dispatched return workflow changes stock.

---

# 41. Customer List

Route:

```text
/sales/customers
```

Columns:

```text
Customer Code
Name
Contact
Status
Sales Count
Actions
```

---

# 42. Customer Detail

Sections:

```text
Profile
Sales History
Returns
Activity
```

---

# 43. Sales Order List

Route:

```text
/sales/orders
```

Filters:

- status
- warehouse
- customer
- date
- created by

Columns:

```text
Order Number
Customer
Warehouse
Date
Total
Status
Actions
```

---

# 44. Sales Order Creation

Form:

```text
Customer
Warehouse

Items
--------------------------------
Product
Available Stock
Quantity
Unit Price
Tax
Line Total
--------------------------------

Subtotal
Tax
Total
```

The displayed availability is informational.

At confirmation, the backend re-checks current inventory.

---

# 45. Sales Confirmation

Before confirmation:

```text
Confirm Sale

This will reserve the required inventory.

[Cancel] [Confirm Sale]
```

After confirmation:

```text
Reserved
```

must be visible.

---

# 46. Sales Completion

Completion should clearly communicate:

```text
Completing this sale will consume the reserved stock.
```

Confirmation:

```text
[Complete Sale]
```

Backend performs the transactional stock consumption.

---

# 47. Concurrent Sales UI

If another transaction consumes the available stock first:

```text
Unable to complete sale.

The available stock changed before this transaction could be completed.

Please review the current quantity.
```

Do not show a generic “something went wrong” message.

---

# 48. Sales Returns

Route:

```text
/sales/returns
```

Flow:

```text
Select Sale
 ↓
Select Items
 ↓
Enter Return Quantity
 ↓
Inspection
 ↓
Classify:
    Sellable
    Damaged
    Expired
 ↓
Complete Return
```

Returned goods must not automatically become sellable.

---

# 49. Transfer List

Route:

```text
/warehouses/transfers
```

Columns:

```text
Transfer Number
Source
Destination
Date
Status
Requested By
Actions
```

---

# 50. Transfer Creation

Form:

```text
Source Warehouse
Destination Warehouse
Items
    Product
    Batch
    Quantity
Notes
```

Source and destination must differ.

---

# 51. Transfer Approval

Approval page:

```text
Transfer Details
Source
Destination
Items
Requested By

[Reject] [Approve]
```

Approval does not mean inventory mutation has bypassed validation.

---

# 52. Transfer Execution

Before execution:

```text
Review current source availability.
```

Backend revalidates current stock.

Completion produces:

```text
TRANSFER_OUT
TRANSFER_IN
```

---

# 53. Reports Screen

Route:

```text
/reports
```

Report categories:

```text
Inventory
Sales
Purchasing
Suppliers
Customers
Warehouse
Audit
AI Insights
```

The actual report catalog is finalized during API/report implementation.

---

# 54. Report Filters

Common filters:

```text
Date Range
Warehouse
Product
Category
Brand
Supplier
Customer
Status
```

Only authorized dimensions appear.

---

# 55. Report Results

Large result sets should support:

```text
Pagination
Sort
Filter
Export
```

Exports may be asynchronous.

---

# 56. Report Job UI

When an export is generated asynchronously:

```text
Export requested.

Status:
Queued
Processing
Completed
Failed
```

On completion:

```text
Download
```

The download must respect organization and authorization.

---

# 57. Notifications Center

Route:

```text
/notifications
```

Tabs:

```text
All
Unread
Approvals
Inventory
System
AI
```

Each notification:

```text
Title
Message
Timestamp
Related Object
Read State
```

---

# 58. Approval Center

Route:

```text
/approvals
```

Filters:

```text
Type
Status
Requested By
Date
```

Cards/table should include:

```text
Action
Object
Requester
Created
Status
```

---

# 59. Approval Detail

Show:

```text
Requested Action
Current Resource State
Proposed Changes
Requester
Created At
AI-generated? Yes/No
Reason
```

For AI actions:

```text
Recommendation
Confidence
Data As Of
Potential Impact
```

Then:

```text
Approve
Reject
```

---

# 60. AI Copilot

Route:

```text
/ai
```

The AI experience should be conversational but operationally grounded.

Structure:

```text
┌─────────────────────────────────────────────────────────────┐
│ AI Copilot                                                  │
├─────────────────────────────────────────────────────────────┤
│ Context: Main Warehouse                                    │
│                                                             │
│ User: Which products are below reorder point?              │
│                                                             │
│ AI: 8 products are currently below their reorder points.   │
│                                                             │
│ [View Products]                                             │
│                                                             │
│ ----------------------------------------------------------- │
│ Ask about inventory...                         [Send]        │
└─────────────────────────────────────────────────────────────┘
```

---

# 61. AI Context Indicator

The Copilot should display current context:

```text
Organization: ABC Trading
Warehouse: Main Warehouse
Data: Live
```

or:

```text
Warehouse: All authorized warehouses
Data retrieved: 4 minutes ago
```

This prevents ambiguous questions.

---

# 62. AI Answer Types

Responses should be visually distinguished.

### FACT

```text
Current available stock is 120 units.
```

### CALCULATION

```text
Sales increased 18% compared with the previous period.
```

### PREDICTION

```text
Demand is projected to increase next week.
```

### RECOMMENDATION

```text
Consider ordering 50 units.
```

### ACTION

```text
I prepared a draft purchase order for review.
```

---

# 63. AI Source/Freshness Panel

For operational answers:

```text
Data source:
Inventory service

Data as of:
10:42 AM

Warehouse:
Main Warehouse
```

This does not expose internal SQL or sensitive implementation details.

---

# 64. AI Recommendation Card

Example:

```text
Reorder Recommendation

Product: ABC-001
Current Available: 12
Reorder Point: 20
Recommended Quantity: 50

Reason:
Recent demand has increased.

Confidence:
Medium

Data as of:
10:41 AM

[View Product] [Create Draft]
```

Creating a draft does not mean executing the action.

---

# 65. AI Draft Action

Example:

```text
Draft Purchase Order

Supplier: Supplier A
Warehouse: Main Warehouse

ABC-001    50 units
ABC-002    30 units

Estimated Total: ₹XX,XXX

Status:
Draft

[Review Draft] [Discard]
```

---

# 66. AI High-Impact Action Approval

Example:

```text
AI Action Requires Approval

Action:
Create Purchase Order

Reason:
Multiple products are below reorder point.

Proposed Changes:
...

Data as of:
...

Impact:
Creates a purchase order for ₹XX,XXX.

[Reject] [Approve]
```

Approval triggers normal backend revalidation.

---

# 67. AI Stale Action

If underlying data changed:

```text
This recommendation is no longer current.

Inventory changed after this action was proposed.

The action must be reviewed again.
```

Do not silently execute stale actions.

---

# 68. AI Tool Error

If a tool fails:

```text
I couldn't retrieve the current inventory data.

Please try again.
```

Do not invent an answer from stale conversation context.

---

# 69. AI Permission Error

If a user asks for restricted information:

```text
I can’t provide that information because it is outside your current access.
```

Do not reveal the existence of unauthorized sensitive data beyond what is appropriate.

---

# 70. AI Prompt Injection UX

If retrieved content contains suspicious instructions, the user should not see hidden system prompts.

The system should safely ignore the untrusted instruction and continue where possible.

---

# 71. AI Conversation History

Users should be able to:

- start a new conversation
- rename a conversation
- reopen previous conversations
- delete/archive where policy permits

Data retention remains subject to DEC-011.

---

# 72. AI Suggested Prompts

Useful starting prompts:

```text
What products are low in stock?

Show me today's inventory changes.

Which products have unusual demand?

Which suppliers have performed best recently?

What purchase orders need approval?

Give me today's inventory briefing.
```

Suggested prompts must not imply capabilities the current user's permissions do not allow.

---

# 73. AI Daily Briefing

Dashboard card:

```text
Today's Inventory Briefing

3 notable observations
2 recommendations
1 approval requiring attention

[View Full Briefing]
```

The briefing must distinguish facts from recommendations.

---

# 74. Search UX

Global search should provide categorized results:

```text
Products
Orders
Suppliers
Customers
Transfers
```

Example:

```text
ABC-001

Products (1)
Purchase Orders (4)
Sales Orders (17)
```

Results remain authorization-scoped.

---

# 75. Tables

Tables should provide:

- column labels
- sorting where useful
- filtering
- pagination
- row actions
- loading state
- empty state
- error state

Avoid overly dense tables on mobile.

---

# 76. Empty States

Good empty state:

```text
No purchase orders found.

Try changing your filters or create a new purchase order.

[Create Purchase Order]
```

Avoid:

```text
No data.
```

---

# 77. Loading States

Use skeletons for:

- dashboards
- tables
- detail panels

Use button loading states for mutations:

```text
Saving...
Submitting...
Approving...
Completing...
```

Prevent accidental duplicate submissions.

---

# 78. Error States

Errors should provide:

```text
What happened
Why it may have happened
What the user can do
```

Example:

```text
Unable to complete the sale.

The available stock changed and is now insufficient.

Review the current inventory and update the quantity.
```

---

# 79. Confirmation Dialogs

Use confirmations for:

- irreversible actions
- high-impact actions
- inventory mutations where appropriate
- cancellation
- approval
- rejection
- deactivation

Do not require confirmation for every minor action.

---

# 80. Form Design

Forms should:

- group related fields
- show required fields
- show inline validation
- preserve user input on validation errors
- disable duplicate submissions
- provide clear save/submit semantics

---

# 81. Form Validation

Validation layers:

```text
UI validation
    ↓
API validation
    ↓
Business validation
    ↓
Database constraints
```

A UI-valid form can still be rejected by the backend.

The UI should display backend validation errors clearly.

---

# 82. Date and Time

Display dates according to the user's locale.

Operational timestamps should retain enough precision for audit/troubleshooting.

Example:

```text
5 Oct 2026, 10:42 AM
```

The underlying API remains timezone-aware.

---

# 83. Currency

Display the organization's configured base currency.

Example:

```text
₹ 12,500.00
```

The UI must not assume INR for every organization.

The database/API organization configuration is authoritative.

---

# 84. Quantity Formatting

Quantities should preserve meaningful decimal precision.

Examples:

```text
12 units
12.5 kg
3.25 liters
```

The UI should use the product/unit context.

---

# 85. Access Denied Screen

If a user navigates to a page they cannot access:

```text
Access Restricted

You don't have permission to view this page.
```

Do not expose sensitive data in the error.

---

# 86. Not Found Screen

For unavailable resources:

```text
Resource not found.

It may have been removed, archived, or you may not have access.
```

Avoid leaking cross-tenant existence information.

---

# 87. Settings

Settings should be role-aware.

Possible sections:

```text
Organization
Users & Roles
Warehouses
Catalog Defaults
Notifications
AI Preferences
Security
```

Do not expose unresolved business configuration as if it were finalized.

---

# 88. Organization Settings

Potential fields:

```text
Organization Name
Organization Code
Base Currency
```

Base currency should not be casually changed once financial/transactional history exists.

Final change policy belongs to business rules/API design.

---

# 89. User Management

Admin interface:

```text
User
Email
Role
Status
Warehouses
Last Activity
Actions
```

Actions:

```text
Invite/Add
Change Role
Assign Warehouse
Deactivate
```

Backend authorization is mandatory.

---

# 90. Permission Visibility

UI action visibility should follow role:

```text
Can Create
Can Edit
Can Approve
Can Execute
Can View Cost
```

The frontend should receive effective permissions from backend context where appropriate.

---

# 91. Cost Visibility

For unauthorized users:

Do not display:

- unit cost
- inventory valuation
- purchase cost
- supplier cost analytics

Do not merely disable the field while leaving the value in HTML/API responses.

The backend must omit unauthorized sensitive values.

---

# 92. Inventory Value Dashboard

If inventory value is shown:

```text
Inventory Value
```

must be restricted to roles allowed to view cost.

If cost visibility is unavailable:

```text
Inventory Value
```

should not be shown.

---

# 93. Audit UI

Route:

```text
/settings/audit
```

or equivalent.

Filters:

```text
Date
User
Action
Entity
Warehouse
```

Columns:

```text
Timestamp
User
Action
Entity
Reference
```

Audit details may show structured metadata subject to permission.

---

# 94. Audit Detail

Display:

```text
Action
Actor
Timestamp
Entity
Request ID
Before/After information where appropriate
Metadata
```

Sensitive security data must remain hidden.

---

# 95. Accessibility

The UI must support:

- keyboard navigation
- visible focus states
- semantic HTML
- accessible labels
- screen-reader-friendly controls
- sufficient contrast
- non-color status indicators
- logical tab order
- accessible dialogs
- accessible tables

Target:

```text
WCAG 2.1 AA
```

where practical.

---

# 96. Keyboard UX

Common actions should be keyboard accessible.

Examples:

```text
Tab
Enter
Escape
Arrow navigation
```

AI chat should support:

```text
Enter → send
Shift+Enter → newline
```

unless a different accessible interaction is required.

---

# 97. Toasts

Use toasts for lightweight feedback:

```text
Product saved.
Purchase order submitted.
Notification marked as read.
```

Do not use a toast as the only way to communicate critical failures.

---

# 98. Unsaved Changes

Forms with meaningful data should warn before navigation if changes would be lost.

Example:

```text
You have unsaved changes.

Leave without saving?
```

---

# 99. Auditability in UI

After important mutations, the UI should provide a route to relevant history.

Example:

```text
Adjustment completed.

[View Inventory]
[View Movement]
```

This supports user trust and troubleshooting.

---

# 100. Optimistic Updates

Avoid optimistic updates for authoritative inventory mutations unless the state can be safely reconciled.

For:

- stock adjustments
- sale completion
- receiving
- transfer completion

prefer:

```text
submit
 ↓
server transaction
 ↓
authoritative response
 ↓
refresh/invalidate UI
```

---

# 101. Query Invalidation

After mutations, invalidate affected TanStack Query data.

Examples:

```text
Sale completed
→ sales detail
→ inventory
→ movements
→ dashboard
→ notifications if applicable
```

The backend remains authoritative.

---

# 102. Real-Time Updates

Real-time updates are optional.

The initial system may use:

```text
request/response
+
query invalidation
+
manual refresh
```

WebSockets/SSE should only be introduced when a concrete workflow benefits from them.

---

# 103. Notification Refresh

Unread notifications may be refreshed periodically or after relevant mutations.

The system should not require real-time infrastructure for initial functionality.

---

# 104. Dashboard Refresh

Dashboard data should support:

```text
Refresh
```

and may use a reasonable polling/staleness strategy.

The refresh strategy must not create excessive database load.

---

# 105. AI Cost/Latency UX

If AI requests take longer:

```text
Analyzing inventory...
Checking authorized data...
Preparing recommendation...
```

The UI should avoid exposing misleading internal details.

A cancel option may be provided for long-running requests where supported.

---

# 106. AI Streaming

Streaming AI responses may be used for conversational UX.

However:

- streamed text is not authoritative
- tool results remain authoritative
- action execution must wait for complete validated workflow

---

# 107. AI Tool Activity Display

The UI may show safe high-level activity:

```text
Checking inventory...
Reviewing recent sales...
Preparing recommendation...
```

Do not expose:

- raw SQL
- secrets
- internal system prompts
- sensitive internal tool payloads

---

# 108. AI Action Audit Trail UI

For AI-created actions:

```text
Created by:
AI Copilot

Requested by:
Anurag

Approved by:
Manager

Executed:
5 Oct 2026, 10:42 AM
```

Only information authorized for the current user should be shown.

---

# 109. Mobile AI

On mobile:

```text
full-screen conversation
+
sticky input
+
collapsible context
+
action cards
```

Approval actions must remain clear and easy to review.

---

# 110. Mobile Inventory

Use cards for key inventory information:

```text
ABC-001
Main Warehouse

Available: 100
Reserved: 20
Damaged: 3
Expired: 2

[View Details]
```

Detailed movement data may remain horizontally scrollable.

---

# 111. Mobile Forms

Use single-column forms.

Long forms should be divided into sections.

Avoid requiring users to manage multiple modal dialogs on small screens.

---

# 112. Desktop Power-User UX

Desktop workflows should support efficient operation through:

- keyboard-friendly tables
- filters
- saved filter state where useful
- bulk selection where safe
- quick navigation
- clear shortcuts where justified

Bulk mutations must retain the same backend validation as individual mutations.

---

# 113. Bulk Actions

Potential bulk actions:

- archive/deactivate products
- mark notifications read
- export selected data
- assign warehouse access

Inventory-changing bulk actions require special review and should not bypass per-item validation.

---

# 114. Bulk Inventory Operations

If introduced:

```text
Upload/Select
 ↓
Preview
 ↓
Validate
 ↓
Show errors
 ↓
Confirm
 ↓
Transactional processing
```

Do not apply a bulk file directly to inventory.

---

# 115. Import Error UI

Example:

```text
Import validation failed

12 rows contain errors.

Row 18:
Unknown SKU

Row 24:
Unauthorized warehouse

[Download Error Report]
[Fix File]
```

---

# 116. Product Picker

Product selectors should support:

```text
SKU
Name
Category
Barcode if later introduced
```

Search must be server-backed for large catalogs.

Only authorized/in-scope products should appear where relevant.

---

# 117. Warehouse Picker

Warehouse selectors must only contain authorized warehouses.

For managers/admins:

```text
All Warehouses
```

may be available for reporting or aggregate views.

---

# 118. Batch Picker

Batch selectors should display:

```text
Batch Number
Expiry
Available Quantity
```

Expired or unavailable batches should not be selectable for normal sellable stock operations.

---

# 119. Sales Quantity UX

When entering quantity:

```text
Available: 20

Quantity: [ 5 ]
```

If quantity exceeds current displayed availability:

```text
Requested quantity exceeds currently displayed available stock.
```

But the backend remains the final authority.

---

# 120. Purchase Quantity UX

For purchase orders:

```text
Quantity
Unit Cost
```

The system may show reorder suggestions but must not silently insert AI recommendations into the order.

---

# 121. AI Recommendation Acceptance UX

The user should explicitly choose:

```text
Use Recommendation
```

rather than:

```text
Auto Apply
```

unless a future business policy explicitly permits automation.

---

# 122. AI Action Confirmation

High-impact actions require two conceptual stages:

```text
Review
 ↓
Approve
```

The UI should not combine them into an accidental one-click execution.

---

# 123. Approval Stale-State UX

If approval becomes stale:

```text
This approval can no longer be executed because the underlying data changed.

[Review Updated Data]
```

---

# 124. Error Recovery

After an error, provide the next useful action.

Examples:

```text
Stock changed
→ Review inventory

Session expired
→ Sign in

Report failed
→ Retry

AI unavailable
→ Try again / use normal workflow
```

---

# 125. Offline Behavior

The initial system is not designed as an offline-first application.

If connectivity is lost:

```text
Connection lost.

Changes have not been submitted.
```

Do not queue critical inventory mutations locally for later execution unless a future architecture explicitly supports safe offline synchronization.

---

# 126. Security UX

Never expose:

- authentication tokens
- raw database errors
- stack traces
- secrets
- internal system prompts
- unauthorized cost data

Production errors should use safe user-facing messages.

---

# 127. Navigation Rules

Navigation must preserve:

- current organization
- current warehouse context
- relevant filters where practical

When changing organization/warehouse context, stale pages should refresh their data.

---

# 128. Context Change Warning

If changing warehouse context while editing a warehouse-specific transaction:

```text
Changing warehouse will discard the current transaction context.

Continue?
```

Use only when necessary.

---

# 129. Breadcrumbs

Detail pages should use breadcrumbs:

```text
Inventory
 > Products
 > ABC-001
```

This is especially useful for deep operational pages.

---

# 130. Page Titles

Use descriptive page titles:

```text
Inventory Overview
Purchase Orders
Create Purchase Order
Sales Order SO-00021
Transfer TRF-00031
AI Copilot
```

---

# 131. URL Design

Conceptual route structure:

```text
/dashboard

/inventory
/inventory/movements
/inventory/adjustments
/inventory/batches

/catalog/products
/catalog/products/[id]
/catalog/categories
/catalog/brands
/catalog/units

/purchasing/orders
/purchasing/orders/[id]
/purchasing/receiving
/purchasing/returns
/purchasing/suppliers

/sales/orders
/sales/orders/[id]
/sales/customers
/sales/customers/[id]
/sales/returns

/warehouses
/warehouses/[id]
/warehouses/transfers
/warehouses/transfers/[id]

/reports
/approvals
/notifications
/ai

/settings
```

Exact Next.js route structure may change during implementation.

---

# 132. Role-Based Action Matrix

| Action | SUPER_ADMIN | ADMIN | MANAGER | INVENTORY_STAFF | SALES_STAFF | VIEWER |
|---|---:|---:|---:|---:|---:|---:|
| View dashboard | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| View inventory | Policy dependent | ✓ | ✓ | ✓ | ✓ | ✓ |
| View cost | Policy dependent | ✓ | ✓ | No | No | No |
| Create product | Policy dependent | ✓ | ✓ | Controlled | No | No |
| Adjust inventory | Policy dependent | ✓ | ✓ | Controlled | No | No |
| Approve PO | Policy dependent | ✓ | ✓ | No | No | No |
| Approve transfer | Policy dependent | ✓ | ✓ | No | No | No |
| Create sale | Policy dependent | ✓ | ✓ | No | ✓ | No |
| Complete sale | Policy dependent | ✓ | ✓ | No | ✓ | No |
| View reports | Policy dependent | ✓ | ✓ | Controlled | Controlled | Controlled |
| Approve AI action | Policy dependent | ✓ | ✓ | No | No | No |
| Execute high-impact AI action | Policy dependent | ✓ | ✓ | No | No | No |

The exact final permission matrix remains governed by the FRD/Business Rules and later API implementation.

---

# 133. AI Capability Visibility

The Copilot should not advertise actions the current user cannot perform.

For example, a viewer may ask:

```text
What products are low in stock?
```

but should not be presented with:

```text
Create Purchase Order
```

if that user cannot create the underlying action.

---

# 134. Cost Privacy in AI

If a user cannot view cost:

AI must not reveal:

- purchase cost
- weighted average cost
- inventory valuation
- cost-based supplier comparisons

The restriction applies to AI answers as well as normal UI.

---

# 135. Warehouse Privacy in AI

AI queries must use the same warehouse scope as the user's normal application access.

Example:

```text
Inventory Staff
Assigned:
Warehouse A

Question:
Show stock in Warehouse B.

Response:
Access restricted.
```

---

# 136. AI Action Visibility

Users should see:

```text
My actions
```

and authorized approval queues.

An AI action from another organization must never appear.

---

# 137. Loading and Freshness for AI Recommendations

Recommendation cards should include:

```text
Generated:
10:42 AM

Data as of:
10:40 AM
```

This makes the distinction explicit.

---

# 138. Empty AI State

When no conversation exists:

```text
What would you like to know about your inventory?

Try:
• Which products are low in stock?
• What changed today?
• Which suppliers are performing best?
```

---

# 139. AI Failure Fallback

If AI is unavailable, normal application functionality must remain usable.

Example:

```text
AI Copilot is temporarily unavailable.

You can still use Inventory, Reports, Purchasing, and Sales normally.
```

AI is an enhancement, not a dependency for core inventory operations.

---

# 140. Core UX Workflows

The following workflows must be visually coherent from start to finish.

---

## Workflow A — Receive Stock

```text
Purchase Orders
 ↓
Approved PO
 ↓
Receive
 ↓
Enter quantities
 ↓
Batch/expiry
 ↓
Review
 ↓
Confirm
 ↓
Success
 ↓
Inventory updated
```

---

## Workflow B — Sell Stock

```text
Sales
 ↓
Create order
 ↓
Confirm
 ↓
Reserve
 ↓
Complete
 ↓
Consume reservation
 ↓
Inventory movement
```

---

## Workflow C — Adjust Stock

```text
Inventory
 ↓
Adjustment
 ↓
Enter reason
 ↓
Review
 ↓
Confirm
 ↓
Movement
 ↓
Audit
```

---

## Workflow D — Transfer Stock

```text
Transfers
 ↓
Create
 ↓
Submit
 ↓
Manager/Admin approval
 ↓
Revalidate
 ↓
Execute
 ↓
Transfer Out + Transfer In
```

---

## Workflow E — AI Recommendation

```text
AI Copilot
 ↓
Question
 ↓
Authorized tool
 ↓
Current data
 ↓
Recommendation
 ↓
User review
 ↓
Optional draft
```

---

## Workflow F — AI High-Impact Action

```text
AI
 ↓
Recommendation
 ↓
Draft action
 ↓
Approval queue
 ↓
Human approval
 ↓
Fresh revalidation
 ↓
Normal application service
 ↓
Execution
 ↓
Audit
```

---

# 141. UI Acceptance Criteria

The UI specification is considered satisfied when:

- [ ] all major modules have defined screens
- [ ] navigation is defined
- [ ] role-aware visibility is defined
- [ ] warehouse scope is reflected
- [ ] inventory screens reflect the approved stock model
- [ ] stock mutations require server confirmation
- [ ] purchase approval flow is defined
- [ ] transfer approval flow is defined
- [ ] sales reservation flow is defined
- [ ] returns inspection flow is defined
- [ ] reports are defined
- [ ] notifications are defined
- [ ] audit visibility is defined
- [ ] AI Copilot is defined
- [ ] AI action approval UX is defined
- [ ] AI freshness/confidence presentation is defined
- [ ] loading states are defined
- [ ] error states are defined
- [ ] empty states are defined
- [ ] responsive behavior is defined
- [ ] accessibility expectations are defined
- [ ] cost privacy is defined
- [ ] AI authorization behavior is defined
- [ ] unresolved business decisions remain open

---

# 142. Implementation Guidance

When implementation begins:

1. Build the application shell.
2. Implement authentication.
3. Implement organization and role context.
4. Implement dashboard foundation.
5. Implement catalog screens.
6. Implement warehouse screens.
7. Implement inventory screens.
8. Implement purchasing.
9. Implement receiving.
10. Implement sales.
11. Implement returns.
12. Implement transfers.
13. Implement reports.
14. Implement notifications and approvals.
15. Implement AI Copilot.
16. Implement AI recommendations.
17. Implement AI action workflows.

Do not implement the AI action UI before the underlying approval and application-service architecture exists.

---

# 143. UI-to-Backend Boundary

The UI may:

```text
collect input
validate basic input
display data
request operations
show progress
show results
```

The UI must not:

```text
decide authorization
decide final stock availability
perform authoritative inventory mutations
approve AI actions implicitly
calculate authoritative financial totals
bypass application services
```

---

# 144. Final UI Principle

> **The interface should make the correct workflow easy, the incorrect workflow difficult, and the system's current state understandable.**

For inventory operations in particular:

> **Show the user what the system currently knows, ask for explicit intent when the action has consequences, and always let the backend make the final decision.**

---

# 145. Next Artifact

The next specification should be:

**AI-SPECIFICATION.md**

It should define:

- AI Copilot behavior
- intent classification
- tool registry
- tool schemas
- read vs write tools
- context construction
- permission propagation
- freshness
- confidence
- structured outputs
- prompt-injection defenses
- AI action lifecycle
- human approval
- revalidation
- AI recommendation types
- demand analysis
- reorder recommendations
- supplier recommendations
- anomaly detection
- daily briefing
- AI report summaries
- provider abstraction
- model configuration
- token/cost controls
- AI observability
- AI testing
- failure behavior

No production AI implementation should begin before that specification is complete.
