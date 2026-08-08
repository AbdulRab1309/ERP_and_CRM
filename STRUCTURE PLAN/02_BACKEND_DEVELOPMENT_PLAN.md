# Backend Development Plan

## 1. Project Overview

**Project:** Mini ERP + CRM Operations Portal

The backend provides the REST API, authentication, authorization,
validation, database access, and business logic required by the
application.

The implementation prioritizes correctness and explainability over
unnecessary enterprise complexity.

------------------------------------------------------------------------

## 2. Technology Stack

  Technology        Purpose
  ----------------- ---------------------------
  Node.js           Runtime
  TypeScript        Type safety
  Express.js        REST API framework
  PostgreSQL        Relational database
  Prisma            ORM/database access
  JWT               Authentication
  bcrypt/bcryptjs   Password hashing
  Zod               Request validation
  dotenv            Environment configuration
  Postman           API testing/documentation

------------------------------------------------------------------------

## 3. Architecture

``` text
HTTP Request
     ↓
Route
     ↓
Middleware
     ↓
Controller
     ↓
Service
     ↓
Prisma
     ↓
PostgreSQL
```

### Responsibilities

**Routes** - Define API endpoints - Attach middleware

**Middleware** - Authentication - Authorization - Validation - Error
handling

**Controllers** - Receive requests - Call services - Return responses

**Services** - Contain business logic - Handle workflows - Enforce
business rules

**Prisma** - Database access - Queries - Transactions

------------------------------------------------------------------------

## 4. Folder Structure

``` text
src/
├── config/
│   ├── env.ts
│   └── database.ts
│
├── middleware/
│   ├── auth.middleware.ts
│   ├── role.middleware.ts
│   ├── validation.middleware.ts
│   └── error.middleware.ts
│
├── modules/
│   ├── auth/
│   ├── customers/
│   ├── products/
│   ├── inventory/
│   └── challans/
│
├── utils/
│   ├── jwt.ts
│   ├── password.ts
│   └── errors.ts
│
├── app.ts
└── server.ts
```

Each module should contain its own routes, controllers, services, and
validation where appropriate.

------------------------------------------------------------------------

## 5. Authentication

### Login

``` http
POST /api/auth/login
```

Flow:

``` text
Credentials
    ↓
Validate
    ↓
Find User
    ↓
Verify Password
    ↓
Generate JWT
    ↓
Return Token + User
```

JWT payload should identify the authenticated user and role.

Example:

``` json
{
  "sub": "user-id",
  "role": "SALES"
}
```

Passwords must never be included in tokens.

------------------------------------------------------------------------

## 6. Password Security

Passwords are stored as secure hashes.

``` text
Password
   ↓
bcrypt
   ↓
password_hash
   ↓
Database
```

During login:

``` text
Supplied password
       ↓
Compare with hash
       ↓
Valid / Invalid
```

------------------------------------------------------------------------

## 7. Role-Based Access Control

Required roles:

-   ADMIN
-   SALES
-   WAREHOUSE
-   ACCOUNTS

Suggested access model:

  Role        Primary Access
  ----------- --------------------------------------------
  Admin       Application-wide administration
  Sales       Customers and Sales Challans
  Warehouse   Products and Inventory
  Accounts    Account-relevant implemented functionality

The exact permission matrix should remain limited to the assignment
scope.

### Request flow

``` text
JWT
 ↓
Authentication Middleware
 ↓
Role Middleware
 ↓
Controller
```

Authorization must be enforced by the backend.

------------------------------------------------------------------------

## 8. Customer APIs

### Create

``` http
POST /api/customers
```

### List

``` http
GET /api/customers
```

Support:

-   Pagination
-   Search
-   Relevant filters

Example:

``` http
GET /api/customers?page=1&limit=20&search=abc
```

### Detail

``` http
GET /api/customers/:id
```

### Update

``` http
PATCH /api/customers/:id
```

### Follow-up

``` http
POST /api/customers/:id/follow-ups
```

Required customer fields:

-   Customer name
-   Mobile number
-   Email
-   Business name
-   GST number
-   Customer type
-   Address
-   Status
-   Follow-up date
-   Notes

------------------------------------------------------------------------

## 9. Product APIs

### Create

``` http
POST /api/products
```

### List

``` http
GET /api/products
```

### Detail

``` http
GET /api/products/:id
```

### Update

``` http
PATCH /api/products/:id
```

Required product fields:

-   Product name
-   SKU/code
-   Category
-   Unit price
-   Current stock
-   Minimum stock quantity
-   Warehouse/location

------------------------------------------------------------------------

## 10. Inventory APIs

### Stock movements

``` http
GET /api/inventory/movements
```

Supported filters may include:

-   Product
-   Movement type
-   Date range
-   Pagination

Movement data:

-   Product
-   Quantity
-   Movement type
-   Reason
-   Created by
-   Timestamp

Stock changes must be controlled by backend business logic.

------------------------------------------------------------------------

## 11. Sales Challan APIs

### Create

``` http
POST /api/challans
```

### List

``` http
GET /api/challans
```

### Detail

``` http
GET /api/challans/:id
```

### Update Draft

``` http
PATCH /api/challans/:id
```

### Confirm

``` http
POST /api/challans/:id/confirm
```

### Cancel

``` http
POST /api/challans/:id/cancel
```

------------------------------------------------------------------------

## 12. Challan Workflow

``` text
Create
  ↓
DRAFT
  ↓
Review
  ↓
CONFIRM
  ↓
Validate Stock
  ↓
Database Transaction
  ├── Reduce Stock
  ├── Create OUT Movements
  ├── Update Challan
  └── Commit
  ↓
CONFIRMED
```

If stock is insufficient:

``` text
Validate
  ↓
Reject
  ↓
No stock change
  ↓
Return API error
```

------------------------------------------------------------------------

## 13. Stock Business Rules

For an OUT operation:

``` text
currentStock >= requestedQuantity
```

If the condition is false:

-   Reject the operation
-   Do not reduce stock
-   Return a clear error

Example:

``` json
{
  "success": false,
  "message": "Insufficient stock",
  "details": {
    "productId": "123",
    "requested": 10,
    "available": 6
  }
}
```

The exact HTTP status should follow the project's final API convention.

------------------------------------------------------------------------

## 14. Challan Confirmation Transaction

Confirmation must be atomic.

``` text
BEGIN
  ↓
Validate challan state
  ↓
Load items
  ↓
Validate products
  ↓
Check stock for every item
  ↓
Reduce stock
  ↓
Create OUT movements
  ↓
Set challan = CONFIRMED
  ↓
COMMIT
```

If any step fails:

``` text
ROLLBACK
```

This prevents partial inventory updates.

------------------------------------------------------------------------

## 15. Product Snapshot

Challan items must preserve relevant product information instead of
relying only on the current product record.

Example:

``` json
{
  "productId": "123",
  "productName": "USB-C Adapter",
  "sku": "USB-C-001",
  "unitPrice": 250,
  "quantity": 4
}
```

The exact snapshot fields will be finalized with the database schema.

------------------------------------------------------------------------

## 16. Validation

Validate every request received from external clients.

Examples:

-   Required fields
-   Email format
-   Mobile format where appropriate
-   Enum values
-   Positive quantities
-   Product/customer IDs
-   Pagination parameters
-   Challan state

Client-side validation improves UX.

Backend validation protects the application.

------------------------------------------------------------------------

## 17. Error Handling

Use centralized error handling.

``` text
Service Error
     ↓
Error Middleware
     ↓
Consistent JSON Response
```

Success:

``` json
{
  "success": true,
  "data": {}
}
```

Error:

``` json
{
  "success": false,
  "message": "Customer not found"
}
```

Validation error:

``` json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email"
    }
  ]
}
```

Do not expose stack traces or secrets to clients.

------------------------------------------------------------------------

## 18. HTTP Status Codes

Use meaningful and consistent status codes.

``` text
200 OK
201 Created
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
500 Internal Server Error
```

The final API convention should remain consistent across modules.

------------------------------------------------------------------------

## 19. Search, Filtering and Pagination

Prioritize:

### Customers

-   Name/business/mobile search
-   Status/type filtering
-   Pagination

### Products

-   Name/SKU search
-   Category filtering
-   Low-stock filtering
-   Pagination

### Challans

-   Challan number/customer search
-   Status filtering
-   Pagination

### Stock Movements

-   Product filtering
-   Movement type filtering
-   Date filtering
-   Pagination

Keep filtering simple and requirement-driven.

------------------------------------------------------------------------

## 20. API Response Format

### Single resource

``` json
{
  "success": true,
  "data": {}
}
```

### Collection

``` json
{
  "success": true,
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 50,
    "totalPages": 3
  }
}
```

### Error

``` json
{
  "success": false,
  "message": "Insufficient stock"
}
```

------------------------------------------------------------------------

## 21. Environment Configuration

Example:

``` env
DATABASE_URL=
JWT_SECRET=
PORT=
NODE_ENV=
CORS_ORIGIN=
```

Never commit secrets.

Provide:

``` text
.env.example
```

------------------------------------------------------------------------

## 22. CORS

Configure CORS for:

-   Local frontend development
-   Deployed frontend

Production should use the actual frontend origin rather than allowing
all origins unnecessarily.

------------------------------------------------------------------------

## 23. Logging

Useful logs include:

-   Server startup
-   API errors
-   Business-operation failures
-   Database errors

Never log:

-   Passwords
-   JWT secrets
-   Sensitive credentials

------------------------------------------------------------------------

## 24. API Documentation

Provide a Postman collection or equivalent documentation containing:

-   Endpoint
-   HTTP method
-   Authentication
-   Request body
-   Query parameters
-   Successful response
-   Error response

------------------------------------------------------------------------

## 25. Development Order

### Phase 1 --- Foundation

-   Node.js
-   TypeScript
-   Express
-   Environment configuration
-   PostgreSQL connection
-   Prisma
-   Error handling

### Phase 2 --- Authentication

-   User model
-   Login
-   Password hashing
-   JWT
-   Authentication middleware
-   Role middleware

### Phase 3 --- CRM

-   Customer CRUD
-   Search
-   Pagination
-   Follow-ups

### Phase 4 --- Inventory

-   Product CRUD
-   Inventory
-   Stock movement

### Phase 5 --- Challans

-   Challan CRUD
-   Product snapshots
-   Stock validation
-   Transaction
-   Confirm/cancel workflow

### Phase 6 --- Finalization

-   API testing
-   Error refinement
-   Postman collection
-   Deployment configuration

------------------------------------------------------------------------

## 26. Scope Boundary

The backend is not intended to become a complete enterprise ERP.

Do not implement unrelated modules such as:

-   Payroll
-   Full accounting
-   Supplier management
-   Advanced procurement
-   Complex taxation
-   Full invoicing
-   Multi-company accounting

unless required by the final agreed scope.

The priority is:

> **Clean APIs + correct business logic + sound database interaction +
> understandable architecture.**
