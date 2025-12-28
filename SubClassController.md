# SubClassController API Documentation

## Base Path
`/api/sub-classes`

---

## Endpoints

### 1. List Sub Classes
- **URL:** `GET /api/sub-classes`
- **Query Parameters:**
  - `page` (int, default: 0): Page number
  - `size` (int, default: 10): Page size
  - `sortBy` (string, default: "id"): Field to sort by
  - `sortDirection` (string, default: "DESC"): Sort direction (ASC/DESC)
  - `draw` (int, optional): For DataTables integration
  - `format` (string, default: "standard"): Response format (`standard`, `jquery-datatable`, `ant-table`)
- **Responses:**
  - Paginated list of sub classes
- **Business Logic:**
  - Retrieves paginated, sorted list of sub classes from the service
  - Returns data in requested format

---

### 2. Get Sub Classes by Class ID (List)
- **URL:** `GET /api/sub-classes/by-class/{classId}/list`
- **Path Variable:**
  - `classId` (int): Class ID
- **Responses:**
  - List of sub classes for the given class
- **Business Logic:**
  - Fetches all sub classes for a class (no pagination)

---

### 3. Get Sub Class by ID
- **URL:** `GET /api/sub-classes/{id}`
- **Path Variable:**
  - `id` (int): Sub class ID
- **Responses:**
  - Sub class details
- **Business Logic:**
  - Fetches sub class by ID using the service

---

### 4. Get Sub Classes by Class ID (Paginated)
- **URL:** `GET /api/sub-classes/by-class/{classId}`
- **Path Variable:**
  - `classId` (int): Class ID
- **Query Parameters:**
  - `page` (int, default: 0): Page number
  - `size` (int, default: 20): Page size
  - `sortBy` (string, default: "id"): Field to sort by
  - `sortDirection` (string, default: "ASC"): Sort direction (ASC/DESC)
  - `format` (string, default: "standard"): Response format
- **Responses:**
  - Paginated list of sub classes for the given class
- **Business Logic:**
  - Fetches paginated sub classes for a class

---

### 5. Create Sub Class
- **URL:** `POST /api/sub-classes`
- **Request Body:**
  - `SubClassRequest` (validated): Sub class data
- **Responses:**
  - Created sub class details
- **Business Logic:**
  - Validates and creates a new sub class via the service

---

### 6. Update Sub Class
- **URL:** `PUT /api/sub-classes/{id}`
- **Path Variable:**
  - `id` (int): Sub class ID
- **Request Body:**
  - `SubClassRequest` (validated): Updated sub class data
- **Responses:**
  - Updated sub class details
- **Business Logic:**
  - Updates the specified sub class via the service

---

### 7. Delete Sub Class
- **URL:** `DELETE /api/sub-classes/{id}`
- **Path Variable:**
  - `id` (int): Sub class ID
- **Responses:**
  - No content (success message)
- **Business Logic:**
  - Deletes the specified sub class via the service

---

## Data Models
- `SubClassRequest`: Input data for creating/updating a sub class
- `SubClassResponse`: Output data for sub class details

## Notes
- All endpoints return standardized API responses.
- Business logic is delegated to `SubClassService`.
- Validation is enforced on request bodies.
