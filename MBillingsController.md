# MBillingsController API Documentation

## Endpoint: `/api/m-billings`

### Create Master Billing
- **POST** `/api/m-billings`
- **Body:** `MBillingsRequest`
- **Auth:** Required
- **Response:**
  - Created master billing details.

### Update Master Billing
- **PUT** `/api/m-billings/{id}`
- **Body:** `MBillingsRequest`
- **Auth:** Required
- **Response:**
  - Updated master billing details.

### List Master Billings (Paginated)
- **GET** `/api/m-billings`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Auth:** Required
- **Response:**
  - Paginated list of master billings in various formats.

### Get Master Billing by ID
- **GET** `/api/m-billings/{id}`
- **Auth:** Required
- **Response:**
  - Master billing details for the given ID.

### Get Master Billing by UUID
- **GET** `/api/m-billings/uuid/{uuid}`
- **Auth:** Required
- **Response:**
  - Master billing details for the given UUID.

### Update Monthly Active
- **PATCH** `/api/m-billings/monthly-active`
- **Body:** `MBillingMonthlyActiveUpdateRequest`
- **Auth:** Required
- **Response:**
  - Updated master billing with new monthly active months.

### Disable Master Billing
- **PATCH** `/api/m-billings/{id}/disable`
- **Auth:** Required
- **Response:**
  - Master billing disabled.

### Enable Master Billing
- **PATCH** `/api/m-billings/{id}/enable`
- **Auth:** Required
- **Response:**
  - Master billing enabled.

### Delete Master Billing
- **DELETE** `/api/m-billings/{id}`
- **Auth:** Required
- **Response:**
  - No content (success message).

### List All Master Billings (No Pagination)
- **GET** `/api/m-billings/show`
- **Query Params:**
  - `tahun-ajaran` (int, optional)
- **Auth:** Required
- **Response:**
  - List of all master billings with details, no pagination.

### List Monthly Master Billings (No Pagination)
- **GET** `/api/m-billings/show/monthly`
- **Query Params:**
  - `tahun-ajaran` (int, optional)
- **Auth:** Required
- **Response:**
  - List of monthly master billings with details, no pagination.

### List General Master Billings (No Pagination)
- **GET** `/api/m-billings/show/general`
- **Query Params:**
  - `tahun-ajaran` (int, optional)
- **Auth:** Required
- **Response:**
  - List of general (non-monthly) master billings with details, no pagination.

---

## Business Logic
- Handles CRUD operations for master billings.
- Supports paginated and non-paginated listing, filtering by academic year.
- Allows enabling/disabling, updating monthly active months, and deleting billings.
- Uses service and repository layers for all business logic and data access.
- Requires authentication and yayasan/institution context.
