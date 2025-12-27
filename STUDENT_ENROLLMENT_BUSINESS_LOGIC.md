# Student Enrollment & Transfer History - Business Logic Documentation

## Overview
Sistem enrollment siswa dengan satu pendaftaran aktif per siswa dan riwayat lengkap di transfer history.

---

## Data Model

### StudentEnrollment (t_student_enrollment)
**Purpose**: Menyimpan **enrollment aktif saat ini** untuk setiap siswa

**Karakteristik**:
- ✅ Hanya 1 record per student (current active enrollment)
- ✅ Data lama akan dihapus dan dipindah ke history
- ✅ Selalu menampilkan status terkini siswa

**Fields**:
```sql
- id: BIGINT PRIMARY KEY
- student_id: BIGINT (FK to students)
- academic_year_id: INT (FK to academic_years)
- class_id: INT (FK to school_classes)
- sub_class_id: INT (FK to sub_classes) [optional]
- enrolled_at: TIMESTAMP
- status_id: INT (1=Aktif, 2=Tidak Aktif)
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

---

### StudentTransferHistory (t_student_transfer_history)
**Purpose**: Menyimpan **riwayat enrollment lama** ketika ada perubahan

**Karakteristik**:
- ✅ Menyimpan semua riwayat perpindahan/mutasi
- ✅ Permanent audit trail (tidak pernah dihapus)
- ✅ Untuk reporting, analytics, student profile

**Fields**:
```sql
- id: BIGINT PRIMARY KEY
- student_id: BIGINT
- academic_year_id: INT (FK to academic_years)
- academic_semester_id: INT (FK to semesters)
- from_class_id: INT (kelas asal)
- to_class_id: INT (kelas tujuan, nullable)
- transfer_status: ENUM (StudentMutasiStatus)
- note: VARCHAR (auto-generated)
- transferred_at: TIMESTAMP
- created_at: TIMESTAMP
- updated_at: TIMESTAMP
```

---

## Transfer Status Enum

```java
public enum StudentMutasiStatus {
    MASUK,              // Siswa baru pertama kali masuk
    NAIK_KELAS,         // Naik ke kelas lebih tinggi
    TIDAK_NAIK_KELAS,   // Tidak naik kelas (mengulang)
    DROP_OUT,           // Keluar/berhenti sekolah
    PINDAH_SEKOLAH,     // Pindah ke sekolah lain
    LAINNYA;            // Status lainnya
}
```

---

## Business Logic

### Scenario 1: Siswa Baru Pertama Kali
**Request:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 1,
  "classId": 1,
  "subClassId": 1,
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```

**Process:**
1. ✅ Check: Siswa belum ada enrollment sebelumnya
2. ✅ Validate: transferStatus == MASUK (REQUIRED)
3. ✅ Create new enrollment di t_student_enrollment
4. ✅ NO transfer history (karena pertama kali)

**Result:**
- 1 record di t_student_enrollment
- 0 record di t_student_transfer_history

---

### Scenario 2: Siswa Naik Kelas
**Request:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 2,
  "classId": 2,
  "subClassId": 1,
  "enrolledAt": "2026-07-01T08:00:00",
  "transferStatus": "NAIK_KELAS"
}
```

**Process:**
1. ✅ Check: Siswa sudah ada enrollment (Year 1, Class 1)
2. ✅ Validate: transferStatus != MASUK (REQUIRED)
3. ✅ Move old enrollment to t_student_transfer_history
4. ✅ Delete old enrollment dari t_student_enrollment
5. ✅ Create new enrollment (Year 2, Class 2)

**Result in t_student_transfer_history:**
```json
{
  "studentId": 1,
  "academicYearId": 1,
  "academicYearName": "2025/2026",
  "fromClassId": 1,
  "fromClassName": "Kelas 1",
  "toClassId": null,
  "transferStatus": "NAIK_KELAS",
  "note": "Naik kelas dari Kelas 1",
  "transferredAt": "2026-07-01T08:00:00"
}
```

**Result in t_student_enrollment:**
- Old record (Year 1, Class 1) **DELETED**
- New record (Year 2, Class 2) **CREATED**

---

### Scenario 3: Siswa Tidak Naik Kelas
**Request:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 2,
  "classId": 1,  // Same class
  "subClassId": 1,
  "enrolledAt": "2026-07-01T08:00:00",
  "transferStatus": "TIDAK_NAIK_KELAS"
}
```

**Process:**
1. ✅ Move old enrollment (Year 1, Class 1) to history
2. ✅ Delete old enrollment
3. ✅ Create new enrollment (Year 2, Class 1) - same class

**Result:**
- History note: "Tidak naik kelas, tetap di Kelas 1"
- Student tetap di kelas yang sama tapi tahun ajaran berbeda

---

### Scenario 4: Siswa Pindah Sekolah
**Request:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 2,
  "classId": 3,
  "enrolledAt": "2026-07-01T08:00:00",
  "transferStatus": "PINDAH_SEKOLAH"
}
```

**Process:**
1. ✅ Move old enrollment to history
2. ✅ History note: "Pindah sekolah dari Kelas X"
3. ✅ Create new enrollment di institusi/kelas baru

---

### Scenario 5: Siswa Drop Out
**Request:**
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 2,
  "classId": 1,
  "enrolledAt": "2026-07-01T08:00:00",
  "transferStatus": "DROP_OUT"
}
```

**Process:**
1. ✅ Move old enrollment to history
2. ✅ History note: "Drop out dari Kelas X"
3. ✅ Create new enrollment (bisa untuk re-enrollment nanti)

---

## Validation Rules

### Rule 1: MASUK hanya untuk siswa baru
```
IF student has NO existing enrollment:
    transferStatus MUST BE "MASUK"
ELSE:
    ERROR: "Siswa baru harus menggunakan status MASUK"
```

### Rule 2: Siswa existing tidak boleh MASUK
```
IF student HAS existing enrollment:
    transferStatus MUST BE one of:
        - NAIK_KELAS
        - TIDAK_NAIK_KELAS
        - DROP_OUT
        - PINDAH_SEKOLAH
        - LAINNYA
ELSE:
    ERROR: "Status MASUK hanya untuk siswa baru"
```

### Rule 3: Transfer status wajib diisi
```
IF transferStatus is NULL:
    ERROR: "Status mutasi wajib diisi (MASUK, NAIK_KELAS, TIDAK_NAIK_KELAS, DROP_OUT, PINDAH_SEKOLAH)"
```

---

## API Endpoints

### Create/Update Enrollment
```
POST /api/student-enrollments
```
**Note**: Tidak ada endpoint UPDATE terpisah. Semua perubahan dilakukan via POST dengan transferStatus yang sesuai.

### Get Current Enrollment
```
GET /api/student-enrollments
GET /api/student-enrollments/{id}
GET /api/student-enrollments/student/{studentId}
```
**Returns**: Current active enrollment only (not history)

### Get Transfer History
```
GET /api/student-enrollments/transfer-history
GET /api/student-enrollments/transfer-history/{id}
GET /api/student-enrollments/transfer-history/student/{studentId}
GET /api/student-enrollments/transfer-history/academic-year/{academicYearId}
```
**Returns**: All old enrollments that were moved to history

---

## Complete Workflow Example

### Year 1: Initial Enrollment
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 1,
  "classId": 1,
  "enrolledAt": "2025-07-01T08:00:00",
  "transferStatus": "MASUK"
}
```
**Result:**
- t_student_enrollment: 1 record (Year 1, Class 1)
- t_student_transfer_history: 0 records

---

### Year 2: Naik Kelas
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 2,
  "classId": 2,
  "enrolledAt": "2026-07-01T08:00:00",
  "transferStatus": "NAIK_KELAS"
}
```
**Result:**
- t_student_enrollment: 1 record (Year 2, Class 2) - OLD DELETED
- t_student_transfer_history: 1 record (Year 1, Class 1, status: NAIK_KELAS)

---

### Year 3: Tidak Naik Kelas
```json
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 3,
  "classId": 2,  // Same class
  "enrolledAt": "2027-07-01T08:00:00",
  "transferStatus": "TIDAK_NAIK_KELAS"
}
```
**Result:**
- t_student_enrollment: 1 record (Year 3, Class 2)
- t_student_transfer_history: 2 records
  - Record 1: Year 1, Class 1, status: NAIK_KELAS
  - Record 2: Year 2, Class 2, status: TIDAK_NAIK_KELAS

---

## Query Examples

### Get Current Enrollment
```http
GET /api/student-enrollments/student/1
```
**Returns**: Year 3, Class 2 (current active)

### Get All History
```http
GET /api/student-enrollments/transfer-history/student/1
```
**Returns**: 
- Year 1, Class 1, NAIK_KELAS
- Year 2, Class 2, TIDAK_NAIK_KELAS

### Get History by Year
```http
GET /api/student-enrollments/transfer-history/academic-year/1
```
**Returns**: All students who had enrollment in Year 1 (now in history)

---

## Benefits of This Design

### 1. Performance
✅ t_student_enrollment always has minimal records (1 per student)
✅ Queries for current data are fast
✅ No need to filter by academic year for current data

### 2. Data Integrity
✅ One source of truth for current enrollment
✅ Complete audit trail in history table
✅ No duplicate or conflicting enrollments

### 3. Reporting
✅ Easy to query current state
✅ Complete history for analytics
✅ Track student progression over years
✅ Generate promotion/retention reports

### 4. Flexibility
✅ Support multiple transfer scenarios
✅ Clear status for each transition
✅ Auto-generated notes for clarity

---

## Error Handling

### Error 1: Wrong status for new student
```json
{
  "success": false,
  "message": "Siswa baru harus menggunakan status MASUK",
  "errorCode": 4001,
  "status": 400
}
```

### Error 2: Wrong status for existing student
```json
{
  "success": false,
  "message": "Status MASUK hanya untuk siswa baru. Gunakan status: NAIK_KELAS, TIDAK_NAIK_KELAS, DROP_OUT, atau PINDAH_SEKOLAH",
  "errorCode": 4001,
  "status": 400
}
```

### Error 3: Missing transfer status
```json
{
  "success": false,
  "message": "Status mutasi wajib diisi (MASUK, NAIK_KELAS, TIDAK_NAIK_KELAS, DROP_OUT, PINDAH_SEKOLAH)",
  "errorCode": 1001,
  "status": 400
}
```

---

## Migration Notes

### For Existing Data
If you have existing enrollments with multiple records per student:

1. **Identify Current Enrollment** for each student (latest by enrolled_at or academic_year)
2. **Keep in t_student_enrollment**
3. **Move Old Enrollments** to t_student_transfer_history with status NAIK_KELAS
4. **Set Appropriate Notes** based on class progression

### Migration Script Example
```sql
-- For each student with multiple enrollments:
-- 1. Find latest enrollment
-- 2. Keep it in t_student_enrollment
-- 3. Move others to t_student_transfer_history
```

---

## Testing Checklist

- [ ] Create enrollment for new student (MASUK)
- [ ] Try MASUK for existing student (should fail)
- [ ] Create enrollment with NAIK_KELAS (should move old to history)
- [ ] Verify old enrollment deleted from t_student_enrollment
- [ ] Verify old enrollment exists in t_student_transfer_history
- [ ] Create enrollment with TIDAK_NAIK_KELAS
- [ ] Create enrollment with DROP_OUT
- [ ] Create enrollment with PINDAH_SEKOLAH
- [ ] Query current enrollment (should show latest only)
- [ ] Query transfer history (should show all old enrollments)
- [ ] Test filtering by institution/yayasan
- [ ] Test all error scenarios

---

## Summary

| Aspect | t_student_enrollment | t_student_transfer_history |
|--------|---------------------|---------------------------|
| **Purpose** | Current active enrollment | Historical enrollments |
| **Records** | 1 per student | Multiple per student |
| **Updates** | Old deleted, new created | New record added, never deleted |
| **Query Use** | "What is student's current class?" | "What classes did student attend?" |
| **Performance** | Very fast (minimal records) | Grows over time |
| **Data Lifecycle** | Volatile (replaced on change) | Permanent (audit trail) |

**Key Principle**: StudentEnrollment = **CURRENT STATE**, StudentTransferHistory = **HISTORICAL RECORD**
