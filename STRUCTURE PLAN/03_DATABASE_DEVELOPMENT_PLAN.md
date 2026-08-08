# Database Development Plan

## 1. Project Overview

**Project:** Mini ERP + CRM Operations Portal

The database uses PostgreSQL to model the required ERP/CRM entities and
support the application's business workflows.

The design prioritizes:

-   Relational integrity
-   Clear relationships
-   Transaction safety
-   Historical data preservation
-   Simple maintainability
-   Assignment-focused scope

------------------------------------------------------------------------

## 2. Database Technology

**PostgreSQL**

PostgreSQL is selected because:

-   It satisfies the assignment requirements.
-   The application contains strongly relational business data.
-   Inventory confirmation requires transactions.
-   PostgreSQL integrates well with Prisma.
-   It can be deployed using a hosted PostgreSQL provider.

------------------------------------------------------------------------

## 3. PostgreSQL and pgAdmin

pgAdmin is a PostgreSQL administration and query tool. It is not the
database itself.

### Local development

``` text
PostgreSQL
    ↑
  pgAdmin
    ↑
Developer
```

### Production

``` text
Frontend
    ↓
Backend
    ↓
Hosted PostgreSQL
```

A hosted PostgreSQL database can also be accessed through pgAdmin when
the provider supports external PostgreSQL connections.

Potential providers permitted by the assignment include:

-   Supabase
-   Neon
-   Render PostgreSQL

------------------------------------------------------------------------

## 4. Core Entities

``` text
User
Customer
FollowUp
Product
StockMovement
Challan
ChallanItem
```

------------------------------------------------------------------------

## 5. Entity Relationships

``` text
User
 ├── 1:N FollowUp
 ├── 1:N Challan
 └── 1:N StockMovement

Customer
 ├── 1:N FollowUp
 └── 1:N Challan

Product
 ├── 1:N StockMovement
 └── 1:N ChallanItem

Challan
 └── 1:N ChallanItem
```

------------------------------------------------------------------------

## 6. User

### Purpose

Stores application users and their roles.

### Fields

``` text
users
-------------------------
id
name
email
password_hash
role
created_at
updated_at
```

### Roles

``` text
ADMIN
SALES
WAREHOUSE
ACCOUNTS
```

### Constraints

-   `email` unique
-   `password_hash` required
-   `role` required

Passwords are stored only as secure hashes.

------------------------------------------------------------------------

## 7. Customer

### Purpose

Stores CRM customer information.

### Fields

``` text
customers
-------------------------
id
customer_name
mobile
email
business_name
gst_number
customer_type
address
status
follow_up_date
notes
created_at
updated_at
```

### Customer Types

``` text
RETAIL
WHOLESALE
DISTRIBUTOR
```

### Customer Status

``` text
LEAD
ACTIVE
INACTIVE
```

### Constraints

-   Customer name required
-   Mobile required
-   Customer type required
-   Status required
-   GST optional

------------------------------------------------------------------------

## 8. FollowUp

A separate table is recommended to preserve multiple CRM interactions.

### Fields

``` text
follow_ups
-------------------------
id
customer_id
created_by
follow_up_date
notes
created_at
```

### Relationships

``` text
Customer 1 ─── N FollowUps
User     1 ─── N FollowUps
```

------------------------------------------------------------------------

## 9. Product

### Purpose

Stores products and their current inventory state.

### Fields

``` text
products
-------------------------
id
name
sku
category
unit_price
current_stock
minimum_stock
warehouse_location
created_at
updated_at
```

### Constraints

-   SKU unique
-   Unit price \>= 0
-   Current stock \>= 0
-   Minimum stock \>= 0

The backend must prevent negative stock even if database constraints are
also used.

------------------------------------------------------------------------

## 10. StockMovement

### Purpose

Stores the history of inventory changes.

### Fields

``` text
stock_movements
-------------------------
id
product_id
quantity
movement_type
reason
created_by
created_at
```

### Movement Types

``` text
IN
OUT
```

### Quantity Semantics

`quantity` represents the amount changed.

Example:

``` text
Current stock: 50
OUT: 5
New stock: 45
```

Movement:

``` text
quantity = 5
movement_type = OUT
```

------------------------------------------------------------------------

## 11. Challan

### Purpose

Stores the sales challan header.

### Fields

``` text
challans
-------------------------
id
challan_number
customer_id
total_quantity
status
created_by
created_at
updated_at
```

### Status

``` text
DRAFT
CONFIRMED
CANCELLED
```

### Constraints

-   Challan number unique
-   Customer must exist
-   Total quantity \>= 0
-   Status constrained to valid values

------------------------------------------------------------------------

## 12. ChallanItem

### Purpose

Stores the products associated with a challan and preserves product
snapshot information.

### Fields

``` text
challan_items
-------------------------
id
challan_id
product_id
product_name_snapshot
sku_snapshot
unit_price_snapshot
quantity
created_at
```

The exact snapshot fields will be finalized based on the implemented
challan UI and business requirements.

------------------------------------------------------------------------

## 13. Product Snapshot

The assignment requires challans to store product snapshot data rather
than relying only on the current product record.

Example:

``` text
At creation:

Product: USB-C Adapter
SKU: USB-C-001
Price: ₹250
```

If the product later changes:

``` text
Product name → changed
SKU           → changed
Price         → changed
```

The historical challan must continue to represent the original product
information.

Therefore:

``` text
ChallanItem
 ├── product_id
 ├── product_name_snapshot
 ├── sku_snapshot
 └── unit_price_snapshot
```

------------------------------------------------------------------------

## 14. Relationship Model

``` text
users.id
   │
   ├──────────────→ challans.created_by
   │
   ├──────────────→ follow_ups.created_by
   │
   └──────────────→ stock_movements.created_by


customers.id
   │
   ├──────────────→ challans.customer_id
   └──────────────→ follow_ups.customer_id


products.id
   │
   ├──────────────→ challan_items.product_id
   └──────────────→ stock_movements.product_id


challans.id
   │
   └──────────────→ challan_items.challan_id
```

------------------------------------------------------------------------

## 15. Challan Confirmation Transaction

Challan confirmation is the most important transactional workflow.

``` text
BEGIN TRANSACTION
       ↓
Load challan
       ↓
Verify status = DRAFT
       ↓
Load challan items
       ↓
Validate products
       ↓
Check stock for every item
       ↓
Reduce product stock
       ↓
Create OUT movements
       ↓
Update challan → CONFIRMED
       ↓
COMMIT
```

If any step fails:

``` text
ROLLBACK
```

This prevents partial inventory updates.

------------------------------------------------------------------------

## 16. Preventing Negative Stock

For an OUT movement:

``` text
current_stock >= requested_quantity
```

If false:

``` text
Reject operation
```

The application/service layer should perform the business check before
updating stock.

A database-level non-negative constraint may also be used as an
additional safeguard.

------------------------------------------------------------------------

## 17. Challan State Rules

Recommended state transitions:

``` text
DRAFT
 ├──→ CONFIRMED
 └──→ CANCELLED

CONFIRMED
 └──→ No normal reversal

CANCELLED
 └──→ No confirmation
```

A confirmed challan must not be confirmed again.

This prevents duplicate stock reduction.

------------------------------------------------------------------------

## 18. Stock Rules

### IN

``` text
current_stock += quantity
create IN movement
```

### OUT

``` text
check available stock
current_stock -= quantity
create OUT movement
```

### Challan Confirmation

``` text
Confirmed challan
      ↓
Stock validation
      ↓
Stock reduction
      ↓
OUT movement
```

All related operations should occur within one database transaction.

------------------------------------------------------------------------

## 19. Indexes

Recommended indexes should support actual application queries.

Candidates:

``` text
users.email
products.sku
customers.business_name
customers.mobile
challans.challan_number
challans.customer_id
stock_movements.product_id
stock_movements.created_at
```

Indexes should not be added indiscriminately.

------------------------------------------------------------------------

## 20. Search and Pagination

### Customers

-   Name/business/mobile search
-   Status/type filters
-   Pagination

### Products

-   Name/SKU search
-   Category filter
-   Low-stock filter
-   Pagination

### Challans

-   Challan number/customer search
-   Status filter
-   Pagination

### Stock Movements

-   Product filter
-   Movement type filter
-   Date filter
-   Pagination

------------------------------------------------------------------------

## 21. Naming Convention

Use `snake_case` for database fields.

Examples:

``` text
created_at
updated_at
customer_id
product_id
challan_number
current_stock
movement_type
```

Use consistent singular/plural conventions for Prisma models and
database tables.

------------------------------------------------------------------------

## 22. Prisma

Prisma is the database access layer.

The Prisma schema should define:

-   Models
-   Relations
-   Enums
-   Constraints
-   Indexes

Migrations should be used to manage schema changes.

------------------------------------------------------------------------

## 23. Seed Data

Create development/demo seed data.

### Users

-   Admin
-   Sales
-   Warehouse
-   Accounts

### Customers

Include examples covering:

-   Retail
-   Wholesale
-   Distributor
-   Lead
-   Active
-   Inactive

### Products

Include examples with:

-   Normal stock
-   Low stock
-   Out of stock

### Challans

Include examples with:

-   Draft
-   Confirmed
-   Cancelled

Seed credentials must be documented separately and must never contain
real secrets.

------------------------------------------------------------------------

## 24. Database Testing

### Customer

-   Create
-   Update
-   Search
-   Add follow-up

### Product

-   Create
-   Update
-   Stock visibility
-   Low-stock detection

### Inventory

-   IN increases stock
-   OUT decreases stock
-   Insufficient OUT is rejected
-   Stock never becomes negative

### Challan

-   Draft creation
-   Draft editing
-   Confirmation
-   Stock reduction
-   OUT movement creation
-   Insufficient-stock rejection
-   Duplicate-confirmation prevention
-   Cancelled-challan protection
-   Product snapshot preservation

------------------------------------------------------------------------

## 25. Development Order

### Phase 1 --- Foundation

-   PostgreSQL
-   Prisma
-   Database connection
-   Initial schema

### Phase 2 --- Core Entities

-   User
-   Customer
-   FollowUp

### Phase 3 --- Inventory

-   Product
-   StockMovement

### Phase 4 --- Challans

-   Challan
-   ChallanItem
-   Product snapshots

### Phase 5 --- Integrity

-   Relationships
-   Constraints
-   Indexes
-   Seed data

### Phase 6 --- Verification

-   Transaction tests
-   Insufficient-stock tests
-   Duplicate-confirmation tests
-   Relationship tests
-   Data integrity checks

------------------------------------------------------------------------

## 26. Scope Boundary

The database is intentionally limited to the assignment's required
workflows.

Do not add unnecessary enterprise modules such as:

-   Payroll
-   Full accounting
-   Supplier management
-   Advanced procurement
-   Complex taxation
-   Full invoicing
-   Multi-company accounting
-   Advanced reporting

unless the final project scope explicitly requires them.

> **The database should be relational, reliable, and easy to
> explain---not unnecessarily large.**
