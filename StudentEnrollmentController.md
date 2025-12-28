# StudentEnrollmentController API Documentation

## Base Path
`/api/student-enrollments`

---

## Endpoints

### 1. List Student Enrollments
- **URL:** `GET /api/student-enrollments`
- **Query Parameters:**
  - `page` (int, default: 0): Page number
  - `size` (int, default: 10): Page size
  - `sortBy` (string, default: "id"): Field to sort by
  - `sortDirection` (string, default: "DESC"): Sort direction (ASC/DESC)
  - `draw` (int, optional): For DataTables integration
  - `format` (string, default: "standard"): Response format
- **Responses:**
  - Paginated list of student enrollments
- **Business Logic:**
  - Retrieves paginated, sorted list of enrollments for the current yayasan/institution

---

### 2. Get Enrollment by ID
- **URL:** `GET /api/student-enrollments/{id}`
- **Path Variable:**
  - `id` (long): Enrollment ID
- **Responses:**
  - Enrollment details
- **Business Logic:**
  - Fetches enrollment by ID

---

### 3. Get All Enrollments for Student
- **URL:** `GET /api/student-enrollments/student/{studentId}`
- **Path Variable:**
  - `studentId` (long): Student ID
- **Responses:**
  - List of enrollments for the student
- **Business Logic:**
  - Fetches all enrollments for a student

---

### 4. Get Enrollment by Student and Academic Year
- **URL:** `GET /api/student-enrollments/student/{studentId}/academic-year/{academicYearId}`
- **Path Variables:**
  - `studentId` (long): Student ID
  - `academicYearId` (int): Academic year ID
- **Responses:**
  - Enrollment details
- **Business Logic:**
  - Fetches enrollment for a student in a specific academic year

---

### 5. Create Enrollment
- **URL:** `POST /api/student-enrollments`
- **Request Body:**
  - `StudentEnrollmentRequest` (validated): Enrollment data
- **Responses:**
  - Created enrollment details
- **Business Logic:**
  - Creates a new student enrollment

---

### 6. Update Enrollment
- **URL:** `PUT /api/student-enrollments/{id}`
- **Path Variable:**
  - `id` (long): Enrollment ID
- **Request Body:**
  - `StudentEnrollmentUpdateRequest` (validated): Updated enrollment data
- **Responses:**
  - Updated enrollment details
- **Business Logic:**
  - Updates enrollment, records transfer history if class changed

---

### 7. Delete Enrollment
- **URL:** `DELETE /api/student-enrollments/{id}`
- **Path Variable:**
  - `id` (long): Enrollment ID
- **Responses:**
  - No content (success message)
- **Business Logic:**
  - Deletes the specified enrollment

---

### 8. Bulk Insert Enrollments
- **URL:** `POST /api/student-enrollments/bulk`
- **Request Body:**
  - `StudentEnrollmentBulkRequest` (validated): Bulk enrollment data
- **Responses:**
  - List of created enrollments
- **Business Logic:**
  - Registers multiple students to the same class

---

### 9. Bulk Transfer Students
- **URL:** `POST /api/student-enrollments/bulk-transfer`
- **Request Body:**
  - `StudentMutasiBulkRequest` (validated): Bulk transfer data
- **Responses:**
  - List of processed enrollments
- **Business Logic:**
  - Transfers multiple students to new sub-classes/academic years, records history

---

### 10. Bulk Assign to SubClass
- **URL:** `POST /api/student-enrollments/bulk-assign-subclass`
- **Request Body:**
  - `StudentEnrollmentBulkAssignSubClassRequest` (validated): Bulk assign data
- **Responses:**
  - List of processed enrollments
- **Business Logic:**
  - Assigns multiple students from class to sub-class

---

### 11. Get Enrollments by Academic Year
- **URL:** `GET /api/student-enrollments/by-academic-year/{academicYearId}`
- **Path Variable:**
  - `academicYearId` (int): Academic year ID
- **Query Parameters:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (as above)
- **Responses:**
  - Paginated list of enrollments for the academic year

---

### 12. Get Enrollments by Active Semester
- **URL:** `GET /api/student-enrollments/by-active-semester`
- **Query Parameters:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (as above)
- **Responses:**
  - Paginated list of enrollments for the active semester

---

### 13. Get Enrollments by Class
- **URL:** `GET /api/student-enrollments/by-class/{classId}`
- **Path Variable:**
  - `classId` (int): Class ID
- **Query Parameters:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (as above)
- **Responses:**
  - Paginated list of enrollments for the class

---

### 14. Get Enrollments by SubClass
- **URL:** `GET /api/student-enrollments/by-subclass/{subClassId}`
- **Path Variable:**
  - `subClassId` (int): SubClass ID
- **Query Parameters:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (as above)
- **Responses:**
  - Paginated list of enrollments for the sub-class

---

### 15. Transfer History Endpoints
- **List Transfer History:** `GET /api/student-enrollments/transfer-history` (paginated)
- **Get by ID:** `GET /api/student-enrollments/transfer-history/{id}`
- **Get by Student:** `GET /api/student-enrollments/transfer-history/student/{studentId}`
- **Get by Academic Year:** `GET /api/student-enrollments/transfer-history/academic-year/{academicYearId}`
- **Get by Student (Paginated):** `GET /api/student-enrollments/transfer-history/student/{studentId}/paginated`
- **Query Parameters:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (as above)
- **Business Logic:**
  - All transfer history endpoints fetch and return transfer/mutation records, with pagination where applicable

---

## Data Models
- `StudentEnrollmentRequest`, `StudentEnrollmentUpdateRequest`, `StudentEnrollmentBulkRequest`, `StudentMutasiBulkRequest`, `StudentEnrollmentBulkAssignSubClassRequest`: Input data for enrollment operations
- `StudentEnrollmentResponse`, `StudentTransferHistoryResponse`: Output data for enrollment and transfer history

## Notes
- All endpoints return standardized API responses.
- Business logic is delegated to `StudentEnrollmentService` and `StudentTransferHistoryService`.
- Validation is enforced on request bodies.
