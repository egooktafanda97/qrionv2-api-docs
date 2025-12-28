# TeacherController API Documentation

## Base Path
`/api/teachers`

---

## Endpoints

### 1. Create Teacher
- **URL:** `POST /api/teachers`
- **Request Body:**
  - `CreateTeacherRequest` (validated): Teacher data
- **Responses:**
  - Created teacher details (with user and wallet account)
- **Business Logic:**
  - Creates a new teacher, auto-creates a user account (role: TEACHER) and wallet
  - Activity is logged at INFO level

---

### 2. Get All Teachers
- **URL:** `GET /api/teachers`
- **Responses:**
  - List of all teachers for the authenticated user's yayasan and institution
- **Business Logic:**
  - Retrieves all teachers for the current yayasan and institution
  - Activity is logged at INFO level

---

### 3. Get Teacher by ID
- **URL:** `GET /api/teachers/{id}`
- **Path Variable:**
  - `id` (int): Teacher ID
- **Responses:**
  - Teacher details
- **Business Logic:**
  - Fetches teacher by ID
  - Activity is logged at INFO level

---

### 4. Update Teacher
- **URL:** `PUT /api/teachers/{id}`
- **Path Variable:**
  - `id` (int): Teacher ID
- **Request Body:**
  - `UpdateTeacherRequest` (validated): Updated teacher data
- **Responses:**
  - Updated teacher details
- **Business Logic:**
  - Updates teacher information
  - Activity is logged at INFO level

---

### 5. Delete Teacher (Soft Delete)
- **URL:** `DELETE /api/teachers/{id}`
- **Path Variable:**
  - `id` (int): Teacher ID
- **Responses:**
  - Success message ("Teacher berhasil dihapus.")
- **Business Logic:**
  - Soft deletes the specified teacher
  - Activity is logged at INFO level

---

## Data Models
- `CreateTeacherRequest`: Input data for creating a teacher
- `UpdateTeacherRequest`: Input data for updating a teacher
- `TeacherResponse`: Output data for teacher details

## Notes
- All endpoints return standardized API responses.
- Business logic is delegated to `TeacherService`.
- Activity logging is applied to all endpoints.
- Validation is enforced on request bodies.
