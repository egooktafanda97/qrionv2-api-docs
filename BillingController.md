# BillingController API Documentation

## Endpoint: `/api/billing`

### Create Billing
- **POST** `/api/billing`
- **Body:** `BillingRequest`
- **Auth:** Required
- **Response:**
  - Created billing details.

### Update Billing
- **PUT** `/api/billing/{id}`
- **Body:** `BillingRequest`
- **Auth:** Required
- **Response:**
  - Updated billing details.

### List Billings (Paginated)
- **GET** `/api/billing`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Auth:** Required
- **Response:**
  - Paginated list of billings in various formats.

### Get Billing by ID
- **GET** `/api/billing/{id}`
- **Auth:** Required
- **Response:**
  - Billing details for the given ID.

### Get Billing Payment Status by ID
- **GET** `/api/billing/{id}/payment-status`
- **Auth:** Required
- **Response:**
  - Payment status for the given billing ID.

### Get Billing by UUID
- **GET** `/api/billing/uuid/{uuid}`
- **Auth:** Required
- **Response:**
  - Billing details for the given UUID.

### Get Billing Payment Status by UUID
- **GET** `/api/billing/uuid/{uuid}/payment-status`
- **Auth:** Required
- **Response:**
  - Payment status for the given billing UUID.

### Delete Billing
- **DELETE** `/api/billing/{id}`
- **Auth:** Required
- **Response:**
  - No content (success message).

### Generate Monthly Billing
- **POST** `/api/billing/generate-monthly`
- **Body:** `BillingGenerateRequest`
- **Auth:** Required
- **Response:**
  - Generated monthly billing details.

### Add User to Billing
- **POST** `/api/billing/add-user`
- **Body:** `AddUserToBillingRequest`
- **Auth:** Required
- **Response:**
  - Result of adding user to billing.

---

## Business Logic
- Handles CRUD operations for billings.
- Supports paginated and formatted listing.
- Provides endpoints for payment status and user management in billing.
- Uses service and repository layers for all business logic and data access.
- Requires authentication and uses user context for yayasan and institution.
