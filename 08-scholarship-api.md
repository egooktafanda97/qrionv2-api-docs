# Scholarship CRUD API Documentation

## Overview
CRUD API untuk mengelola Master Beasiswa (m_scholarship) dengan fitur:
- ✅ Create, Read, Update, Delete (Soft Delete)
- ✅ Multi-tenancy (Filter by Yayasan & Institution)
- ✅ Pagination dengan 3 format (Standard, jQuery DataTable, Ant Design)
- ✅ Validasi input lengkap dengan pesan error yang user-friendly
- ✅ Status beasiswa otomatis (ACTIVE, EXPIRED, UPCOMING)
- ✅ Standarisasi API Response & Error Handling

## Database Schema

### Table: m_scholarship
```sql
CREATE TABLE `m_scholarship` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `uuid` VARCHAR(36) NOT NULL UNIQUE,
  `name` VARCHAR(100) NOT NULL,
  `description` TEXT,
  `start_date` DATE NOT NULL,
  `end_date` DATE NULL,
  `yayasan_id` BIGINT NULL,
  `institution_id` BIGINT NULL,
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `deleted_at` DATETIME NULL,
  KEY `idx_scholarship_yayasan_id` (`yayasan_id`),
  KEY `idx_scholarship_institution_id` (`institution_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Table: billing_scholarship
```sql
CREATE TABLE `billing_scholarship` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `uuid` VARCHAR(36) NOT NULL UNIQUE,
  `scholarship_id` BIGINT NOT NULL,
  `billing_id` BIGINT NOT NULL,
  `amount` DECIMAL(15,2) NOT NULL,
  `notes` TEXT,
  `created_at` DATETIME NULL,
  `updated_at` DATETIME NULL,
  `deleted_at` DATETIME NULL,
  KEY `idx_bs_scholarship_id` (`scholarship_id`),
  KEY `idx_bs_billing_id` (`billing_id`),
  CONSTRAINT `fk_bs_scholarship` FOREIGN KEY (`scholarship_id`) 
    REFERENCES `m_scholarship` (`id`) ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT `fk_bs_billing` FOREIGN KEY (`billing_id`) 
    REFERENCES `billing` (`id`) ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## API Endpoints

### Base URL
```
/api/scholarships
```

### Authentication
Semua endpoint memerlukan Bearer Token:
```
Authorization: Bearer <your_token>
```

---

## 1. List All Scholarships (Paginated)

**Endpoint:** `GET /api/scholarships`

**Query Parameters:**
- `page` (optional, default: 0) - Halaman ke-
- `size` (optional, default: 10) - Jumlah data per halaman
- `sortBy` (optional, default: id) - Field untuk sorting
- `sortDirection` (optional, default: DESC) - ASC atau DESC
- `draw` (optional) - Untuk jQuery DataTable format

**Headers:**
- `format` (optional, default: standard) - Format response: `standard`, `jquery-datatable`, `ant-table`

**Response (Standard Format):**
```json
{
  "data": [
    {
      "id": 1,
      "uuid": "123e4567-e89b-12d3-a456-426614174000",
      "name": "Beasiswa Prestasi Akademik 2025",
      "description": "Program beasiswa untuk siswa berprestasi...",
      "startDate": "2025-07-01",
      "endDate": "2026-06-30",
      "yayasanId": 1,
      "yayasanName": "Yayasan Pendidikan ABC",
      "institutionId": 1,
      "institutionName": "SMA Negeri 1 Jakarta",
      "createdAt": "2025-11-23T10:00:00",
      "updatedAt": "2025-11-23T10:00:00",
      "isActive": true,
      "status": "ACTIVE"
    }
  ],
  "total": 25,
  "page": 0,
  "size": 10,
  "totalPages": 3,
  "hasNext": true,
  "hasPrevious": false
}
```

---

## 2. Get Scholarship by ID

**Endpoint:** `GET /api/scholarships/{id}`

**Response:**
```json
{
  "success": true,
  "message": "Data beasiswa berhasil diambil",
  "data": {
    "id": 1,
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Beasiswa Prestasi Akademik 2025",
    "description": "Program beasiswa untuk siswa berprestasi...",
    "startDate": "2025-07-01",
    "endDate": "2026-06-30",
    "yayasanId": 1,
    "yayasanName": "Yayasan Pendidikan ABC",
    "institutionId": 1,
    "institutionName": "SMA Negeri 1 Jakarta",
    "createdAt": "2025-11-23T10:00:00",
    "updatedAt": "2025-11-23T10:00:00",
    "isActive": true,
    "status": "ACTIVE"
  },
  "errorCode": null,
  "status": null,
  "timestamp": "2025-11-23T10:00:00Z",
  "path": null,
  "errors": null
}
```

---

## 3. Create New Scholarship

**Endpoint:** `POST /api/scholarships`

**Request Body:**
```json
{
  "name": "Beasiswa Prestasi Akademik 2025",
  "description": "Program beasiswa untuk siswa berprestasi dengan nilai rata-rata minimal 85",
  "startDate": "2025-07-01",
  "endDate": "2026-06-30"
}
```

**Required Fields:**
- `name` (string, max 100 chars)
- `startDate` (date, format: YYYY-MM-DD)

**Optional Fields:**
- `description` (string, max 5000 chars)
- `endDate` (date, format: YYYY-MM-DD)

**Response:**
```json
{
  "success": true,
  "message": "Program beasiswa berhasil dibuat",
  "data": {
    "id": 1,
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Beasiswa Prestasi Akademik 2025",
    ...
  }
}
```

---

## 4. Update Scholarship

**Endpoint:** `PUT /api/scholarships/{id}`

**Request Body:**
```json
{
  "name": "Beasiswa Prestasi Akademik 2025 (Updated)",
  "description": "Program beasiswa untuk siswa berprestasi dengan nilai rata-rata minimal 90",
  "startDate": "2025-07-01",
  "endDate": "2026-06-30"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Program beasiswa berhasil diperbarui",
  "data": {
    "id": 1,
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Beasiswa Prestasi Akademik 2025 (Updated)",
    ...
  }
}
```

---

## 5. Delete Scholarship (Soft Delete)

**Endpoint:** `DELETE /api/scholarships/{id}`

**Response:**
```json
{
  "success": true,
  "message": "Program beasiswa berhasil dihapus",
  "data": null
}
```

---

## Error Responses

### 1. Validation Error (1000 - INVALID_INPUT)
```json
{
  "message": "Validation failed",
  "data": null,
  "errorCode": 1000,
  "timestamp": "2025-11-23T10:00:00Z",
  "errors": {
    "name": "Nama program beasiswa wajib diisi",
    "startDate": "Tanggal mulai beasiswa wajib diisi"
  }
}
```
**HTTP Status:** 400 Bad Request

### 2. Duplicate Entry (1006 - DUPLICATE_ENTRY)
```json
{
  "message": "Program beasiswa dengan nama 'Beasiswa Prestasi Akademik 2025' sudah ada. Silakan gunakan nama lain.",
  "data": null,
  "errorCode": 1006,
  "timestamp": "2025-11-23T10:00:00Z"
}
```
**HTTP Status:** 409 Conflict

### 3. Resource Not Found (3000 - RESOURCE_NOT_FOUND)
```json
{
  "message": "Program beasiswa tidak ditemukan atau sudah dihapus",
  "data": null,
  "errorCode": 3000,
  "timestamp": "2025-11-23T10:00:00Z"
}
```
**HTTP Status:** 404 Not Found

### 4. Permission Denied (2003 - PERMISSION_DENIED)
```json
{
  "message": "Anda tidak memiliki akses untuk mengubah program beasiswa ini",
  "data": null,
  "errorCode": 2003,
  "timestamp": "2025-11-23T10:00:00Z"
}
```
**HTTP Status:** 403 Forbidden

### 5. Invalid Date Range (1000 - INVALID_INPUT)
```json
{
  "message": "Tanggal mulai beasiswa tidak boleh lebih besar dari tanggal berakhir",
  "data": null,
  "errorCode": 1000,
  "timestamp": "2025-11-23T10:00:00Z"
}
```
**HTTP Status:** 400 Bad Request

---

## Business Rules

### 1. Multi-Tenancy
- Semua data scholarship difilter berdasarkan `yayasan_id` dan `institution_id` dari user yang login
- User hanya bisa melihat/mengelola scholarship milik yayasan & institusinya sendiri

### 2. Unique Name
- Nama beasiswa harus unik per kombinasi yayasan & institusi
- Validasi saat create dan update

### 3. Date Validation
- `startDate` wajib diisi
- `endDate` opsional (null = beasiswa tidak ada batas waktu)
- Jika `endDate` ada, harus >= `startDate`

### 4. Scholarship Status
Status dihitung otomatis berdasarkan tanggal hari ini:
- **UPCOMING**: Hari ini < startDate (Beasiswa belum dimulai)
- **ACTIVE**: startDate <= Hari ini <= endDate (Beasiswa sedang berjalan)
- **EXPIRED**: Hari ini > endDate (Beasiswa sudah berakhir)

### 5. Soft Delete
- DELETE endpoint tidak menghapus data fisik dari database
- Hanya mengisi field `deleted_at` dengan timestamp
- Data yang sudah di-soft delete tidak muncul di list/get

---

## File Structure

```
src/main/java/com/phoenix/qrion/
├── entities/
│   ├── Scholarship.java              # Entity untuk m_scholarship
│   └── BillingScholarship.java       # Entity untuk billing_scholarship
├── repository/
│   ├── ScholarshipRepository.java    # Repository dengan query JPQL
│   └── BillingScholarshipRepository.java
├── dto/
│   ├── ScholarshipRequest.java       # DTO Request dengan validasi
│   ├── ScholarshipResponse.java      # DTO Response dengan computed fields
│   ├── BillingScholarshipRequest.java
│   └── BillingScholarshipResponse.java
├── service/
│   ├── ScholarshipService.java       # Service Interface
│   └── impl/
│       └── ScholarshipServiceImpl.java # Service Implementation
└── controllers/
    └── ScholarshipController.java     # REST Controller

http/
└── ScholarshipController.http         # HTTP Request untuk testing
```

---

## Testing with HTTP File

Gunakan file `http/ScholarshipController.http` untuk testing:

```http
### 1. List All Scholarships
GET {{host}}/api/scholarships?page=0&size=10
Authorization: Bearer {{token}}

### 2. Get Scholarship by ID
GET {{host}}/api/scholarships/1
Authorization: Bearer {{token}}

### 3. Create New Scholarship
POST {{host}}/api/scholarships
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Beasiswa Prestasi Akademik 2025",
  "description": "Program beasiswa untuk siswa berprestasi",
  "startDate": "2025-07-01",
  "endDate": "2026-06-30"
}

### 4. Update Scholarship
PUT {{host}}/api/scholarships/1
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "name": "Beasiswa Prestasi Akademik 2025 (Updated)",
  "startDate": "2025-07-01",
  "endDate": "2026-06-30"
}

### 5. Delete Scholarship
DELETE {{host}}/api/scholarships/1
Authorization: Bearer {{token}}
```

---

## Next Steps

### BillingScholarship CRUD (Future Implementation)
Untuk mengimplementasikan relasi beasiswa dengan billing:

1. **Entity**: BillingScholarship sudah dibuat
2. **Repository**: BillingScholarshipRepository sudah dibuat
3. **TODO**: Buat Service & Controller untuk:
   - Assign scholarship ke billing
   - Hitung total potongan dari multiple scholarships
   - List billing yang mendapat scholarship tertentu
   - List scholarships yang diterapkan pada billing tertentu

---

## Changelog

### v1.0.0 (2025-11-23)
- ✅ Initial implementation of Scholarship CRUD
- ✅ Multi-tenancy support (Yayasan & Institution filter)
- ✅ Pagination with 3 formats (Standard, jQuery DataTable, Ant Design)
- ✅ Soft delete functionality
- ✅ Auto-computed scholarship status
- ✅ Complete validation with user-friendly error messages
- ✅ Standardized API response & error handling
- ✅ HTTP request file for testing
