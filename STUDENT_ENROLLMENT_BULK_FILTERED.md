# Student Enrollment Bulk Insert & Filtered Queries

## Overview
Implementasi fitur bulk insert untuk pendaftaran siswa dan query filtering berdasarkan berbagai kriteria dengan dukungan multi-tenant (yayasan & institusi).

## Date
November 27, 2025

## Features Implemented

### 1. Bulk Insert Student Enrollments
- **Endpoint**: `POST /api/student-enrollments/bulk`
- **Purpose**: Mendaftarkan beberapa siswa sekaligus ke kelas yang sama
- **Request Body**:
```json
{
  "studentId": [1, 2, 3, 4],
  "academicYearId": 1,
  "subClassId": 1,
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

#### Key Features:
- ✅ Accepts array of student IDs
- ✅ Automatically derives `classId` from `subClassId`
- ✅ Skips duplicate enrollments (same student + academic year)
- ✅ Validates all students belong to authenticated user's yayasan/institution
- ✅ Transaction-safe: all or nothing for each student
- ✅ Returns list of successfully created enrollments
- ✅ Logs warnings for skipped duplicates
- ✅ Validates academic year and subclass access

#### Business Logic:
1. Validate SubClass exists and belongs to user's institution
2. Get SchoolClass from SubClass relationship
3. Validate AcademicYear access
4. For each student:
   - Validate student exists and belongs to user's yayasan/institution
   - Check if enrollment already exists (skip if found)
   - Create enrollment with derived classId
   - Save and add to results
5. Return list of created enrollments

### 2. Get Enrollments by Academic Year
- **Endpoint**: `GET /api/student-enrollments/by-academic-year/{academicYearId}`
- **Purpose**: Mendapatkan semua enrollment untuk tahun akademik tertentu
- **Query Parameters**:
  - `page` (default: 0)
  - `size` (default: 10)
  - `sortBy` (default: "id")
  - `sortDirection` (default: "DESC")
  - `draw` (optional, for jQuery DataTable)
- **Headers**:
  - `format`: "standard" | "jquery-datatable" | "ant-table"

#### Features:
- ✅ Filtered by authenticated user's yayasan and institution
- ✅ Pagination support
- ✅ Sorting support
- ✅ Multiple response formats (3 formats)
- ✅ Validates academic year access

### 3. Get Enrollments by Active Semester
- **Endpoint**: `GET /api/student-enrollments/by-active-semester`
- **Purpose**: Mendapatkan enrollment untuk semester yang sedang aktif
- **Query Parameters**: Same as academic year endpoint

#### Features:
- ✅ Automatically finds active semester for user's institution
- ✅ Returns enrollments for that semester's academic year
- ✅ Filtered by yayasan and institution
- ✅ Returns empty page if no active semester
- ✅ Same pagination and format support

### 4. Get Enrollments by Class
- **Endpoint**: `GET /api/student-enrollments/by-class/{classId}`
- **Purpose**: Mendapatkan semua enrollment untuk kelas tertentu
- **Query Parameters**: Same as above

#### Features:
- ✅ Filtered by class ID
- ✅ Validates class belongs to user's institution
- ✅ Filtered by student's yayasan and institution
- ✅ Pagination and format support

### 5. Get Enrollments by SubClass
- **Endpoint**: `GET /api/student-enrollments/by-subclass/{subClassId}`
- **Purpose**: Mendapatkan semua enrollment untuk sub-kelas tertentu
- **Query Parameters**: Same as above

#### Features:
- ✅ Filtered by subclass ID
- ✅ Validates subclass exists
- ✅ Filtered by student's yayasan and institution
- ✅ Pagination and format support

## Files Created/Modified

### Created Files:
1. **StudentEnrollmentBulkRequest.java** (DTO)
   - List of studentIds
   - academicYearId
   - subClassId
   - enrolledAt
   - transferStatus (optional)
   - Jakarta Validation annotations

2. **StudentEnrollmentBulk.http** (HTTP Test File)
   - 25 comprehensive test cases
   - Covers success and error scenarios
   - Tests all 3 response formats
   - Tests pagination and sorting

### Modified Files:
1. **StudentEnrollmentService.java**
   - Added `createBulk()` method signature
   - Added filtered query method signatures:
     - `findByAcademicYear()`
     - `findByActiveSemester()`
     - `findByClass()`
     - `findBySubClass()`

2. **StudentEnrollmentServiceImpl.java**
   - Added SemesterRepository injection
   - Implemented `createBulk()` with:
     - SubClass validation and classId derivation
     - Duplicate detection and skipping
     - Transaction handling
     - Error logging
   - Implemented filtered query methods with:
     - Multi-tenant filtering
     - Pagination support
     - Student relationship filtering

3. **StudentEnrollmentController.java**
   - Added `POST /bulk` endpoint
   - Added filtered GET endpoints:
     - `/by-academic-year/{id}`
     - `/by-active-semester`
     - `/by-class/{id}`
     - `/by-subclass/{id}`
   - All support 3 response formats

## Security Implementation

### Multi-Tenant Filtering
All endpoints enforce security through:
1. **Direct Validation**: Academic year, class, subclass validated against user's institution
2. **Student Filtering**: Enrollments filtered by student's yayasan and institution
3. **Authenticated User Context**: Using `getCurrentUser()` from SecurityContext

### Access Control:
- User can only bulk insert students from their yayasan/institution
- User can only query enrollments for students in their yayasan/institution
- Academic years, classes, and subclasses validated against user's institution
- Active semester lookup filtered by user's yayasan and institution

## API Response Formats

### 1. Standard Format
```json
{
  "data": [...],
  "total": 100,
  "page": 0,
  "size": 10,
  "totalPages": 10,
  "hasNext": true,
  "hasPrevious": false
}
```

### 2. jQuery DataTable Format
```json
{
  "draw": 1,
  "recordsTotal": 100,
  "recordsFiltered": 100,
  "data": [...]
}
```

### 3. Ant Design Table Format
```json
{
  "data": [...],
  "success": true,
  "total": 100,
  "current": 1,
  "pageSize": 10
}
```

## Error Handling

### Bulk Insert Errors:
- **Empty student array**: Validation error from `@NotEmpty`
- **Invalid academicYearId**: Resource not found exception
- **Invalid subClassId**: Resource not found exception
- **Invalid studentId**: Skipped with warning log, continues with next student
- **Duplicate enrollment**: Skipped with warning log, continues with next student
- **No enrollments created**: Business rule violation exception
- **Access denied**: Permission denied exception for different yayasan/institution

### Query Errors:
- **Invalid academicYearId**: Resource not found exception
- **Invalid classId**: Resource not found exception
- **Invalid subClassId**: Resource not found exception
- **No active semester**: Returns empty page
- **Access denied**: Permission denied exception

## Usage Examples

### Example 1: Bulk Insert New Students
```bash
POST /api/student-enrollments/bulk
Content-Type: application/json
Authorization: Bearer {token}

{
  "studentId": [1, 2, 3, 4, 5],
  "academicYearId": 1,
  "subClassId": 1,
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

**Response**:
```json
{
  "success": true,
  "message": "Bulk insert berhasil. 5 dari 5 siswa berhasil didaftarkan",
  "data": [
    {
      "id": 101,
      "studentId": 1,
      "studentName": "Ahmad",
      "academicYearId": 1,
      "academicYearName": "2025/2026",
      "classId": 1,
      "className": "Kelas 10",
      "subClassId": 1,
      "subClassName": "A",
      "enrolledAt": "2025-07-01T08:00:00",
      "statusId": 1,
      "statusName": "Aktif"
    },
    // ... 4 more enrollments
  ]
}
```

### Example 2: Get Enrollments by Active Semester
```bash
GET /api/student-enrollments/by-active-semester?page=0&size=20
Authorization: Bearer {token}
format: standard
```

**Response**:
```json
{
  "data": [...],
  "total": 150,
  "page": 0,
  "size": 20,
  "totalPages": 8,
  "hasNext": true,
  "hasPrevious": false
}
```

### Example 3: Get Enrollments by Class with jQuery Format
```bash
GET /api/student-enrollments/by-class/1?page=0&size=10&draw=1
Authorization: Bearer {token}
format: jquery-datatable
```

**Response**:
```json
{
  "draw": 1,
  "recordsTotal": 30,
  "recordsFiltered": 30,
  "data": [...]
}
```

## Testing Guide

### Test File Location
`http/StudentEnrollmentBulk.http`

### Test Categories:
1. **Bulk Insert Tests** (Tests 1, 14, 15)
   - Basic bulk insert
   - Different student groups
   - Duplicate handling

2. **Filtered Query Tests** (Tests 2-13)
   - By academic year (all formats)
   - By active semester (all formats)
   - By class (all formats)
   - By subclass (all formats)

3. **Pagination Tests** (Tests 16-17)
   - Multiple pages
   - Page navigation

4. **Sorting Tests** (Tests 18-19)
   - Ascending order
   - Descending order

5. **Error Handling Tests** (Tests 20-25)
   - Invalid IDs
   - Empty arrays
   - Non-existent resources

## Performance Considerations

1. **Bulk Insert**:
   - Each student validated individually
   - Skips duplicates instead of failing entire batch
   - Transaction per student (not entire batch)
   - Logs warnings for better monitoring

2. **Filtered Queries**:
   - In-memory filtering after DB query (can be optimized with native queries)
   - Pagination applied after filtering
   - Efficient for small to medium datasets
   - Consider adding database-level filtering for large datasets

## Future Enhancements

### Potential Improvements:
1. **Database-level Filtering**: Create native queries with joins to filter at database level
2. **Batch Insert Optimization**: Use JPA batch insert for better performance
3. **Transfer Status Mapping**: Add proper status mapping table instead of hardcoded values
4. **Async Processing**: For very large bulk inserts, consider async processing with job queue
5. **Rollback Handling**: Add option for all-or-nothing transaction for entire bulk insert
6. **Progress Tracking**: Add progress updates for bulk inserts with many students
7. **Duplicate Strategy**: Add option to update instead of skip duplicates
8. **Transfer History**: Optionally create transfer history records during bulk insert

## Known Limitations

1. **In-Memory Filtering**: Current implementation filters students in memory, which may be slow for large datasets
2. **No Partial Rollback**: If one student fails, it's skipped but others continue (not atomic across all students)
3. **Transfer Status**: `transferStatus` field in request is not currently used (defaults to statusId=1)
4. **No Progress Callback**: Bulk insert doesn't provide progress updates for large batches

## Migration Notes

### Breaking Changes:
None - All new endpoints, existing functionality unchanged

### Database Changes:
None - Uses existing tables

### Configuration Changes:
None required

## Compilation Status
✅ **BUILD SUCCESS** - Compiled successfully with no errors

### Warnings (Non-critical):
- `@Builder` initialization warnings (Lombok configuration warnings, not errors)
- Deprecated API usage in DataFactory (existing code)

## Conclusion

Successfully implemented:
- ✅ Bulk insert API with duplicate handling
- ✅ 4 filtered query endpoints with pagination
- ✅ Support for 3 response formats
- ✅ Multi-tenant security enforcement
- ✅ Comprehensive HTTP test file (25 tests)
- ✅ Transaction safety
- ✅ Error handling and validation
- ✅ Documentation

All features compile successfully and are ready for testing!
