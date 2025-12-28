# AcademicYearController API Documentation

## Endpoint: `/api/academic-years`

### List Academic Years
- **GET** `/api/academic-years`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Returns paginated list of academic years in different formats based on `format` param.

### Get Academic Year by ID
- **GET** `/api/academic-years/{id}`
- **Response:**
  - Academic year details.

### Create Academic Year
- **POST** `/api/academic-years`
- **Body:** `AcademicYearRequest`
- **Response:**
  - Created academic year.

### Update Academic Year
- **PUT** `/api/academic-years/{id}`
- **Body:** `AcademicYearRequest`
- **Response:**
  - Updated academic year.

### Delete Academic Year
- **DELETE** `/api/academic-years/{id}`
- **Response:**
  - No content (success message).

### Activate Academic Year
- **PUT** `/api/academic-years/{id}/activate`
- **Response:**
  - Activated academic year.

---

## Business Logic
- Pagination, sorting, and response format switching.
- CRUD operations for academic years.
- Activation endpoint to set an academic year as active.
- Uses service layer for all business logic and data access.
