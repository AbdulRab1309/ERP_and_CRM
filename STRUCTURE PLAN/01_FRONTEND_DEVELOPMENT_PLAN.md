# Frontend Development Plan

## 1. Project Overview

**Project:** Mini ERP + CRM Operations Portal

The frontend is a responsive internal operations interface for managing
customers, products, inventory, and sales challans.

The implementation prioritizes:

-   Functional business workflows
-   Clear information hierarchy
-   Maintainable React architecture
-   Responsive design
-   Minimal visual complexity
-   Consistent user experience

The UI should feel purpose-built rather than like a generic AI-generated
dashboard.

------------------------------------------------------------------------

## 2. Design Direction

The visual direction is inspired by the provided **Oranssi Fluid**
reference, adapted for an ERP/CRM application.

### Design principles

-   Strong typography
-   Clear hierarchy
-   High contrast
-   Generous spacing
-   Restrained color palette
-   Purposeful cards and sections
-   Clean tables and forms
-   Minimal decorative elements
-   Consistent spacing and interaction patterns

### Avoid

-   Excessive gradients
-   Glassmorphism
-   3D graphics
-   Unnecessary animations
-   Decorative dashboards
-   Fake analytics
-   Excessive charts
-   Unnecessary UI elements
-   Features outside the assignment scope

> **The interface should communicate the business workflow clearly
> rather than compete with it visually.**

------------------------------------------------------------------------

## 3. Technology Stack

  Technology        Purpose
  ----------------- ---------------------------
  React             Frontend framework
  TypeScript        Type safety
  Vite              Development/build tooling
  Tailwind CSS      Styling
  React Router      Routing
  TanStack Query    Server-state management
  React Hook Form   Form management
  Zod               Client-side validation
  Lucide React      UI icons

The stack should remain intentionally lightweight. Additional
dependencies should only be introduced when they solve a clear
requirement.

------------------------------------------------------------------------

## 4. Application Structure

``` text
Login
  │
  ▼
Application
  ├── Overview
  ├── Customers
  ├── Products
  ├── Inventory
  └── Sales Challans
```

### Primary navigation

-   Overview
-   Customers
-   Products
-   Inventory
-   Sales Challans

Navigation and available actions should reflect the authenticated user's
role.

------------------------------------------------------------------------

## 5. Pages

### 5.1 Login

Requirements:

-   Application identity
-   Email/username
-   Password
-   Login action
-   Loading state
-   Authentication error
-   Network/API error

Authentication is handled by the backend.

------------------------------------------------------------------------

### 5.2 Overview

The overview page should provide a concise operational summary.

Suggested information:

-   Customer count
-   Product count
-   Low-stock items
-   Draft challans
-   Upcoming follow-ups
-   Recent activity

Avoid creating statistics or charts that are not supported by actual
application data.

------------------------------------------------------------------------

### 5.3 Customers

#### Customer List

Required functionality:

-   List customers
-   Add customer
-   Edit customer
-   Search
-   Filter where useful
-   View customer details

Fields:

-   Customer name
-   Business name
-   Mobile
-   Customer type
-   Status
-   Follow-up date

Customer types:

-   Retail
-   Wholesale
-   Distributor

Customer statuses:

-   Lead
-   Active
-   Inactive

#### Customer Detail

Display:

-   Customer information
-   Contact information
-   Business information
-   GST number
-   Customer type
-   Status
-   Address
-   Follow-up date
-   Notes
-   Follow-up history

Actions:

-   Edit customer
-   Add follow-up note
-   Update follow-up date

------------------------------------------------------------------------

### 5.4 Products

#### Product List

Display:

-   Product name
-   SKU/code
-   Category
-   Unit price
-   Current stock
-   Minimum stock
-   Warehouse/location
-   Stock status

Actions:

-   Add product
-   Edit product
-   View product

Stock states:

-   In Stock
-   Low Stock
-   Out of Stock

------------------------------------------------------------------------

### 5.5 Inventory

The inventory interface focuses on stock visibility and movement
history.

#### Inventory View

Display:

-   Product
-   SKU
-   Current stock
-   Minimum stock
-   Location
-   Stock status

#### Stock Movement Log

Display:

-   Product
-   Quantity
-   Movement type
-   Reason
-   Created by
-   Timestamp

Movement types:

-   IN
-   OUT

------------------------------------------------------------------------

### 5.6 Sales Challans

#### Challan List

Display:

-   Challan number
-   Customer
-   Total quantity
-   Status
-   Created by
-   Created date

Statuses:

-   Draft
-   Confirmed
-   Cancelled

#### Create Challan

Workflow:

``` text
Select Customer
      ↓
Add Products
      ↓
Enter Quantities
      ↓
Review
      ↓
Save Draft / Confirm
      ↓
Backend Validation
      ↓
Success / Error
```

Each item should show the relevant product and available stock.

When confirming:

-   Backend performs the authoritative stock check.
-   The UI communicates that stock will be reduced.
-   Insufficient stock errors are displayed clearly.
-   The frontend must not assume that its own stock calculation is
    authoritative.

Example:

``` text
Insufficient stock

USB-C Adapter
Requested: 12
Available: 7

The challan was not confirmed.
```

------------------------------------------------------------------------

## 6. Authentication and Roles

Required roles:

-   Admin
-   Sales
-   Warehouse
-   Accounts

The frontend should:

-   Maintain authentication state
-   Store the authenticated user's role
-   Protect application routes
-   Show role-appropriate navigation/actions

Frontend role checks are for user experience only.

> **Backend authorization remains the security boundary.**

------------------------------------------------------------------------

## 7. Forms

Forms should provide:

-   Clear labels
-   Required-field indicators
-   Client-side validation
-   Inline errors
-   Loading states
-   Submit states
-   Cancel actions where applicable

The frontend should never rely on client-side validation as the only
validation layer.

------------------------------------------------------------------------

## 8. API Integration

API requests should be centralized rather than scattered throughout
components.

``` text
src/
└── api/
    ├── client.ts
    ├── auth.ts
    ├── customers.ts
    ├── products.ts
    ├── inventory.ts
    └── challans.ts
```

The frontend communicates with the backend through REST APIs.

------------------------------------------------------------------------

## 9. Data States

Every major data-driven page should handle:

### Loading

``` text
Loading customers...
```

### Empty

``` text
No customers found.

[Add Customer]
```

### Error

``` text
Unable to load customers.

[Retry]
```

### Validation/API Error

Display useful backend errors in a clear user-facing format.

------------------------------------------------------------------------

## 10. Responsive Design

The application should support:

-   Desktop
-   Laptop
-   Tablet
-   Smaller screens where practical

The primary target is a desktop/laptop internal operations application.

------------------------------------------------------------------------

## 11. Reusable Components

``` text
components/
├── layout/
│   ├── Sidebar
│   ├── Header
│   └── PageContainer
│
├── ui/
│   ├── Button
│   ├── Input
│   ├── Select
│   ├── Modal
│   ├── Badge
│   ├── Table
│   ├── EmptyState
│   ├── LoadingState
│   └── ErrorState
│
├── customers/
├── products/
├── inventory/
└── challans/
```

Components should be created when they provide genuine reuse or improve
maintainability.

------------------------------------------------------------------------

## 12. Recommended Folder Structure

``` text
src/
├── api/
├── assets/
├── components/
│   ├── layout/
│   ├── ui/
│   ├── customers/
│   ├── products/
│   ├── inventory/
│   └── challans/
├── hooks/
├── layouts/
├── pages/
│   ├── Login/
│   ├── Dashboard/
│   ├── Customers/
│   ├── Products/
│   ├── Inventory/
│   └── Challans/
├── routes/
├── types/
├── utils/
├── App.tsx
└── main.tsx
```

------------------------------------------------------------------------

## 13. Development Order

### Phase 1 --- Foundation

-   Vite/React/TypeScript setup
-   Tailwind setup
-   Global styles
-   Layout
-   Navigation
-   Routing
-   Login UI

### Phase 2 --- Authentication

-   Login API integration
-   Authentication state
-   Protected routes
-   Role-aware navigation

### Phase 3 --- CRM

-   Customer list
-   Customer search/filter
-   Customer form
-   Customer detail
-   Follow-ups

### Phase 4 --- Products & Inventory

-   Product list
-   Product form
-   Inventory view
-   Stock movement log
-   Stock status

### Phase 5 --- Challans

-   Challan list
-   Challan detail
-   Challan creation
-   Draft workflow
-   Confirmation workflow
-   Error handling

### Phase 6 --- Finalization

-   Loading states
-   Empty states
-   Error states
-   Responsive adjustments
-   API integration refinement
-   Visual consistency
-   Final testing

------------------------------------------------------------------------

## 14. Scope Boundary

The frontend will not attempt to become a complete enterprise ERP.

Do not add unnecessary:

-   Analytics systems
-   Complex reporting
-   Payroll
-   Supplier management
-   Full accounting interfaces
-   Advanced invoicing
-   Complex dashboards
-   Decorative marketing sections

The priority is a **complete, functional, explainable implementation of
the assignment requirements**.
