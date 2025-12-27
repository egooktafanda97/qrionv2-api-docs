# Cascading Inactive untuk Academic Year dan Semester

## Overview
Fitur ini mengimplementasikan logika cascading inactive pada tabel Academic Year dan Semester untuk menjaga konsistensi data dan memastikan hanya ada satu Academic Year ACTIVE di setiap yayasan dan institusi.

## Business Rules

### Saat Create Academic Year Baru dengan Status ACTIVE

1. **Validasi Multi-tenant**
   - Semua operasi harus berdasarkan yayasan dan institusi dari user yang sedang login
   - Tidak boleh mempengaruhi data yayasan dan institusi lain

2. **Cascading Inactive Process**
   - Jika academic year baru dibuat dengan `status = "ACTIVE"`
   - Sistem akan mencari semua academic year yang statusnya `"ACTIVE"` di yayasan dan institusi yang sama
   - Untuk setiap academic year lama yang ACTIVE:
     - Cari semua semester dengan `status = "ACTIVE"` yang terkait dengan academic year tersebut
     - Set semua semester tersebut menjadi `status = "INACTIVE"`
     - Set academic year lama menjadi `status = "INACTIVE"`
   - Setelah itu, baru create academic year baru

3. **Contoh Skenario**
   ```
   State Awal:
   - Academic Year: 2024/2025 (ACTIVE)
     - Semester: GANJIL (ACTIVE)
     - Semester: GENAP (ACTIVE)
   
   Action:
   - Create Academic Year: 2026/2027 (ACTIVE)
   
   State Akhir:
   - Academic Year: 2024/2025 (INACTIVE) ← Automatically set
     - Semester: GANJIL (INACTIVE) ← Automatically set
     - Semester: GENAP (INACTIVE) ← Automatically set
   - Academic Year: 2026/2027 (ACTIVE) ← Newly created
   ```

## Implementation Details

### Modified Files

1. **SemesterRepository.java**
   - Added method: `findByAcademicYearAndStatusAndYayasanAndInstitutionAndDeletedAtIsNull()`
   - Purpose: Find active semesters for a specific academic year

2. **AcademicYearServiceImpl.java**
   - Added dependency: `SemesterRepository`
   - Modified method: `create(AcademicYearRequest request)`
   - Added logic:
     ```java
     if ("ACTIVE".equalsIgnoreCase(request.getStatus())) {
         // Find all active academic years
         // For each active academic year:
         //   - Find all active semesters
         //   - Set semesters to INACTIVE
         //   - Set academic year to INACTIVE
     }
     ```

### Database Impact

**Tables Affected:**
- `m_academic_year` - Status column will be updated
- `m_semester` - Status column will be updated

**Queries Executed:**
1. Find active academic years by yayasan and institution
2. Find active semesters for each academic year
3. Update semester status to INACTIVE
4. Update academic year status to INACTIVE
5. Insert new academic year with ACTIVE status

## Testing

### Test Scenario 1: Basic Cascading Inactive

**Pre-conditions:**
```http
# Create first academic year
POST /api/academic-years
{
  "name": "2024/2025",
  "startDate": "2024-07-01",
  "endDate": "2025-06-30",
  "status": "ACTIVE"
}

# Create semesters for 2024/2025
POST /api/semesters
{
  "academicYearId": "2024/2025",
  "name": "GANJIL",
  "startDate": "2024-07-01",
  "endDate": "2024-12-31",
  "status": "ACTIVE"
}
```

**Test Action:**
```http
# Create new academic year (should trigger cascading)
POST /api/academic-years
{
  "name": "2026/2027",
  "startDate": "2026-07-01",
  "endDate": "2027-06-30",
  "status": "ACTIVE"
}
```

**Expected Result:**
- Academic Year 2024/2025: status changed to INACTIVE
- Semester GANJIL (from 2024/2025): status changed to INACTIVE
- Academic Year 2026/2027: created with status ACTIVE

### Test Scenario 2: Multi-tenant Isolation

**Pre-conditions:**
- Yayasan A, Institution 1: Has Academic Year 2024/2025 (ACTIVE)
- Yayasan B, Institution 2: Has Academic Year 2024/2025 (ACTIVE)

**Test Action:**
- User from Yayasan A creates Academic Year 2026/2027 (ACTIVE)

**Expected Result:**
- Yayasan A: Academic Year 2024/2025 becomes INACTIVE
- Yayasan B: Academic Year 2024/2025 remains ACTIVE ← No impact
- Multi-tenant isolation is maintained

### Test Scenario 3: Create with INACTIVE Status

**Test Action:**
```http
POST /api/academic-years
{
  "name": "2027/2028",
  "startDate": "2027-07-01",
  "endDate": "2028-06-30",
  "status": "INACTIVE"
}
```

**Expected Result:**
- New academic year created with INACTIVE status
- No cascading happens (existing active academic years remain unchanged)

## Security Considerations

✅ **Multi-tenant Security**
- All queries filter by yayasan_id and institution_id
- User cannot affect data from other yayasan/institution
- Auth.user() is used to get authenticated user's context

✅ **Data Consistency**
- Transactional integrity (@Transactional on service)
- All updates happen in single transaction
- Rollback on any error

## API Endpoints

### Create Academic Year
```
POST /api/academic-years
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "2026/2027",
  "startDate": "2026-07-01",
  "endDate": "2027-06-30",
  "status": "ACTIVE"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Academic year created successfully",
  "data": {
    "id": 5,
    "yayasanId": 1,
    "institutionId": 1,
    "name": "2026/2027",
    "startDate": "2026-07-01",
    "endDate": "2027-06-30",
    "status": "ACTIVE",
    "createdAt": "2025-11-27T16:00:00",
    "updatedAt": null
  }
}
```

## Limitations

1. **Only affects ACTIVE status**
   - Cascading only triggers when new academic year has `status = "ACTIVE"`
   - Creating academic year with `status = "INACTIVE"` does not trigger cascading

2. **Single Active Academic Year per Tenant**
   - System assumes only one active academic year should exist per yayasan/institution
   - If business logic changes, this constraint needs review

3. **No validation for overlapping dates**
   - System does not check if academic year dates overlap
   - This is a separate validation concern

## Future Enhancements

- [ ] Add option to prevent cascading via request parameter
- [ ] Add audit log for cascading operations
- [ ] Add notification when academic year becomes inactive
- [ ] Add validation for overlapping academic year dates
- [ ] Add rollback mechanism if cascading fails partially

## Related Documentation

- [Academic Year API](documentations/01-academic-year-api.md)
- [Semester API](documentations/02-semester-api.md)
- [Authentication Guide](AUTH_USAGE_GUIDE.md)
