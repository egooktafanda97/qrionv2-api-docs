# SemesterController API Documentation

## Base Path
`/api/semesters`

---

## Endpoints

### 1. List Semesters
- **URL:** `GET /api/semesters`
- **Query Parameters:**
  - `page` (int, default: 0): Page number
  - `size` (int, default: 10): Page size
  - `sortBy` (string, default: "id"): Field to sort by
  - `sortDirection` (string, default: "DESC"): Sort direction (ASC/DESC)
  - `draw` (int, optional): For DataTables integration
  - `format` (string, default: "standard"): Response format (`standard`, `jquery-datatable`, `ant-table`)
- **Responses:**
  - Standard paginated list of semesters
  - Supports jQuery DataTable and Ant Design Table formats
- **Business Logic:**
  - Retrieves paginated, sorted list of semesters from the service
  - Returns data in requested format

---

### 2. Get Semester by ID
- **URL:** `GET /api/semesters/{id}`
- **Path Variable:**
  - `id` (int): Semester ID
- **Responses:**
  - Semester details
- **Business Logic:**
  - Fetches semester by ID using the service

---

### 3. Create Semester
- **URL:** `POST /api/semesters`
- **Request Body:**
  - `SemesterRequest` (validated): Semester data
- **Responses:**
  - Created semester details
- **Business Logic:**
  - Validates and creates a new semester via the service

---

### 4. Update Semester
- **URL:** `PUT /api/semesters/{id}`
- **Path Variable:**
  - `id` (int): Semester ID
- **Request Body:**
  - `SemesterRequest` (validated): Updated semester data
- **Responses:**
  - Updated semester details
- **Business Logic:**
  - Updates the specified semester via the service

---

### 5. Delete Semester
- **URL:** `DELETE /api/semesters/{id}`
- **Path Variable:**
  - `id` (int): Semester ID
- **Responses:**
  - No content (success message)
- **Business Logic:**
  - Deletes the specified semester via the service

---

### 6. Activate Semester
- **URL:** `PUT /api/semesters/{id}/activate`
- **Path Variable:**
  - `id` (int): Semester ID
- **Responses:**
  - Activated semester details
- **Business Logic:**
  - Activates the specified semester via the service

---

## Data Models
- `SemesterRequest`: Input data for creating/updating a semester
- `SemesterResponse`: Output data for semester details

## Notes
- All endpoints return standardized API responses.
- Business logic is delegated to `SemesterService`.
- Validation is enforced on request bodies.
