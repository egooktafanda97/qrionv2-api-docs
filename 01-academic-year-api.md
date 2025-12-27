# Academic Year API Documentation

## Daftar Isi
1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Endpoints](#endpoints)
4. [Data Models](#data-models)
5. [Response Format Variants](#response-format-variants)
6. [Validation Rules](#validation-rules)
7. [Error Handling](#error-handling)
8. [Frontend Integration Examples](#frontend-integration-examples)

---

## Overview

Academic Year API mengelola data tahun ajaran di institusi pendidikan.

**Base URL**: `/api/academic-years`

**Features**:
- ✅ CRUD lengkap (Create, Read, Update, Delete)
- ✅ Pagination dengan 3 format response (Standard, jQuery DataTable, Ant Design)
- ✅ Sorting dan filtering
- ✅ Auto-fill context (yayasanId, institutionId dari JWT)
- ✅ Multi-tenant support

---

## Authentication

Semua endpoint memerlukan JWT Bearer Token:

```
Authorization: Bearer {your-jwt-token}
Content-Type: application/json
```

### Auto-Fill dari JWT Token
Field berikut otomatis diisi dari JWT token dan **TIDAK perlu dikirim** dalam request body:
- `yayasanId` - ID Yayasan dari user login
- `institutionId` - ID Institusi dari user login

User hanya dapat mengakses data yang sesuai dengan yayasan dan institusi mereka.

---

## Endpoints

### 1. Get All Academic Years (Paginated)

Mengambil daftar semua tahun ajaran dengan pagination.

**Endpoint**: `GET /api/academic-years`

**Query Parameters**:

| Parameter | Type | Default | Required | Description |
|-----------|------|---------|----------|-------------|
| page | Integer | 0 | No | Nomor halaman (0-indexed) |
| size | Integer | 10 | No | Jumlah item per halaman |
| sortBy | String | id | No | Field untuk sorting (id, name, startDate, endDate) |
| sortDirection | String | DESC | No | Arah sorting (ASC atau DESC) |
| format | String | standard | No | Format response (standard/jquery-datatable/ant-table) |
| draw | Integer | - | No | Draw counter untuk jQuery DataTable format |

**Request Example**:
```bash
# Standard format
curl -X GET "http://localhost:8080/api/academic-years?page=0&size=10&sortBy=name&sortDirection=ASC" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# jQuery DataTable format
curl -X GET "http://localhost:8080/api/academic-years?page=0&size=10&draw=1&format=jquery-datatable" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Ant Design Table format
curl -X GET "http://localhost:8080/api/academic-years?page=0&size=10&format=ant-table" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response Success (200)** - Standard Format:
```json
{
  "data": [
    {
      "id": 1,
      "name": "2024/2025",
      "startDate": "2024-07-01",
      "endDate": "2025-06-30",
      "status": "ACTIVE",
      "yayasanId": 1,
      "institutionId": 1,
      "createdAt": "2024-01-15T10:30:00",
      "updatedAt": "2024-01-15T10:30:00"
    }
  ],
  "total": 50,
  "page": 0,
  "size": 10,
  "totalPages": 5,
  "hasNext": true,
  "hasPrevious": false
}
```

**Response Success (200)** - jQuery DataTable Format:
```json
{
  "draw": 1,
  "recordsTotal": 50,
  "recordsFiltered": 50,
  "data": [
    {
      "id": 1,
      "name": "2024/2025",
      "startDate": "2024-07-01",
      "endDate": "2025-06-30",
      "status": "ACTIVE"
    }
  ]
}
```

**Response Success (200)** - Ant Design Table Format:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "2024/2025",
      "startDate": "2024-07-01",
      "endDate": "2025-06-30",
      "status": "ACTIVE"
    }
  ],
  "total": 50,
  "current": 1,
  "pageSize": 10
}
```

---

### 2. Get Academic Year by ID

Mengambil detail satu tahun ajaran berdasarkan ID.

**Endpoint**: `GET /api/academic-years/{id}`

**Path Parameters**:
- `id` (Integer) - ID tahun ajaran

**Request Example**:
```bash
curl -X GET "http://localhost:8080/api/academic-years/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Academic year retrieved successfully",
  "data": {
    "id": 1,
    "name": "2024/2025",
    "startDate": "2024-07-01",
    "endDate": "2025-06-30",
    "status": "ACTIVE",
    "yayasanId": 1,
    "institutionId": 1,
    "createdAt": "2024-01-15T10:30:00",
    "updatedAt": "2024-01-15T10:30:00"
  },
  "status": 200,
  "timestamp": "2025-01-22T08:15:30Z",
  "path": "/api/academic-years/1"
}
```

**Response Error (404)**:
```json
{
  "success": false,
  "message": "Academic year not found with id: 1",
  "errorCode": 3003,
  "status": 404,
  "timestamp": "2025-01-22T08:15:30Z",
  "path": "/api/academic-years/1"
}
```

---

### 3. Create New Academic Year

Membuat tahun ajaran baru.

**Endpoint**: `POST /api/academic-years`

**Request Body**:
```json
{
  "name": "2025/2026",
  "startDate": "2025-07-01",
  "endDate": "2026-06-30",
  "status": "INACTIVE"
}
```

**Request Example**:
```bash
curl -X POST "http://localhost:8080/api/academic-years" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "2025/2026",
    "startDate": "2025-07-01",
    "endDate": "2026-06-30",
    "status": "INACTIVE"
  }'
```

**Response Success (201)**:
```json
{
  "success": true,
  "message": "Academic year created successfully",
  "data": {
    "id": 3,
    "name": "2025/2026",
    "startDate": "2025-07-01",
    "endDate": "2026-06-30",
    "status": "INACTIVE",
    "yayasanId": 1,
    "institutionId": 1,
    "createdAt": "2025-01-22T08:20:00",
    "updatedAt": "2025-01-22T08:20:00"
  },
  "status": 201,
  "timestamp": "2025-01-22T08:20:00Z",
  "path": "/api/academic-years"
}
```

**Response Error (400) - Validation Failed**:
```json
{
  "success": false,
  "message": "Validation failed",
  "errorCode": 1000,
  "status": 400,
  "timestamp": "2025-01-22T08:20:00Z",
  "path": "/api/academic-years",
  "errors": {
    "name": "Name is required and must not be blank",
    "startDate": "Start date is required"
  }
}
```

---

### 4. Update Academic Year

Memperbarui data tahun ajaran yang sudah ada.

**Endpoint**: `PUT /api/academic-years/{id}`

**Path Parameters**:
- `id` (Integer) - ID tahun ajaran yang akan diupdate

**Request Body**: (sama dengan Create)
```json
{
  "name": "2024/2025 (Updated)",
  "startDate": "2024-07-01",
  "endDate": "2025-06-30",
  "status": "ACTIVE"
}
```

**Request Example**:
```bash
curl -X PUT "http://localhost:8080/api/academic-years/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "2024/2025 (Updated)",
    "startDate": "2024-07-01",
    "endDate": "2025-06-30",
    "status": "ACTIVE"
  }'
```

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Academic year updated successfully",
  "data": {
    "id": 1,
    "name": "2024/2025 (Updated)",
    "startDate": "2024-07-01",
    "endDate": "2025-06-30",
    "status": "ACTIVE",
    "yayasanId": 1,
    "institutionId": 1,
    "createdAt": "2024-01-15T10:30:00",
    "updatedAt": "2025-01-22T08:25:00"
  },
  "status": 200,
  "timestamp": "2025-01-22T08:25:00Z",
  "path": "/api/academic-years/1"
}
```

---

### 5. Delete Academic Year

Menghapus tahun ajaran berdasarkan ID.

**Endpoint**: `DELETE /api/academic-years/{id}`

**Path Parameters**:
- `id` (Integer) - ID tahun ajaran yang akan dihapus

**Request Example**:
```bash
curl -X DELETE "http://localhost:8080/api/academic-years/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Academic year deleted successfully",
  "data": null,
  "status": 200,
  "timestamp": "2025-01-22T08:30:00Z",
  "path": "/api/academic-years/1"
}
```

**Response Error (404)**:
```json
{
  "success": false,
  "message": "Academic year not found with id: 1",
  "errorCode": 3003,
  "status": 404,
  "timestamp": "2025-01-22T08:30:00Z",
  "path": "/api/academic-years/1"
}
```

---

## Data Models

### AcademicYearRequest (Request Body untuk Create & Update)

```typescript
interface AcademicYearRequest {
  name: string;          // Required, max 100 characters
  startDate: string;     // Required, format: yyyy-MM-dd
  endDate: string;       // Required, format: yyyy-MM-dd
  status: string;        // Required, valid values: "ACTIVE" | "INACTIVE"
}
```

### AcademicYearResponse (Response Body)

```typescript
interface AcademicYearResponse {
  id: number;
  name: string;
  startDate: string;     // Format: yyyy-MM-dd
  endDate: string;       // Format: yyyy-MM-dd
  status: string;        // "ACTIVE" | "INACTIVE"
  yayasanId: number;
  institutionId: number;
  createdAt: string;     // ISO 8601 format
  updatedAt: string;     // ISO 8601 format
}
```

---

## Response Format Variants

API ini mendukung 3 format response untuk list endpoint:

### 1. Standard Format (Default)
Gunakan tanpa parameter `format` atau `format=standard`

**Response Structure**:
```typescript
{
  data: AcademicYearResponse[];
  total: number;
  page: number;          // 0-indexed
  size: number;
  totalPages: number;
  hasNext: boolean;
  hasPrevious: boolean;
}
```

### 2. jQuery DataTable Format
Gunakan `format=jquery-datatable` atau tambahkan parameter `draw`

**Response Structure**:
```typescript
{
  draw: number;
  recordsTotal: number;
  recordsFiltered: number;
  data: AcademicYearResponse[];
}
```

### 3. Ant Design Table Format
Gunakan `format=ant-table`

**Response Structure**:
```typescript
{
  success: boolean;
  data: AcademicYearResponse[];
  total: number;
  current: number;       // 1-indexed
  pageSize: number;
}
```

---

## Validation Rules

| Field | Rules | Description |
|-------|-------|-------------|
| name | Required, NotBlank, MaxLength(100) | Nama tahun ajaran (contoh: "2024/2025") |
| startDate | Required, ValidDate | Tanggal mulai tahun ajaran (format: yyyy-MM-dd) |
| endDate | Required, ValidDate, AfterStartDate | Tanggal akhir tahun ajaran (harus setelah startDate) |
| status | Required, ValidStatus | Status: "ACTIVE" atau "INACTIVE" |

**Business Rules**:
- `endDate` harus setelah `startDate`
- Hanya satu tahun ajaran dengan status "ACTIVE" yang diperbolehkan per institusi
- `yayasanId` dan `institutionId` otomatis diambil dari JWT token
- User hanya dapat mengelola tahun ajaran di institusi mereka sendiri

---

## Error Handling

### Common Error Codes

| Error Code | HTTP Status | Description | Solution |
|------------|-------------|-------------|----------|
| 1000 | 400 | Validation failed - input tidak valid | Periksa field yang required dan formatnya |
| 1001 | 400 | Missing required field | Pastikan semua field required terisi |
| 1002 | 400 | Invalid date format | Gunakan format yyyy-MM-dd untuk tanggal |
| 1006 | 409 | Duplicate entry - nama sudah digunakan | Gunakan nama tahun ajaran yang berbeda |
| 2000 | 401 | Unauthorized - token tidak ada | Sertakan Authorization header dengan Bearer token |
| 2001 | 401 | Invalid token | Token expired atau tidak valid, lakukan login ulang |
| 2002 | 403 | Forbidden - tidak ada akses | User tidak memiliki permission untuk resource ini |
| 3003 | 404 | Academic year not found | ID tahun ajaran tidak ditemukan |
| 4001 | 400 | Invalid scope | Data tidak sesuai dengan yayasan/institusi user |
| 5000 | 500 | Internal server error | Hubungi administrator |

---

## Frontend Integration Examples

### React with Axios

```typescript
import axios from 'axios';

const API_BASE = 'http://localhost:8080/api/academic-years';
const getToken = () => localStorage.getItem('token');

const api = axios.create({
  baseURL: API_BASE,
  headers: { 'Content-Type': 'application/json' }
});

api.interceptors.request.use((config) => {
  const token = getToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Get all academic years (Standard format)
export const getAcademicYears = async (page = 0, size = 10, sortBy = 'id', sortDirection = 'DESC') => {
  const response = await api.get('', { params: { page, size, sortBy, sortDirection } });
  return response.data;
};

// Get academic years for Ant Design Table
export const getAcademicYearsAntTable = async (page = 0, size = 10) => {
  const response = await api.get('', { params: { page, size, format: 'ant-table' } });
  return response.data;
};

// Get by ID
export const getAcademicYearById = async (id: number) => {
  const response = await api.get(`/${id}`);
  return response.data;
};

// Create
export const createAcademicYear = async (data: {
  name: string; startDate: string; endDate: string; status: string;
}) => {
  const response = await api.post('', data);
  return response.data;
};

// Update
export const updateAcademicYear = async (id: number, data: {
  name: string; startDate: string; endDate: string; status: string;
}) => {
  const response = await api.put(`/${id}`, data);
  return response.data;
};

// Delete
export const deleteAcademicYear = async (id: number) => {
  const response = await api.delete(`/${id}`);
  return response.data;
};
```

### Vanilla JavaScript with Fetch API

```javascript
const API_BASE = 'http://localhost:8080/api/academic-years';
const token = localStorage.getItem('token');

async function getAcademicYears(page = 0, size = 10) {
  const response = await fetch(`${API_BASE}?page=${page}&size=${size}`, {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  return await response.json();
}

async function createAcademicYear(data) {
  const response = await fetch(API_BASE, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  });
  return await response.json();
}
```

---

## Tips untuk Frontend Developer

1. **Authentication**: Simpan JWT token setelah login dan sertakan di setiap request
2. **Error Handling**: Selalu handle error response dan tampilkan pesan yang sesuai ke user
3. **Date Format**: Pastikan menggunakan format `yyyy-MM-dd` saat mengirim tanggal
4. **Pagination**: Gunakan format response yang sesuai dengan library UI yang digunakan
5. **Auto-refresh**: Refresh data setelah operasi create, update, atau delete berhasil
6. **Loading State**: Tampilkan loading indicator saat melakukan request
7. **Validation**: Lakukan validasi di frontend sebelum mengirim request
8. **Multi-tenant**: Tidak perlu mengirim yayasanId dan institutionId, sudah otomatis dari token
