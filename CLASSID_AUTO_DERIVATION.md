# Update: Automatic classId Derivation from subClassId

## Date
November 27, 2025

## Changes Summary

### Problem
Sebelumnya, user harus menyertakan `classId` dalam request untuk create dan update student enrollment, meskipun `classId` sebenarnya sudah bisa didapatkan dari relasi `SubClass -> SchoolClass`.

### Solution
Menghapus field `classId` dari request DTO dan mengimplementasikan automatic derivation dari `subClassId`.

## Files Modified

### 1. StudentEnrollmentRequest.java
**Sebelum:**
```java
@NotNull(message = "ID kelas wajib diisi")
@JsonProperty("classId")
private Integer classId;

@JsonProperty("subClassId")
private Integer subClassId;
```

**Sesudah:**
```java
@NotNull(message = "ID sub kelas wajib diisi")
@JsonProperty("subClassId")
private Integer subClassId;
```

**Changes:**
- ❌ Removed `classId` field
- ✅ Made `subClassId` required with `@NotNull`

### 2. StudentEnrollmentUpdateRequest.java
**Sebelum:**
```java
@JsonProperty("classId")
private Integer classId;

@JsonProperty("subClassId")
private Integer subClassId;
```

**Sesudah:**
```java
@JsonProperty("subClassId")
private Integer subClassId;
```

**Changes:**
- ❌ Removed `classId` field

### 3. StudentEnrollmentServiceImpl.java

#### Create Method
**Sebelum:**
```java
SchoolClass schoolClass = validateAndGetSchoolClass(request.getClassId());

if (request.getSubClassId() != null) {
    SubClass subClass = subClassRepository.findById(request.getSubClassId())
            .orElseThrow(...);
}

// ...
.classId(request.getClassId())
```

**Sesudah:**
```java
// Get SubClass and derive classId from it
SubClass subClass = subClassRepository.findById(request.getSubClassId())
        .orElseThrow(() -> new ApiException(ApiErrorCode.RESOURCE_NOT_FOUND,
                "Sub kelas dengan ID " + request.getSubClassId() + " tidak ditemukan"));

Integer classId = subClass.getSchoolClass().getId();

// Validate school class belongs to user's institution
validateAndGetSchoolClass(classId);

// ...
.classId(classId)
```

**Changes:**
- ✅ Always fetch SubClass (now required)
- ✅ Derive classId from `subClass.getSchoolClass().getId()`
- ✅ Validate derived classId belongs to user's institution
- ✅ Use derived classId when building enrollment entity

#### Update Method
**Sebelum:**
```java
if (request.getClassId() != null) {
    validateAndGetSchoolClass(request.getClassId());
    enrollment.setClassId(request.getClassId());
}

if (request.getSubClassId() != null) {
    SubClass subClass = subClassRepository.findById(request.getSubClassId())
            .orElseThrow(...);
    enrollment.setSubClassId(request.getSubClassId());
}
```

**Sesudah:**
```java
if (request.getSubClassId() != null) {
    SubClass subClass = subClassRepository.findById(request.getSubClassId())
            .orElseThrow(() -> new ApiException(ApiErrorCode.RESOURCE_NOT_FOUND,
                    "Sub kelas dengan ID " + request.getSubClassId() + " tidak ditemukan"));
    
    // Get classId from subClass
    newClassId = subClass.getSchoolClass().getId();
    
    // Validate school class belongs to user's institution
    validateAndGetSchoolClass(newClassId);
    
    // Update both classId and subClassId
    enrollment.setClassId(newClassId);
    enrollment.setSubClassId(request.getSubClassId());
    
    // Check if class changed
    classChanged = !newClassId.equals(oldClassId);
}
```

**Changes:**
- ✅ Removed separate classId update logic
- ✅ When subClassId is updated, automatically derive and update classId
- ✅ Validate derived classId
- ✅ Update both fields together
- ✅ Track class changes based on derived classId

### 4. StudentEnrollmentBulk.http
**Updated test cases to remove `classId` from requests**

## API Changes

### Create Enrollment
**Before:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 1,
  "classId": 1,          // ❌ Required
  "subClassId": 1,       // Optional
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

**After:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 1,
  "subClassId": 1,       // ✅ Required (classId auto-derived)
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

### Update Enrollment
**Before:**
```json
PUT /api/student-enrollments/{id}
{
  "classId": 2,          // Optional
  "subClassId": 2        // Optional
}
```

**After:**
```json
PUT /api/student-enrollments/{id}
{
  "subClassId": 2        // Optional (classId auto-derived and updated)
}
```

### Bulk Insert
**Before:**
```json
POST /api/student-enrollments/bulk
{
  "studentId": [1, 2, 3],
  "academicYearId": 1,
  "subClassId": 1,
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

**After:**
```json
POST /api/student-enrollments/bulk
{
  "studentId": [1, 2, 3],
  "academicYearId": 1,
  "subClassId": 1,       // classId auto-derived
  "enrolledAt": "2025-07-01T08:00:00"
}
```

## Benefits

### 1. Simplified API
- ✅ Less fields to send in request
- ✅ No redundant data
- ✅ Cleaner request payload

### 2. Data Consistency
- ✅ Impossible to send mismatched classId and subClassId
- ✅ Always consistent with SubClass -> SchoolClass relationship
- ✅ Single source of truth (SubClass relationship)

### 3. Better Validation
- ✅ SubClass validation ensures SchoolClass exists
- ✅ Automatic institution/yayasan validation through SchoolClass
- ✅ Prevents orphaned relationships

### 4. Easier Maintenance
- ✅ Changes to class structure handled automatically
- ✅ No need to update multiple fields when moving subclass
- ✅ Business logic in one place

## Migration Guide

### For Existing API Consumers

#### Before (Old Way):
```javascript
// Create enrollment
await fetch('/api/student-enrollments', {
  method: 'POST',
  body: JSON.stringify({
    studentId: 1,
    academicYearId: 1,
    classId: 1,              // ❌ Remove this
    subClassId: 1,
    enrolledAt: '2025-07-01T08:00:00',
    transferStatus: 'MASUK'
  })
});
```

#### After (New Way):
```javascript
// Create enrollment
await fetch('/api/student-enrollments', {
  method: 'POST',
  body: JSON.stringify({
    studentId: 1,
    academicYearId: 1,
    subClassId: 1,           // ✅ classId auto-derived
    enrolledAt: '2025-07-01T08:00:00',
    transferStatus: 'MASUK'
  })
});
```

### Breaking Changes
⚠️ **This is a breaking change for API consumers**

**Impact:**
- Create endpoint: Sending `classId` will be ignored (field not in DTO)
- Update endpoint: Sending `classId` will be ignored (field not in DTO)
- Bulk insert: Sending `classId` will be ignored (field not in DTO)

**Migration Steps:**
1. Update frontend/client code to remove `classId` from requests
2. Ensure `subClassId` is always provided (now required for create)
3. Update API documentation
4. Notify all API consumers of the change

## Error Scenarios

### Invalid SubClass ID
**Request:**
```json
{
  "studentId": 1,
  "academicYearId": 1,
  "subClassId": 9999,
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

**Response:**
```json
{
  "success": false,
  "message": "Sub kelas dengan ID 9999 tidak ditemukan",
  "errorCode": "RESOURCE_NOT_FOUND"
}
```

### SubClass from Different Institution
**Request:**
```json
{
  "studentId": 1,
  "academicYearId": 1,
  "subClassId": 5,  // Belongs to different institution
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

**Response:**
```json
{
  "success": false,
  "message": "Anda tidak memiliki akses ke kelas ini",
  "errorCode": "PERMISSION_DENIED"
}
```

## Testing

### Updated Test Cases
All test cases in `StudentEnrollmentBulk.http` have been updated to:
- ❌ Remove `classId` from all requests
- ✅ Keep only `subClassId`
- ✅ Verify automatic classId derivation works

### Test Coverage
- ✅ Create with valid subClassId
- ✅ Create with invalid subClassId (error)
- ✅ Update with new subClassId
- ✅ Bulk insert with valid subClassIds
- ✅ Multi-tenant validation through derived classId

## Compilation Status
✅ **BUILD SUCCESS**
- No compilation errors
- All warnings are non-critical (Lombok @Builder warnings)

## Database Schema
**No changes required** - `classId` column still exists in `t_student_enrollment` table, it's just auto-populated from SubClass relationship.

## Conclusion

✅ **Successfully implemented automatic classId derivation**
- Cleaner API with less redundant data
- Better data consistency
- Stronger validation
- Single source of truth for class relationships
- All existing functionality preserved
- Multi-tenant security maintained

**Note:** This is a breaking change for API consumers - they need to update their code to remove `classId` from requests.
