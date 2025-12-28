# BillingScholarshipController API Documentation

## Endpoint: `/api/billing-scholarships`

### List Billing Scholarships (Paginated)
- **GET** `/api/billing-scholarships`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Paginated list of billing scholarships in various formats.

### Get Billing Scholarship by ID
- **GET** `/api/billing-scholarships/{id}`
- **Response:**
  - Billing scholarship details for the given ID.

### List Scholarships for a Billing
- **GET** `/api/billing-scholarships/billing/{billingId}`
- **Response:**
  - List of scholarships applied to the specified billing.

### List Billings Using a Scholarship
- **GET** `/api/billing-scholarships/scholarship/{scholarshipId}`
- **Response:**
  - List of billings using the specified scholarship.

### Apply Scholarship to Billing
- **POST** `/api/billing-scholarships`
- **Body:** `BillingScholarshipRequest`
- **Response:**
  - Created billing scholarship details.

### Update Billing Scholarship
- **PUT** `/api/billing-scholarships/{id}`
- **Body:** `BillingScholarshipRequest`
- **Response:**
  - Updated billing scholarship details.

### Delete Billing Scholarship
- **DELETE** `/api/billing-scholarships/{id}`
- **Response:**
  - Success message (soft delete).

### Show Billing Scholarships (Advanced)
- **GET** `/api/billing-scholarships/show`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `academic-years` (list of int, optional)
- **Response:**
  - Paginated, detailed list of billing scholarships with related billing, user, and month data.

---

## Business Logic
- Handles CRUD operations for billing scholarships.
- Supports paginated and formatted listing.
- Provides endpoints for advanced queries and relations (billings, users, months, etc.).
- Uses service and repository layers for all business logic and data access.
- Logs activities for auditing.
