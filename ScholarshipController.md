# ScholarshipController API Documentation

## Endpoint: `/api/scholarships`

### List Scholarships (Paginated)
- **GET** `/api/scholarships`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Paginated list of scholarships in various formats.

### Get Scholarship by ID
- **GET** `/api/scholarships/{id}`
- **Response:**
  - Scholarship details for the given ID.

### Create Scholarship
- **POST** `/api/scholarships`
- **Body:** `ScholarshipRequest`
- **Response:**
  - Created scholarship details.

### Update Scholarship
- **PUT** `/api/scholarships/{id}`
- **Body:** `ScholarshipRequest`
- **Response:**
  - Updated scholarship details.

### Delete Scholarship (Soft Delete)
- **DELETE** `/api/scholarships/{id}`
- **Response:**
  - Success message (scholarship deleted).

---

## Business Logic
- Handles CRUD operations for scholarships (beasiswa).
- Supports paginated and formatted listing.
- Uses service layer for all business logic and data access.
- Soft deletes scholarships on delete.
