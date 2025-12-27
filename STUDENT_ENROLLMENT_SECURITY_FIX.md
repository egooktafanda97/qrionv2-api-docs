# Student Enrollment Security Fix

## 🔒 Critical Security Issue - FIXED

### Problem
Sebelumnya, API bulk insert Student Enrollment **tidak memvalidasi kepemilikan resource** (SubClass, AcademicYear, Students) terhadap Yayasan dan Institusi yang sedang login. Ini berarti:

❌ User dari Institusi A bisa mendaftarkan siswa ke SubClass milik Institusi B  
❌ User bisa mengakses Academic Year dari institusi lain  
❌ Tidak ada cross-institution boundary enforcement

### Solution Implemented

#### 1. **New Validation Method: `validateAndGetSubClass()`**
```java
/**
 * Validate sub class belongs to current user's institution (through SchoolClass)
 */
private SubClass validateAndGetSubClass(Integer subClassId) {
    Users currentUser = getCurrentUser();
    Integer institutionId = currentUser.getInstitution() != null 
        ? currentUser.getInstitution().getId() : null;

    SubClass subClass = subClassRepository.findById(subClassId)
        .orElseThrow(() -> new ApiException(ApiErrorCode.RESOURCE_NOT_FOUND,
            "Sub kelas dengan ID " + subClassId + " tidak ditemukan"));

    // Validate through SchoolClass relationship
    SchoolClass schoolClass = subClass.getSchoolClass();
    if (schoolClass == null) {
        throw new ApiException(ApiErrorCode.DATA_INTEGRITY_VIOLATION,
            "Sub kelas tidak memiliki kelas induk yang valid");
    }

    if (institutionId != null && !institutionId.equals(schoolClass.getInstitution().getId())) {
        throw new ApiException(ApiErrorCode.PERMISSION_DENIED,
            "Anda tidak memiliki akses ke sub kelas ini karena berbeda institusi");
    }

    return subClass;
}
```

#### 2. **Security Validations Applied**

**Before** (Vulnerable):
```java
@Override
@Transactional
public List<StudentEnrollmentResponse> createBulk(StudentEnrollmentBulkRequest request) {
    // ❌ No validation - langsung ambil dari DB
    SubClass subClass = subClassRepository.findById(request.getSubClassId())
        .orElseThrow(...);
    
    // ❌ User bisa input subClassId dari institusi lain!
}
```

**After** (Secured):
```java
@Override
@Transactional
public List<StudentEnrollmentResponse> createBulk(StudentEnrollmentBulkRequest request) {
    // ✅ Validate SubClass belongs to user's institution BEFORE processing
    SubClass subClass = validateAndGetSubClass(request.getSubClassId());
    
    // ✅ Validate AcademicYear belongs to user's institution
    AcademicYear academicYear = validateAndGetAcademicYear(request.getAcademicYearId());
    
    for (Long studentId : request.getStudentIds()) {
        // ✅ Validate each Student belongs to user's yayasan/institution
        Students student = validateAndGetStudent(studentId);
        
        // ... create enrollment
    }
}
```

#### 3. **Methods Updated with Security**

| Method | Validation Added |
|--------|------------------|
| `create()` | ✅ validateAndGetSubClass() |
| `createBulk()` | ✅ validateAndGetSubClass() + validateAndGetAcademicYear() |
| `update()` | ✅ validateAndGetSubClass() |

### Security Layers

```
┌─────────────────────────────────────────────┐
│ 1. JWT Authentication (JwtAuthenticationFilter) │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 2. Institution/Yayasan Extraction from JWT  │
│    - currentUser.getYayasan().getId()       │
│    - currentUser.getInstitution().getId()   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 3. Resource Ownership Validation            │
│    ✅ validateAndGetStudent()               │
│    ✅ validateAndGetAcademicYear()          │
│    ✅ validateAndGetSubClass()              │
│    ✅ validateAndGetSchoolClass()           │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ 4. Business Logic Execution                 │
│    - Create enrollment                      │
│    - Update enrollment                      │
│    - Delete enrollment                      │
└─────────────────────────────────────────────┘
```

### Error Messages

**Before**:
```json
{
  "code": "RESOURCE_NOT_FOUND",
  "message": "Sub kelas dengan ID 999 tidak ditemukan"
}
```

**After** (Cross-institution attempt):
```json
{
  "code": "PERMISSION_DENIED",
  "message": "Anda tidak memiliki akses ke sub kelas ini karena berbeda institusi"
}
```

### Test Scenarios

#### ✅ Valid Request (Same Institution)
```http
POST /api/student-enrollments/bulk
Authorization: Bearer <JWT_INSTITUTION_1>
Content-Type: application/json

{
  "studentId": [1, 2, 3],     // Students from Institution 1
  "academicYearId": 1,         // Academic Year from Institution 1
  "subClassId": 1              // SubClass from Institution 1
}

Response: 200 OK ✅
```

#### ❌ Invalid Request (Cross-institution)
```http
POST /api/student-enrollments/bulk
Authorization: Bearer <JWT_INSTITUTION_1>
Content-Type: application/json

{
  "studentId": [1, 2, 3],     // Students from Institution 1
  "academicYearId": 99,        // ❌ Academic Year from Institution 2
  "subClassId": 1              // SubClass from Institution 1
}

Response: 403 PERMISSION_DENIED
{
  "code": "PERMISSION_DENIED",
  "message": "Anda tidak memiliki akses ke tahun ajaran ini"
}
```

```http
POST /api/student-enrollments/bulk
Authorization: Bearer <JWT_INSTITUTION_1>
Content-Type: application/json

{
  "studentId": [1, 2, 3],     // Students from Institution 1
  "academicYearId": 1,         // Academic Year from Institution 1
  "subClassId": 99             // ❌ SubClass from Institution 2
}

Response: 403 PERMISSION_DENIED
{
  "code": "PERMISSION_DENIED",
  "message": "Anda tidak memiliki akses ke sub kelas ini karena berbeda institusi"
}
```

### Impact
✅ **Security Risk Eliminated**: Cross-institution data manipulation is now blocked  
✅ **Multi-tenant Integrity**: Each institution can only access their own resources  
✅ **Consistent Validation**: Same security rules applied across create, update, bulk operations  
✅ **Clear Error Messages**: Users understand why access is denied

### Related Files Modified
- `StudentEnrollmentServiceImpl.java`
  - Added: `validateAndGetSubClass()` method
  - Modified: `create()`, `createBulk()`, `update()` methods

### Date Fixed
November 27, 2025

### Reported By
User (ego.oktafanda)

### Severity
**CRITICAL** - Allows unauthorized cross-institution data manipulation
