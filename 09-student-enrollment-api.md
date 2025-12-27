# Student Enrollment API Documentation

## Overview
API untuk mengelola pendaftaran siswa (Student Enrollment) pada tahun ajaran tertentu, serta riwayat mutasi/perpindahan kelas (Transfer History). Semua operasi difilter berdasarkan **yayasan** dan **institusi** user yang sedang login untuk keamanan data.

## Base URL
```
/api/student-enrollments
```

## Authentication
Semua endpoint memerlukan Bearer Token authentication.

---

## TABLE OF CONTENTS
1. [Student Enrollment Endpoints](#student-enrollment-endpoints)
2. [Student Transfer History Endpoints](#student-transfer-history-endpoints)
3. [Business Rules](#business-rules)
4. [Error Codes](#error-codes)
5. [Common Use Cases](#common-use-cases)

---

# STUDENT ENROLLMENT ENDPOINTS

### 1. List All Student Enrollments
Get semua data pendaftaran siswa dengan pagination.

**Endpoint:**
```
GET /api/student-enrollments
```

**Query Parameters:**
- `page` (optional, default: 0) - Halaman data
- `size` (optional, default: 10) - Jumlah data per halaman
- `sortBy` (optional, default: "id") - Field untuk sorting
- `sortDirection` (optional, default: "DESC") - Arah sorting (ASC/DESC)
- `draw` (optional) - Draw number untuk jQuery DataTable
- `format` (optional, default: "standard") - Format response (standard/jquery-datatable/ant-table)

**Response Formats:**

**Standard Format:**
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

**jQuery DataTable Format:**
```json
{
  "draw": 1,
  "recordsTotal": 100,
  "recordsFiltered": 100,
  "data": [...]
}
```

**Ant Design Table Format:**
```json
{
  "data": [...],
  "success": true,
  "total": 100,
  "current": 1,
  "pageSize": 10
}
```

---

### 2. Get Student Enrollment by ID
Get detail data pendaftaran siswa berdasarkan ID.

**Endpoint:**
```
GET /api/student-enrollments/{id}
```

**Path Parameters:**
- `id` (required) - ID pendaftaran siswa

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data pendaftaran siswa berhasil ditemukan",
  "data": {
    "id": 1,
    "studentId": 1,
    "studentName": "Budi Santoso",
    "academicYearId": 1,
    "academicYearName": "2025/2026",
    "classId": 1,
    "className": "Kelas 1",
    "subClassId": 1,
    "subClassName": "1A",
    "enrolledAt": "2025-07-01T08:00:00",
    "statusId": 1,
    "statusName": "Aktif",
    "createdAt": "2025-11-26T10:00:00",
    "updatedAt": "2025-11-26T10:00:00"
  },
  "errorCode": null,
  "status": 200,
  "timestamp": "2025-11-26T10:00:00Z",
  "path": "/api/student-enrollments/1"
}
```

**Error Responses:**

**404 Not Found:**
```json
{
  "success": false,
  "message": "Data pendaftaran siswa dengan ID 1 tidak ditemukan",
  "data": null,
  "errorCode": 3000,
  "status": 404,
  "timestamp": "2025-11-26T10:00:00Z",
  "path": "/api/student-enrollments/1"
}
```

**403 Forbidden:**
```json
{
  "success": false,
  "message": "Anda tidak memiliki akses ke siswa ini karena berbeda yayasan",
  "data": null,
  "errorCode": 2003,
  "status": 403
}
```

---

### 3. Get All Enrollments for Specific Student
Get semua data pendaftaran untuk siswa tertentu.

**Endpoint:**
```
GET /api/student-enrollments/student/{studentId}
```

**Path Parameters:**
- `studentId` (required) - ID siswa

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data pendaftaran siswa berhasil ditemukan. Total: 3 data",
  "data": [
    {
      "id": 1,
      "studentId": 1,
      "studentName": "Budi Santoso",
      "academicYearId": 1,
      "academicYearName": "2025/2026",
      "classId": 1,
      "className": "Kelas 1",
      "subClassId": 1,
      "subClassName": "1A",
      "enrolledAt": "2025-07-01T08:00:00",
      "statusId": 1,
      "statusName": "Aktif",
      "createdAt": "2025-11-26T10:00:00",
      "updatedAt": "2025-11-26T10:00:00"
    }
  ]
}
```

---

### 4. Get Enrollment by Student ID and Academic Year ID
Get data pendaftaran siswa untuk tahun ajaran tertentu.

**Endpoint:**
```
GET /api/student-enrollments/student/{studentId}/academic-year/{academicYearId}
```

**Path Parameters:**
- `studentId` (required) - ID siswa
- `academicYearId` (required) - ID tahun ajaran

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data pendaftaran siswa berhasil ditemukan",
  "data": {
    "id": 1,
    "studentId": 1,
    "studentName": "Budi Santoso",
    "academicYearId": 1,
    "academicYearName": "2025/2026",
    "classId": 1,
    "className": "Kelas 1",
    "subClassId": 1,
    "subClassName": "1A",
    "enrolledAt": "2025-07-01T08:00:00",
    "statusId": 1,
    "statusName": "Aktif",
    "createdAt": "2025-11-26T10:00:00",
    "updatedAt": "2025-11-26T10:00:00"
  }
}
```

---

### 5. Create New Student Enrollment
Membuat data pendaftaran siswa baru.

**Endpoint:**
```
POST /api/student-enrollments
```

**Request Body:**
```json
{
  "studentId": 1,
  "academicYearId": 1,
  "classId": 1,
  "subClassId": 1,
  "enrolledAt": "2025-07-01T08:00:00",
  "statusId": 1
}
```

**Field Validations:**
- `studentId` (required) - ID siswa wajib diisi
- `academicYearId` (required) - ID tahun ajaran wajib diisi
- `classId` (required) - ID kelas wajib diisi
- `subClassId` (optional) - ID sub kelas (opsional)
- `enrolledAt` (required) - Tanggal pendaftaran wajib diisi
- `statusId` (required, min: 1) - Status pendaftaran wajib diisi, minimal 1

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Pendaftaran siswa berhasil dibuat",
  "data": {
    "id": 1,
    "studentId": 1,
    "studentName": "Budi Santoso",
    "academicYearId": 1,
    "academicYearName": "2025/2026",
    "classId": 1,
    "className": "Kelas 1",
    "subClassId": 1,
    "subClassName": "1A",
    "enrolledAt": "2025-07-01T08:00:00",
    "statusId": 1,
    "statusName": "Aktif",
    "createdAt": "2025-11-26T10:00:00",
    "updatedAt": "2025-11-26T10:00:00"
  },
  "status": 201
}
```

**Error Responses:**

**400 Bad Request - Validation Error:**
```json
{
  "success": false,
  "message": "ID siswa wajib diisi",
  "data": null,
  "errorCode": 1001,
  "status": 400
}
```

**404 Not Found - Student Not Found:**
```json
{
  "success": false,
  "message": "Siswa dengan ID 99999 tidak ditemukan",
  "data": null,
  "errorCode": 3000,
  "status": 404
}
```

**409 Conflict - Duplicate Entry:**
```json
{
  "success": false,
  "message": "Siswa Budi Santoso sudah terdaftar pada tahun ajaran 2025/2026",
  "data": null,
  "errorCode": 1006,
  "status": 409
}
```

---

### 6. Update Student Enrollment
Update data pendaftaran siswa. **Jika kelas berubah, akan otomatis mencatat riwayat mutasi ke StudentTransferHistory.**

**Endpoint:**
```
PUT /api/student-enrollments/{id}
```

**Path Parameters:**
- `id` (required) - ID pendaftaran siswa

**Request Body (Semua field opsional untuk partial update):**
```json
{
  "studentId": 1,
  "academicYearId": 1,
  "classId": 3,
  "subClassId": 2,
  "enrolledAt": "2025-07-01T09:00:00",
  "statusId": 1
}
```

**Field Validations:**
- `studentId` (optional) - ID siswa
- `academicYearId` (optional) - ID tahun ajaran
- `classId` (optional) - ID kelas
- `subClassId` (optional) - ID sub kelas
- `enrolledAt` (optional) - Tanggal pendaftaran
- `statusId` (optional, min: 1) - Status pendaftaran minimal 1

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data pendaftaran siswa berhasil diperbarui",
  "data": {
    "id": 1,
    "studentId": 1,
    "studentName": "Budi Santoso",
    "academicYearId": 1,
    "academicYearName": "2025/2026",
    "classId": 3,
    "className": "Kelas 3",
    "subClassId": 2,
    "subClassName": "3B",
    "enrolledAt": "2025-07-01T09:00:00",
    "statusId": 1,
    "statusName": "Aktif",
    "createdAt": "2025-11-26T10:00:00",
    "updatedAt": "2025-11-26T11:00:00"
  }
}
```

**Note:** Jika `classId` berubah, sistem akan otomatis mencatat riwayat mutasi ke tabel `t_student_transfer_history` dengan informasi:
- Kelas asal (from_class_id)
- Kelas tujuan (to_class_id)
- Tahun ajaran
- Status mutasi: MUTASI_KELAS
- Tanggal mutasi
- Catatan otomatis

---

### 7. Delete Student Enrollment
Hapus data pendaftaran siswa (hard delete).

**Endpoint:**
```
DELETE /api/student-enrollments/{id}
```

**Path Parameters:**
- `id` (required) - ID pendaftaran siswa

**Success Response (204 No Content):**
```json
{
  "success": true,
  "message": "Data pendaftaran siswa berhasil dihapus",
  "data": null,
  "status": 204
}
```

---

## Business Rules

### 1. Institution & Yayasan Filtering
- Semua operasi **otomatis difilter** berdasarkan yayasan dan institusi user yang sedang login
- User tidak bisa melihat atau mengubah data dari yayasan/institusi lain
- Error `PERMISSION_DENIED (2003)` akan muncul jika mencoba akses data lain

### 2. Duplicate Prevention
- Tidak bisa membuat enrollment duplikat (siswa yang sama pada tahun ajaran yang sama)
- Error `DUPLICATE_ENTRY (1006)` akan muncul jika mencoba membuat duplikat

### 3. Transfer History Recording
- Setiap perubahan kelas akan otomatis dicatat ke `t_student_transfer_history`
- Recording dilakukan secara otomatis di background
- Jika recording gagal, update tetap berhasil (non-blocking)

### 4. Validation Rules
- `studentId`: Harus valid dan milik institusi yang sama
- `academicYearId`: Harus valid dan milik institusi yang sama
- `classId`: Harus valid dan milik institusi yang sama
- `subClassId`: Opsional, tapi jika diisi harus valid
- `statusId`: Minimal 1
- `enrolledAt`: Harus valid datetime

---

## Error Codes

| Code | Name | Description | HTTP Status |
|------|------|-------------|-------------|
| 1000 | INVALID_INPUT | Input request tidak valid | 400 |
| 1001 | MISSING_FIELD | Field wajib tidak dikirim | 400 |
| 1006 | DUPLICATE_ENTRY | Entry duplikat | 409 |
| 2003 | PERMISSION_DENIED | Tidak memiliki izin akses | 403 |
| 3000 | RESOURCE_NOT_FOUND | Resource tidak ditemukan | 404 |
| 5000 | INTERNAL_ERROR | Kesalahan server | 500 |

---

## Common Use Cases

### Case 1: Mendaftarkan Siswa Baru untuk Tahun Ajaran Baru
```http
POST /api/student-enrollments
{
  "studentId": 1,
  "academicYearId": 2,
  "classId": 2,
  "subClassId": 1,
  "enrolledAt": "2026-07-01T08:00:00",
  "statusId": 1
}
```

### Case 2: Mutasi Siswa Pindah Kelas
```http
PUT /api/student-enrollments/1
{
  "classId": 3,
  "subClassId": 2
}
```
☑️ Akan otomatis mencatat riwayat mutasi ke `t_student_transfer_history`

### Case 3: Menonaktifkan Enrollment Siswa
```http
PUT /api/student-enrollments/1
{
  "statusId": 2
}
```

### Case 4: Melihat Riwayat Pendaftaran Siswa
```http
GET /api/student-enrollments/student/1
```

---

## Testing Notes

### Test Scenarios:
1. ✅ Create enrollment dengan data lengkap
2. ✅ Create enrollment tanpa subClassId (opsional)
3. ✅ Update kelas (cek riwayat mutasi tercatat)
4. ✅ Update status only
5. ✅ Coba create duplikat (harus error 409)
6. ✅ Coba akses data yayasan lain (harus error 403)
7. ✅ Coba dengan ID tidak valid (harus error 404)
8. ✅ Coba dengan field wajib kosong (harus error 400)
9. ✅ Get by student ID
10. ✅ Get by student ID & academic year ID
11. ✅ Delete enrollment

---

## Integration with Other Modules

### Related Entities:
- `Students` - Siswa yang didaftarkan
- `AcademicYear` - Tahun ajaran
- `SchoolClass` - Kelas sekolah
- `SubClass` - Sub kelas (opsional)
- `StudentTransferHistory` - Riwayat mutasi kelas

### Automatic Operations:
- ✅ Filtering by institution & yayasan
- ✅ Recording transfer history on class change
- ✅ Populating related entity names in response
- ✅ Duplicate prevention validation

---

# STUDENT TRANSFER HISTORY ENDPOINTS

## Overview
Endpoint untuk melihat riwayat mutasi/perpindahan kelas siswa. Transfer history **otomatis tercatat** ketika enrollment di-update dengan classId yang berbeda.

---

### 8. List All Transfer History
Get semua riwayat mutasi dengan pagination.

**Endpoint:**
```
GET /api/student-enrollments/transfer-history
```

**Query Parameters:**
- `page` (optional, default: 0) - Halaman data
- `size` (optional, default: 10) - Jumlah data per halaman
- `sortBy` (optional, default: "transferredAt") - Field untuk sorting
- `sortDirection` (optional, default: "DESC") - Arah sorting (ASC/DESC)
- `draw` (optional) - Draw number untuk jQuery DataTable
- `format` (optional, default: "standard") - Format response (standard/jquery-datatable/ant-table)

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 1,
      "studentId": 1,
      "studentName": "Budi Santoso",
      "academicYearId": 1,
      "academicYearName": "2025/2026",
      "academicSemesterId": 1,
      "academicSemesterName": "Ganjil",
      "fromClassId": 1,
      "fromClassName": "Kelas 1",
      "toClassId": 2,
      "toClassName": "Kelas 2",
      "transferStatus": "MUTASI_KELAS",
      "note": "Mutasi dari kelas Kelas 1 ke Kelas 2",
      "transferredAt": "2025-11-27T10:00:00",
      "createdAt": "2025-11-27T10:00:00",
      "updatedAt": "2025-11-27T10:00:00"
    }
  ],
  "total": 100,
  "page": 0,
  "size": 10,
  "totalPages": 10,
  "hasNext": true,
  "hasPrevious": false
}
```

---

### 9. Get Transfer History by ID
Get detail riwayat mutasi berdasarkan ID.

**Endpoint:**
```
GET /api/student-enrollments/transfer-history/{id}
```

**Path Parameters:**
- `id` (required) - ID riwayat mutasi

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data riwayat mutasi berhasil ditemukan",
  "data": {
    "id": 1,
    "studentId": 1,
    "studentName": "Budi Santoso",
    "academicYearId": 1,
    "academicYearName": "2025/2026",
    "academicSemesterId": 1,
    "academicSemesterName": "Ganjil",
    "fromClassId": 1,
    "fromClassName": "Kelas 1",
    "toClassId": 2,
    "toClassName": "Kelas 2",
    "transferStatus": "MUTASI_KELAS",
    "note": "Mutasi dari kelas Kelas 1 ke Kelas 2",
    "transferredAt": "2025-11-27T10:00:00",
    "createdAt": "2025-11-27T10:00:00",
    "updatedAt": "2025-11-27T10:00:00"
  }
}
```

---

### 10. Get Transfer History by Student ID
Get semua riwayat mutasi untuk siswa tertentu.

**Endpoint:**
```
GET /api/student-enrollments/transfer-history/student/{studentId}
```

**Path Parameters:**
- `studentId` (required) - ID siswa

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data riwayat mutasi siswa berhasil ditemukan. Total: 5 data",
  "data": [
    {
      "id": 1,
      "studentId": 1,
      "studentName": "Budi Santoso",
      "academicYearId": 1,
      "academicYearName": "2025/2026",
      "fromClassId": 1,
      "fromClassName": "Kelas 1",
      "toClassId": 2,
      "toClassName": "Kelas 2",
      "transferStatus": "MUTASI_KELAS",
      "transferredAt": "2025-11-27T10:00:00"
    }
  ]
}
```

---

### 11. Get Transfer History by Academic Year
Get semua riwayat mutasi untuk tahun ajaran tertentu.

**Endpoint:**
```
GET /api/student-enrollments/transfer-history/academic-year/{academicYearId}
```

**Path Parameters:**
- `academicYearId` (required) - ID tahun ajaran

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data riwayat mutasi untuk tahun ajaran berhasil ditemukan. Total: 25 data",
  "data": [...]
}
```

---

### 12. Get Transfer History by Student ID (Paginated)
Get riwayat mutasi siswa dengan pagination.

**Endpoint:**
```
GET /api/student-enrollments/transfer-history/student/{studentId}/paginated
```

**Path Parameters:**
- `studentId` (required) - ID siswa

**Query Parameters:**
- `page` (optional, default: 0) - Halaman data
- `size` (optional, default: 10) - Jumlah data per halaman
- `sortBy` (optional, default: "transferredAt") - Field untuk sorting
- `sortDirection` (optional, default: "DESC") - Arah sorting (ASC/DESC)
- `draw` (optional) - Draw number untuk jQuery DataTable
- `format` (optional, default: "standard") - Format response

**Success Response (200 OK):**
```json
{
  "data": [...],
  "total": 5,
  "page": 0,
  "size": 10,
  "totalPages": 1,
  "hasNext": false,
  "hasPrevious": false
}
```

---

## Transfer History Features

### Automatic Recording
- ✅ **Auto-created** ketika enrollment di-update dengan classId berbeda
- ✅ Recording dilakukan di background (non-blocking)
- ✅ Update enrollment tetap berhasil meskipun recording gagal
- ✅ Mencatat: fromClassId, toClassId, academicYear, semester, note

### Data Fields
| Field | Type | Description |
|-------|------|-------------|
| id | Long | Primary key |
| studentId | Long | ID siswa yang dimutasi |
| studentName | String | Nama siswa (enriched) |
| academicYearId | Integer | ID tahun ajaran saat mutasi |
| academicYearName | String | Nama tahun ajaran (enriched) |
| academicSemesterId | Integer | ID semester saat mutasi |
| academicSemesterName | String | Nama semester (enriched) |
| fromClassId | Integer | ID kelas asal |
| fromClassName | String | Nama kelas asal (enriched) |
| toClassId | Integer | ID kelas tujuan |
| toClassName | String | Nama kelas tujuan (enriched) |
| transferStatus | String | Status mutasi (MUTASI_KELAS) |
| note | String | Catatan otomatis |
| transferredAt | LocalDateTime | Tanggal mutasi |
| createdAt | LocalDateTime | Tanggal record dibuat |
| updatedAt | LocalDateTime | Tanggal record diupdate |

### Transfer Status Values
- `MUTASI_KELAS` - Perpindahan kelas

### Security & Filtering
- ✅ All queries filtered by yayasan & institution
- ✅ Filtering melalui Student entity relationship
- ✅ User hanya bisa lihat mutasi siswa di institusinya
- ✅ Access validation pada setiap operasi

---

## Updated Common Use Cases

### Case 5: Melihat Riwayat Mutasi Siswa
```http
GET /api/student-enrollments/transfer-history/student/1
```

### Case 6: Melihat Semua Mutasi dalam Tahun Ajaran
```http
GET /api/student-enrollments/transfer-history/academic-year/1
```

### Case 7: Generate Report Mutasi dengan Pagination
```http
GET /api/student-enrollments/transfer-history?page=0&size=50&sortBy=transferredAt&sortDirection=DESC
```

### Case 8: Export Mutasi untuk Frontend (Ant Design)
```http
GET /api/student-enrollments/transfer-history?page=0&size=100
Header: format: ant-table
```

---

## Updated Testing Notes

### Additional Test Scenarios:
12. ✅ Update kelas kemudian cek transfer history tercatat
13. ✅ Get transfer history by student ID
14. ✅ Get transfer history by academic year
15. ✅ Verify transfer history filtered by institution
16. ✅ Test pagination on transfer history
17. ✅ Test multiple format responses (standard, jQuery, Ant Design)

---

## API Summary

### Student Enrollment Endpoints (7)
1. `GET /api/student-enrollments` - List dengan pagination
2. `GET /api/student-enrollments/{id}` - Get by ID
3. `GET /api/student-enrollments/student/{studentId}` - Get by student
4. `GET /api/student-enrollments/student/{studentId}/academic-year/{academicYearId}` - Get by student & year
5. `POST /api/student-enrollments` - Create
6. `PUT /api/student-enrollments/{id}` - Update (auto-records transfer history)
7. `DELETE /api/student-enrollments/{id}` - Delete

### Transfer History Endpoints (5)
8. `GET /api/student-enrollments/transfer-history` - List dengan pagination
9. `GET /api/student-enrollments/transfer-history/{id}` - Get by ID
10. `GET /api/student-enrollments/transfer-history/student/{studentId}` - Get by student
11. `GET /api/student-enrollments/transfer-history/academic-year/{academicYearId}` - Get by academic year
12. `GET /api/student-enrollments/transfer-history/student/{studentId}/paginated` - Get by student (paginated)

**Total: 12 Endpoints**

---
- `AcademicYear` - Tahun ajaran
- `SchoolClass` - Kelas sekolah
- `SubClass` - Sub kelas (opsional)
- `StudentTransferHistory` - Riwayat mutasi kelas

### Automatic Operations:
- ✅ Filtering by institution & yayasan
- ✅ Recording transfer history on class change
- ✅ Populating related entity names in response
- ✅ Duplicate prevention validation
