# Qrion API Documentation Overview

## Table of Contents
1. [Introduction](#introduction)
2. [API Response Format](#api-response-format)
3. [Error Codes](#error-codes)
4. [HTTP Status Codes](#http-status-codes)
5. [Authentication](#authentication)
6. [Base URL](#base-url)
7. [Available Modules](#available-modules)

---

## Introduction

Qrion adalah sistem manajemen sekolah yang menyediakan API untuk mengelola berbagai aspek operasional sekolah. API ini dibangun menggunakan Spring Boot 3.x dengan Jakarta EE dan menggunakan pattern yang konsisten untuk semua endpoint.

---

## API Response Format

Semua respons API mengikuti format standar berikut:

### Success Response (Status 200, 201)
```json
{
  "success": true,
  "message": "Deskripsi pesan sukses",
  "data": {
    // Konten data yang diminta
  },
  "status": 200,
  "timestamp": "2025-11-22T06:45:31Z",
  "path": "/api/v1/endpoint"
}
```

### Error Response (Status 400, 404, 409, 500, dll)
```json
{
  "success": false,
  "message": "Deskripsi error",
  "errorCode": 3000,
  "status": 404,
  "timestamp": "2025-11-22T06:45:31Z",
  "path": "/api/v1/endpoint"
}
```

### Validation Error Response (Status 400)
```json
{
  "success": false,
  "message": "Validation failed",
  "errorCode": 1000,
  "status": 400,
  "timestamp": "2025-11-22T06:45:31Z",
  "path": "/api/v1/endpoint",
  "errors": {
    "fieldName1": "Error message for field 1",
    "fieldName2": "Error message for field 2"
  }
}
```

### Response Fields Explanation

| Field | Type | Description |
|-------|------|-------------|
| success | Boolean | Indicator apakah request berhasil atau gagal |
| message | String | Pesan detail tentang hasil request |
| data | Object/Array | Data yang dikembalikan (null untuk error atau delete) |
| errorCode | Integer | Kode error untuk klasifikasi error (lihat tabel Error Codes) |
| status | Integer | HTTP status code |
| timestamp | String | Waktu request dalam format ISO 8601 UTC |
| path | String | Endpoint path yang diakses |
| errors | Object | Detail field yang gagal validasi (hanya untuk validation error) |

---

## Error Codes

Error codes dalam Qrion dikelompokkan berdasarkan kategori:

### 1000-1999: Validation & Input Errors
| Code | Name | HTTP Status | Description |
|------|------|-------------|-------------|
| 1000 | INVALID_INPUT | 400 | Input request tidak sesuai dengan format yang diharapkan |
| 1001 | MISSING_FIELD | 400 | Field yang diperlukan tidak disertakan dalam request |
| 1002 | INVALID_FORMAT | 400 | Format data tidak valid (misal: format email, tanggal) |
| 1003 | INVALID_LENGTH | 400 | Panjang string melebihi atau kurang dari yang diharapkan |
| 1004 | CONSTRAINT_VIOLATION | 400 | Melanggar constraint database (misal: unique, not null) |
| 1005 | INVALID_VALUE | 400 | Nilai field tidak valid untuk konteks tertentu |
| 1006 | DUPLICATE_ENTRY | 409 | Data sudah ada (misal: NISN, NIP, kode kelas sudah terdaftar) |

### 2000-2999: Authentication & Authorization Errors
| Code | Name | HTTP Status | Description |
|------|------|-------------|-------------|
| 2000 | UNAUTHORIZED | 401 | Request tidak disertai token authentication |
| 2001 | INVALID_TOKEN | 401 | Token authentication tidak valid atau sudah expired |
| 2002 | FORBIDDEN | 403 | User tidak memiliki akses ke resource ini |
| 2003 | INSUFFICIENT_PERMISSION | 403 | User tidak memiliki permission untuk operasi ini |

### 3000-3999: Resource Not Found Errors
| Code | Name | HTTP Status | Description |
|------|------|-------------|-------------|
| 3000 | RESOURCE_NOT_FOUND | 404 | Resource yang dicari tidak ditemukan |
| 3001 | YAYASAN_NOT_FOUND | 404 | Yayasan dengan ID tertentu tidak ditemukan |
| 3002 | INSTITUTION_NOT_FOUND | 404 | Institusi dengan ID tertentu tidak ditemukan |
| 3003 | ACADEMIC_YEAR_NOT_FOUND | 404 | Tahun akademik dengan ID tertentu tidak ditemukan |
| 3004 | SEMESTER_NOT_FOUND | 404 | Semester dengan ID tertentu tidak ditemukan |
| 3005 | SCHOOL_CLASS_NOT_FOUND | 404 | Kelas dengan ID tertentu tidak ditemukan |
| 3006 | SUB_CLASS_NOT_FOUND | 404 | Sub kelas dengan ID tertentu tidak ditemukan |
| 3007 | STUDENT_NOT_FOUND | 404 | Siswa dengan ID tertentu tidak ditemukan |
| 3008 | TEACHER_NOT_FOUND | 404 | Guru dengan ID tertentu tidak ditemukan |

### 4000-4999: Business Logic Errors
| Code | Name | HTTP Status | Description |
|------|------|-------------|-------------|
| 4000 | BUSINESS_RULE_VIOLATION | 400 | Melanggar aturan bisnis aplikasi |
| 4001 | INVALID_SCOPE | 400 | Data tidak sesuai dengan scope yayasan/institusi user |
| 4002 | OPERATION_NOT_ALLOWED | 400 | Operasi tidak diijinkan untuk status/kondisi tertentu |
| 4003 | DUPLICATE_CODE | 409 | Kode sudah digunakan (misal: kode kelas, kode sub kelas) |

### 5000-5999: Server Errors
| Code | Name | HTTP Status | Description |
|------|------|-------------|-------------|
| 5000 | INTERNAL_ERROR | 500 | Kesalahan server internal yang tidak terduga |
| 5001 | DATABASE_ERROR | 500 | Error pada database connection atau query |
| 5002 | SERVICE_UNAVAILABLE | 503 | Service sedang tidak tersedia |

---

## HTTP Status Codes

| Status | Description | Use Case |
|--------|-------------|----------|
| 200 | OK | Request berhasil, biasanya untuk GET, PUT, DELETE |
| 201 | Created | Resource berhasil dibuat dengan POST |
| 204 | No Content | Request berhasil tapi tidak ada content yang dikembalikan |
| 400 | Bad Request | Request tidak valid, ada validation error atau business rule violation |
| 401 | Unauthorized | Authentication gagal atau token tidak ada |
| 403 | Forbidden | User tidak memiliki akses ke resource |
| 404 | Not Found | Resource tidak ditemukan |
| 409 | Conflict | Duplicate data atau conflict dengan data existing |
| 422 | Unprocessable Entity | Request format benar tapi data tidak bisa diproses |
| 500 | Internal Server Error | Kesalahan server yang tidak terduga |
| 503 | Service Unavailable | Service sedang tidak tersedia |

---

## Authentication

Semua endpoint (kecuali login dan register) memerlukan authentication menggunakan JWT Bearer Token.

### Request Header
```
Authorization: Bearer <token>
Content-Type: application/json
```

### Contoh
```bash
curl -X GET http://localhost:8080/api/v1/academic-years \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

### Token Format
Token menggunakan JWT (JSON Web Token) format:
```
Header.Payload.Signature
```

Token biasanya diperoleh dari endpoint login:
```
POST /api/v1/auth/login
```

---

## Base URL

```
http://localhost:8080/api/v1
```

Atau di production:
```
https://your-domain.com/api/v1
```

---

## Available Modules

### ✅ Fully Documented Modules (10)

| # | Module | Endpoints | Documentation File | HTTP Test File | Status |
|---|--------|-----------|-------------------|---------------|--------|
| 00 | Authentication | 8 | [00-authentication-api.md](00-authentication-api.md) | [Auth.http](../http/Auth.http) | ✅ Complete |
| 01 | Academic Year | 6 | [01-academic-year-api.md](01-academic-year-api.md) | [AcademicYearController.http](../http/AcademicYearController.http) | ✅ Complete |
| 02 | Semester | 5 | [02-semester-api.md](02-semester-api.md) | [SemesterController.http](../http/SemesterController.http) | ✅ Complete |
| 03 | School Class | 5 | [03-school-class-api.md](03-school-class-api.md) | [SchoolClassController.http](../http/SchoolClassController.http) | ✅ Complete |
| 04 | Sub Class | 5 | [04-sub-class-api.md](04-sub-class-api.md) | [SubClassController.http](../http/SubClassController.http) | ✅ Complete |
| 05 | Student | 8 | [05-student-api.md](05-student-api.md) | [StudentController.http](../http/StudentController.http) | ✅ Updated |
| 06 | Teacher | 6 | [06-teacher-api.md](06-teacher-api.md) | [TeacherController.http](../http/TeacherController.http) | ✅ Complete |
| 07 | Parent | 7 | [07-parent-api.md](07-parent-api.md) | [ParentController.http](../http/ParentController.http) | ✅ Complete |
| 08 | Scholarship | 5 | [08-scholarship-api.md](08-scholarship-api.md) | [ScholarshipController.http](../http/ScholarshipController.http) | ✅ Complete |
| 09 | Student Enrollment | 6 | [09-student-enrollment-api.md](09-student-enrollment-api.md) | [StudentEnrollmentController.http](../http/StudentEnrollmentController.http) | ✅ Complete |
| 10 | Billing | 10 | [10-billing-api.md](10-billing-api.md) | [BillingController.http](../http/BillingController.http) | ✅ NEW |

**Total Documented:** 71 endpoints across 11 modules

### 🔴 Critical Modules (Documentation Needed)

| # | Module | Priority | Controller | HTTP File | Est. Endpoints |
|---|--------|----------|-----------|-----------|----------------|
| 11 | User Billing | 🔴 Critical | UserBillingController.java | ✅ Exists | 12 |
| 12 | Transaction Biller | 🔴 Critical | TransactionBillerController.java | ✅ Updated | 10 |
| 13 | Transaction Journal | 🔴 Critical | TransactionJournalController.java | ✅ Updated | 12 |
| 14 | Account | 🔴 Critical | AccountController.java | ✅ Exists | 10 |
| 15 | M-Billings | 🔴 High | MBillingsController.java | ✅ Exists | 8 |
| 16 | Billing Scholarship | 🔴 High | BillingScholarshipController.java | ✅ Exists | 10 |
| 17 | User Scholarship | 🔴 High | UserScholarshipController.java | ✅ Exists | 8 |
| 18 | Broadcast | 🔴 High | BroadcastController.java | ✅ Exists | 12 |
| 19 | Broadcast Recipient | 🔴 High | BroadcastRecipientController.java | ✅ Exists | 15 |

### 🟡 Supporting Modules (Lower Priority)

**Configuration & Setup:**
- Institution Management
- Role & Permission Management
- Payment Method Configuration
- Tax Configuration
- Subscription Plans

**Transaction Management:**
- Invoice Management
- Payment Gateway Logs
- Transaction Addons
- Transaction Payments
- Withdrawal Management

**Query & Reporting:**
- Student Query (Advanced Search)
- User Billing Query
- Broadcast Recipient Query
- Reports & Analytics

**Utilities:**
- Seeder (Database Seeding)
- Debug Tools (iPaymu)
- Template Fields
- Show Aliases

---

## Documentation Status

### Coverage Statistics
- **Fully Documented:** 11/50 controllers (22%)
- **HTTP Test Files:** 55 files
- **Total Endpoints Documented:** 71 endpoints
- **Target Coverage:** 80% (40/50 controllers)

### Recent Updates (Dec 22, 2025)
- ✅ Student API - Added 4 new endpoints (search, compact, bulk operations)
- ✅ Billing API - Complete documentation created
- ✅ Transaction Biller HTTP - Updated with 4 endpoints
- ✅ Transaction Journal HTTP - Fixed and updated
- ✅ Audit Report - Comprehensive 50-controller audit completed

### Next Priority
1. User Billing API Documentation
2. Transaction Biller API Documentation
3. Transaction Journal API Documentation
4. Account API Documentation

---

Qrion menyediakan berbagai modul API yang terintegrasi untuk mengelola operasional sekolah. Berikut adalah daftar lengkap modul yang tersedia:

### 1. Authentication & Account Module
Mengelola autentikasi, registrasi, dan manajemen akun pengguna.

**Endpoints:**
- `POST /api/v1/auth/login` - Login dengan email/username dan password
- `POST /api/v1/auth/logout` - Logout dan invalidasi token
- `POST /api/v1/auth/refresh` - Refresh JWT token
- `POST /api/v1/register` - Registrasi institusi baru
- `GET /api/v1/account` - Get user profile
- `GET /api/v1/account/admin` - Get admin account info
- `GET /api/v1/account/check` - Check account status

**File Dokumentasi:** `00-authentication-api.md`

---

### 2. Academic Year Module
Mengelola tahun akademik di institusi.

**Endpoints:**
- `GET /api/v1/academic-years` - Get all academic years (dengan pagination)
- `GET /api/v1/academic-years/{id}` - Get academic year by ID
- `POST /api/v1/academic-years` - Create new academic year
- `PUT /api/v1/academic-years/{id}` - Update academic year
- `DELETE /api/v1/academic-years/{id}` - Delete academic year

**File Dokumentasi:** `01-academic-year-api.md`

---

### 3. Semester Module
Mengelola semester dalam tahun akademik.

**Endpoints:**
- `GET /api/v1/semesters` - Get all semesters (dengan pagination)
- `GET /api/v1/semesters/{id}` - Get semester by ID
- `POST /api/v1/semesters` - Create new semester
- `PUT /api/v1/semesters/{id}` - Update semester
- `DELETE /api/v1/semesters/{id}` - Delete semester

**File Dokumentasi:** `02-semester-api.md`

---

### 4. School Class Module
Mengelola tingkat kelas (misal: Kelas 1, Kelas 2, dst).

**Endpoints:**
- `GET /api/v1/classes` - Get all school classes (dengan pagination)
- `GET /api/v1/classes/{id}` - Get school class by ID
- `POST /api/v1/classes` - Create new school class
- `PUT /api/v1/classes/{id}` - Update school class
- `DELETE /api/v1/classes/{id}` - Delete school class

**File Dokumentasi:** `03-school-class-api.md`

---

### 5. Sub Class Module
Mengelola rombongan belajar/sub kelas (misal: Kelas 1A, 1B, dst).

**Endpoints:**
- `GET /api/v1/sub-classes` - Get all sub classes (dengan pagination)
- `GET /api/v1/sub-classes/{id}` - Get sub class by ID
- `POST /api/v1/sub-classes` - Create new sub class
- `PUT /api/v1/sub-classes/{id}` - Update sub class
- `DELETE /api/v1/sub-classes/{id}` - Delete sub class

**File Dokumentasi:** `04-sub-class-api.md`

---

### 6. Student Module
Mengelola data siswa dan operasi bulk.

**Endpoints:**
- `GET /api/v1/students` - Get all students (dengan pagination)
- `GET /api/v1/students/{id}` - Get student by ID
- `POST /api/v1/students` - Create new student
- `PUT /api/v1/students/{id}` - Update student
- `DELETE /api/v1/students/{id}` - Delete student
- `POST /api/v1/students/bulk` - Bulk create students
- `PUT /api/v1/students/bulk` - Bulk update students
- `DELETE /api/v1/students/bulk` - Bulk delete students

**File Dokumentasi:** `05-student-api.md`

---

### 7. Teacher Module
Mengelola data guru.

**Endpoints:**
- `GET /api/v1/teachers` - Get all teachers (dengan pagination)
- `GET /api/v1/teachers/{id}` - Get teacher by ID
- `POST /api/v1/teachers` - Create new teacher
- `PUT /api/v1/teachers/{id}` - Update teacher
- `DELETE /api/v1/teachers/{id}` - Delete teacher

**File Dokumentasi:** `06-teacher-api.md`

---

### 8. Parent Module
Mengelola data orang tua siswa dengan sistem registrasi OTP.

**Endpoints:**
- `POST /api/v1/parents/register` - Register parent dengan OTP
- `POST /api/v1/parents/verify-otp` - Verify OTP untuk parent
- `GET /api/v1/parents` - Get all parents (dengan pagination)
- `GET /api/v1/parents/{id}` - Get parent by ID
- `PUT /api/v1/parents/{id}` - Update parent
- `DELETE /api/v1/parents/{id}` - Delete parent

**File Dokumentasi:** `07-parent-api.md`

---

### 9. Scholarship Module
Mengelola data beasiswa.

**Endpoints:**
- `GET /api/v1/scholarships` - Get all scholarships (dengan pagination)
- `GET /api/v1/scholarships/{id}` - Get scholarship by ID
- `POST /api/v1/scholarships` - Create new scholarship
- `PUT /api/v1/scholarships/{id}` - Update scholarship
- `DELETE /api/v1/scholarships/{id}` - Delete scholarship

**File Dokumentasi:** `08-scholarship-api.md`

---

### 10. Student Enrollment Module
Mengelola pendaftaran siswa per tahun ajaran dan riwayat mutasi/transfer.

**Endpoints:**
- `GET /api/v1/student-enrollments` - Get all enrollments (dengan pagination)
- `GET /api/v1/student-enrollments/{id}` - Get enrollment by ID
- `GET /api/v1/student-enrollments/student/{studentId}` - Get enrollments by student
- `GET /api/v1/student-enrollments/student/{studentId}/academic-year/{academicYearId}` - Get specific enrollment
- `POST /api/v1/student-enrollments` - Create new enrollment
- `PUT /api/v1/student-enrollments/{id}` - Update enrollment
- `DELETE /api/v1/student-enrollments/{id}` - Delete enrollment
- `GET /api/v1/student-enrollments/transfer-history` - Get all transfer history
- `GET /api/v1/student-enrollments/transfer-history/{id}` - Get transfer history by ID
- `GET /api/v1/student-enrollments/transfer-history/student/{studentId}` - Get transfer history by student
- `GET /api/v1/student-enrollments/transfer-history/academic-year/{academicYearId}` - Get transfer history by academic year
- `GET /api/v1/student-enrollments/transfer-history/student/{studentId}/paginated` - Get paginated transfer history

**File Dokumentasi:** `09-student-enrollment-api.md`

---

### 11. Master Billing (MBillings) Module
Mengelola template tagihan master yang digunakan untuk generate tagihan bulanan.

**Endpoints:**
- `GET /api/v1/mbillings` - Get all master billings (dengan pagination)
- `GET /api/v1/mbillings/{id}` - Get master billing by ID
- `GET /api/v1/mbillings/uuid/{uuid}` - Get master billing by UUID
- `POST /api/v1/mbillings` - Create new master billing
- `PUT /api/v1/mbillings/{id}` - Update master billing
- `PATCH /api/v1/mbillings/monthly-active` - Update monthly active status
- `PATCH /api/v1/mbillings/{id}/disable` - Disable master billing
- `PATCH /api/v1/mbillings/{id}/enable` - Enable master billing
- `DELETE /api/v1/mbillings/{id}` - Delete master billing

**File Dokumentasi:** `MBILLINGS_SCHEMA.md`

---

### 12. Billing Module
Mengelola tagihan yang di-generate dari master billing.

**Endpoints:**
- `GET /api/v1/billings` - Get all billings (dengan pagination)
- `GET /api/v1/billings/{id}` - Get billing by ID
- `GET /api/v1/billings/uuid/{uuid}` - Get billing by UUID
- `POST /api/v1/billings` - Create new billing
- `PUT /api/v1/billings/{id}` - Update billing
- `DELETE /api/v1/billings/{id}` - Delete billing
- `POST /api/v1/billings/generate-monthly` - Generate monthly billings
- `POST /api/v1/billings/add-user` - Add users to billing

---

### 13. User Billing Module
Mengelola tagihan per user/siswa.

**Endpoints:**
- `GET /api/v1/user-billings` - Get all user billings (dengan pagination)
- `GET /api/v1/user-billings/{id}` - Get user billing by ID
- `GET /api/v1/user-billings/uuid/{uuid}` - Get user billing by UUID
- `GET /api/v1/user-billings/user/{billedUserId}` - Get user billings by user ID
- `GET /api/v1/user-billings/status/{paymentStatus}` - Get user billings by payment status
- `POST /api/v1/user-billings` - Create new user billing
- `PUT /api/v1/user-billings/{id}` - Update user billing
- `DELETE /api/v1/user-billings/{id}` - Delete user billing

**File Dokumentasi:** `AUTO_GENERATE_USER_BILLING.md`

---

### 14. User Scholarship Module
Mengelola beasiswa yang diberikan ke siswa (bulk operations supported).

**Endpoints:**
- `GET /api/v1/user-scholarships` - Get all user scholarships (dengan pagination)
- `GET /api/v1/user-scholarships/{id}` - Get user scholarship by ID
- `GET /api/v1/user-scholarships/user/{userId}` - Get scholarships by user
- `GET /api/v1/user-scholarships/scholarship/{scholarshipId}` - Get users by scholarship
- `POST /api/v1/user-scholarships` - Create new user scholarship
- `POST /api/v1/user-scholarships/bulk` - Bulk assign scholarships
- `GET /api/v1/user-scholarships/bulk` - Get bulk scholarship assignments
- `PUT /api/v1/user-scholarships/bulk` - Bulk update scholarships
- `PUT /api/v1/user-scholarships/{id}` - Update user scholarship
- `DELETE /api/v1/user-scholarships/{id}` - Delete user scholarship

---

### 15. Billing Scholarship Module
Mengelola relasi antara tagihan dan beasiswa (diskon/potongan).

**Endpoints:**
- `GET /api/v1/billing-scholarships` - Get all billing scholarships (dengan pagination)
- `GET /api/v1/billing-scholarships/{id}` - Get billing scholarship by ID
- `GET /api/v1/billing-scholarships/billing/{billingId}` - Get by billing ID
- `GET /api/v1/billing-scholarships/scholarship/{scholarshipId}` - Get by scholarship ID
- `POST /api/v1/billing-scholarships` - Create new billing scholarship
- `PUT /api/v1/billing-scholarships/{id}` - Update billing scholarship
- `DELETE /api/v1/billing-scholarships/{id}` - Delete billing scholarship

---

### 16. Seeder Module
Utility untuk populate data sample/testing (development only).

**Endpoints:**
- `POST /api/v1/seeders/all` - Seed all data
- `GET /api/v1/seeders/info` - Get seeder information

---

## Response Format Variants

Qrion API mendukung 3 format response untuk endpoint list/pagination:

### 1. Standard Format (Default)
```json
{
  "success": true,
  "message": "Data berhasil diambil",
  "data": {
    "content": [...],
    "page": 0,
    "size": 10,
    "totalElements": 100,
    "totalPages": 10,
    "first": true,
    "last": false
  }
}
```

### 2. jQuery DataTable Format
Tambahkan parameter `format=jquery-datatable` atau `draw` parameter.

```json
{
  "success": true,
  "message": "Data berhasil diambil",
  "data": [...],
  "draw": 1,
  "recordsTotal": 100,
  "recordsFiltered": 100
}
```

### 3. Ant Design Table Format
Tambahkan parameter `format=ant-table`.

```json
{
  "success": true,
  "message": "Data berhasil diambil",
  "data": [...],
  "total": 100,
  "current": 1,
  "pageSize": 10
}
```

---

## Multi-Tenant Filtering

Semua data secara otomatis difilter berdasarkan:
- **yayasanId** (Foundation ID) - dari JWT token
- **institutionId** (Institution ID) - dari JWT token

User hanya dapat melihat dan mengakses data yang sesuai dengan yayasan dan institusi mereka.

**Contoh:**
```
User A (yayasanId=1, institutionId=1) → hanya melihat data institusi 1
User B (yayasanId=1, institutionId=2) → hanya melihat data institusi 2
```

---

## Pagination Parameters

Semua endpoint list mendukung pagination dengan parameter berikut:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | Integer | 0 | Nomor halaman (0-indexed) |
| size | Integer | 10 | Jumlah item per halaman |
| sortBy | String | id | Field untuk sorting |
| sortDirection | String | ASC | Arah sorting (ASC/DESC) |
| format | String | standard | Format response (standard/jquery-datatable/ant-table) |
| draw | Integer | - | Draw counter untuk jQuery DataTable |

**Contoh Request:**
```
GET /api/v1/students?page=0&size=20&sortBy=name&sortDirection=ASC&format=ant-table

**Endpoint Base Path:** `/api/v1/academic-years`

**Fitur:**
- Create tahun akademik baru
- Read (list semua, get by ID)
- Update informasi tahun akademik
- Delete (soft delete)

**Entities Terkait:** Yayasan, Institusi, Semester

---

### 2. Semester Module
Mengelola semester dalam tahun akademik.

**File Dokumentasi:** `02-semester-api.md`

**Endpoint Base Path:** `/api/v1/semesters`

**Fitur:**
- Create semester baru
- Read (list semua, get by ID, get by academic year)
- Update informasi semester
- Delete (soft delete)

**Entities Terkait:** Tahun Akademik, Institusi

---

### 3. School Class Module
Mengelola kelas di institusi.

**File Dokumentasi:** `03-school-class-api.md`

**Endpoint Base Path:** `/api/v1/school-classes`

**Fitur:**
- Create kelas baru dengan kode unik
- Read (list semua, get by ID, get by code)
- Update informasi kelas
- Delete (soft delete)
- Validasi kode duplikat

**Entities Terkait:** Institusi, Sub Kelas

---

### 4. Sub Class Module
Mengelola divisi kelas dalam semester tertentu.

**File Dokumentasi:** `04-sub-class-api.md`

**Endpoint Base Path:** `/api/v1/sub-classes`

**Fitur:**
- Create sub kelas dengan kapasitas
- Read (list semua, get by ID, get by school class)
- Update informasi sub kelas
- Delete (soft delete)

**Entities Terkait:** Kelas, Semester, Siswa

---

### 5. Student Module
Mengelola siswa dengan dukungan pembuatan akun pengguna otomatis.

**File Dokumentasi:** `05-student-api.md`

**Endpoint Base Path:** `/api/v1/students`

**Fitur:**
- Create siswa dengan akun pengguna otomatis
- Create siswa tanpa akun pengguna
- Read (list semua, get by ID, get by sub class)
- Update informasi siswa
- Delete (soft delete)
- Validasi NISN unik 16 digit
- Validasi email unik

**Entities Terkait:** Sub Kelas, Institusi, Position, User Account

---

### 6. Teacher Module
Mengelola guru dengan dukungan pembuatan akun pengguna otomatis.

**File Dokumentasi:** `06-teacher-api.md`

**Endpoint Base Path:** `/api/v1/teachers`

**Fitur:**
- Create guru dengan akun pengguna otomatis
- Create guru tanpa akun pengguna
- Read (list semua, get by ID)
- Update informasi guru
- Delete (soft delete)
- Validasi NIP unik 16 digit
- Validasi email unik

**Entities Terkait:** Institusi, Position, User Account

---

## Common Query Parameters

Mayoritas endpoint GET mengunakan query parameters berikut untuk scope security:

### Yayasan & Institusi Scope
```
GET /api/v1/academic-years?yayasanId=1&institutionId=1
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| yayasanId | Integer | ✅ | ID Yayasan untuk filtering data |
| institutionId | Integer | ✅ | ID Institusi untuk filtering data |

**Catatan:** Query parameters ini memastikan data hanya diakses dalam scope yang sesuai dengan user's organization.

---

## Common Field Validations

### Standard Fields
```json
{
  "name": {
    "type": "String",
    "min": 2,
    "max": 100,
    "message": "Nama harus antara 2 hingga 100 karakter"
  },
  "email": {
    "type": "String",
    "pattern": "valid email",
    "unique": true,
    "message": "Format email tidak valid" / "Email sudah terdaftar"
  },
  "status": {
    "type": "String",
    "options": ["ACTIVE", "INACTIVE"],
    "message": "Status wajib diisi"
  }
}
```

### ID Fields
Semua ID field harus bernilai > 0 dan harus exist di database.

### Date Fields
Menggunakan format ISO 8601: `yyyy-MM-ddTHH:mm:ss`

Contoh: `2024-07-01T00:00:00`

---

## Soft Delete Behavior

Semua entity menggunakan soft delete pattern:

- Saat delete endpoint dipanggil, field `deletedAt` diisi dengan timestamp
- Record tetap ada di database tapi hidden dari query normal
- Recovery tersedia jika diperlukan restore

### Database Query Impact
```sql
-- Normal query (excludes deleted records)
SELECT * FROM academic_year WHERE deleted_at IS NULL

-- Recovery query (only deleted records)
SELECT * FROM academic_year WHERE deleted_at IS NOT NULL
```

---

## Audit Fields

Semua entity memiliki audit fields otomatis:

| Field | Type | Description |
|-------|------|-------------|
| createdAt | LocalDateTime | Timestamp saat record dibuat |
| updatedAt | LocalDateTime | Timestamp saat record terakhir diupdate |
| deletedAt | LocalDateTime | Timestamp saat record dihapus (soft delete) |
| createdBy | String | User ID yang membuat record (opsional) |
| updatedBy | String | User ID yang terakhir update record (opsional) |

---

## UUID Fields

Beberapa entity memiliki UUID field untuk data sensitivity:

| Entity | UUID Field |
|--------|-----------|
| Student | uuid |
| Teacher | uuid |

UUID di-generate otomatis dan dapat digunakan untuk public API reference.

---

## Response Pagination (Future Feature)

Untuk endpoint yang mengembalikan list besar, pagination akan diimplementasikan dengan format:

```json
{
  "success": true,
  "message": "Data retrieved successfully",
  "data": [
    { /* item 1 */ },
    { /* item 2 */ }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

Query parameter: `?page=1&pageSize=20`

---

## Rate Limiting (Future Feature)

API akan menerapkan rate limiting dengan header response:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640190331
```

---

## Support & Documentation

- **API Base URL:** http://localhost:8080/api/v1
- **Documentation Folder:** `.docs/`
- **Module Documentation Files:**
  - `01-academic-year-api.md`
  - `02-semester-api.md`
  - `03-school-class-api.md`
  - `04-sub-class-api.md`
  - `05-student-api.md`
  - `06-teacher-api.md`

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-22 | Initial release dengan 6 core modules |

---

**Last Updated:** 2025-11-22

**Dokumentasi dibuat menggunakan:** Spring Boot 3.x, Jakarta EE 10.0, Java 21
