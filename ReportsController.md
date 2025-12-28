# ReportsController API Documentation

## Endpoint: `/api/reports`

### Get Academic Year Reports
- **GET** `/api/reports/academic-years`
- **Query Params:**
  - `academicYears` (list of int, optional)
- **Response:**
  - `ReportResponse` containing a list of `AcademicYearReport` objects for the specified academic years (or all if not specified).

---

## Business Logic
- Fetches academic year reports, optionally filtered by academic year IDs.
- Uses service layer for all business logic and data access.
