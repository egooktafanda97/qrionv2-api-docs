# BillingByAcademicYearController API Documentation

## Endpoint: `/api/billing-by-academic-year`

### Get General Billings by Academic Year
- **GET** `/api/billing-by-academic-year/general`
- **Query Params:**
  - `academicYearId` (int, required)
- **Auth:** Required
- **Response:**
  - List of general billings and their user billings for the specified academic year.

### Get Monthly Billings by Academic Year
- **GET** `/api/billing-by-academic-year/monthly`
- **Query Params:**
  - `academicYearId` (int, required)
- **Auth:** Required
- **Response:**
  - List of monthly billings and their user billings for the specified academic year.

### Get All Billings by Academic Year
- **GET** `/api/billing-by-academic-year/all`
- **Query Params:**
  - `academicYearId` (int, required)
- **Auth:** Required
- **Response:**
  - List of all billings (general, monthly, etc.) and their user billings for the specified academic year, including student and enrollment details.

---

## Business Logic
- Uses authenticated user to determine yayasan and institution context.
- Fetches billings by type and academic year.
- Aggregates user billing, student, and enrollment/class/subclass data for each billing.
- Uses repository layer for all business logic and data access.
- Throws errors for missing permissions or context.
