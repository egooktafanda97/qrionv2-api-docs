# StudentController API Documentation

## Base Path
`/api/students`

---

## Endpoints

### 1. List Students
- **URL:** `GET /api/students`
- **Query Parameters:**
  - `page` (int, default: 0): Page number
  - `size` (int, default: 10): Page size
  - `sortBy` (string, default: "id"): Field to sort by
  - `sortDirection` (string, default: "DESC"): Sort direction (ASC/DESC)
  - `draw` (int, optional): For DataTables integration
  - `format` (string, default: "standard"): Response format (`standard`, `jquery-datatable`, `ant-table`)
- **Responses:**
  - Paginated list of students
- **Business Logic:**
  - Retrieves paginated, sorted list of students from the service
  - Returns data in requested format

---

### 2. Paginated Search Students
- **URL:** `GET /api/students/search`
- **Query Parameters:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (as above)
  - `search` (string, optional): Free-text search
  - `nis`, `name`, `uuid`, `phone`, `parentName` (string, optional): Field filters
  - `academicYearId`, `classId`, `subClassId` (int, optional): Filter by related entities
- **Responses:**
  - Paginated, filtered list of students with related entities (institution, yayasan, user, parent, latest enrollment)
- **Business Logic:**
  - Filters students by provided fields and/or free-text search
  - Returns enriched student data with relations and latest enrollment/class info

---

### 3. Compact List by Academic Year
- **URL:** `GET /api/students/compact`
- **Query Parameters:**
  - `academicYearId` (int, required): Academic year to filter
- **Responses:**
  - Non-paginated list of students with class/subclass display
- **Business Logic:**
  - Returns compact student info for a given academic year, including class display

---

### 4. Get Current Student Profile
- **URL:** `GET /api/students/me`
- **Responses:**
  - Current logged-in student's profile
- **Business Logic:**
  - Fetches student profile for the authenticated user

---

### 5. Get Student by ID
- **URL:** `GET /api/students/{id}`
- **Path Variable:**
  - `id` (int): Student ID
- **Responses:**
  - Student details
- **Business Logic:**
  - Fetches student by ID using the service

---

### 6. Create Student
- **URL:** `POST /api/students`
- **Request Body:**
  - `StudentRequest` (validated): Student data
- **Responses:**
  - Created student details
- **Business Logic:**
  - Validates and creates a new student via the service

---

### 7. Update Student
- **URL:** `PUT /api/students/{id}`
- **Path Variable:**
  - `id` (int): Student ID
- **Request Body:**
  - `StudentUpdateRequest` (validated): Updated student data
- **Responses:**
  - Updated student details
- **Business Logic:**
  - Updates the specified student via the service

---

### 8. Delete Student
- **URL:** `DELETE /api/students/{id}`
- **Path Variable:**
  - `id` (int): Student ID
- **Responses:**
  - No content (success message)
- **Business Logic:**
  - Deletes the specified student via the service

---

### 9. Bulk Create Students
- **URL:** `POST /api/students/bulk`
- **Request Body:**
  - `StudentBulkCreateRequest` (validated): List of students to create
- **Responses:**
  - Bulk operation result (success/failure counts)
- **Business Logic:**
  - Creates multiple students in bulk (e.g., from Excel/CSV import)

---

### 10. Bulk Update Students
- **URL:** `PUT /api/students/bulk`
- **Request Body:**
  - `StudentBulkUpdateRequest` (validated): List of students to update
- **Responses:**
  - Bulk operation result (success/failure counts)
- **Business Logic:**
  - Updates multiple students in bulk

---

### 11. Bulk Delete Students
- **URL:** `DELETE /api/students/bulk`
- **Request Body:**
  - `StudentBulkDeleteRequest` (validated): List of students to delete
- **Responses:**
  - Bulk operation result (success/failure counts)
- **Business Logic:**
  - Deletes multiple students in bulk

---

## Data Models
- `StudentRequest`, `StudentUpdateRequest`: Input data for creating/updating a student
- `StudentBulkCreateRequest`, `StudentBulkUpdateRequest`, `StudentBulkDeleteRequest`: Bulk operations
- `StudentResponse`: Output data for student details
- `StudentBulkResponse`: Bulk operation result

## Notes
- All endpoints return standardized API responses.
- Business logic is delegated to `StudentService` and related repositories.
- Validation is enforced on request bodies.
- Some endpoints require authentication and use the current user's context.
