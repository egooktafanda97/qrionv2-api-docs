# Semester API Documentation

## Overview

Semester API mengelola semester dalam tahun ajaran.

**Base URL**: `/api/semesters`

**Features**:
- ✅ CRUD lengkap untuk semester
- ✅ Terkait dengan Academic Year
- ✅ Soft delete
- ✅ Auto-fill context dari JWT

---

## Authentication

```
Authorization: Bearer {token}
```

**Context dari Token** (otomatis diisi):
- `yayasanId`
- `institutionId`

---

## Endpoints

### 1. List Semesters

**GET** `/api/semesters`

**Response**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "academicYearId": 1,
      "name": "GANJIL",
      "startDate": "2025-07-01",
      "endDate": "2025-12-31",
      "status": "ACTIVE"
    }
  ]
}
```

---

### 2. Get Semester by ID

**GET** `/api/semesters/{id}`

---

### 3. Create Semester

**POST** `/api/semesters`

**Request**:
```json
{
  "academicYearId": 1,
  "name": "GANJIL",
  "startDate": "2025-07-01",
  "endDate": "2025-12-31",
  "status": "ACTIVE"
}
```

**Fields**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| academicYearId | Integer | ✅ | ID tahun ajaran |
| name | String | ✅ | Nama semester (GANJIL/GENAP) |
| startDate | String | ✅ | Format yyyy-MM-dd |
| endDate | String | ✅ | Format yyyy-MM-dd |
| status | String | ❌ | ACTIVE/INACTIVE (default INACTIVE) |

---

### 4. Update Semester

**PUT** `/api/semesters/{id}`

Request body sama dengan Create.

---

### 5. Delete Semester

**DELETE** `/api/semesters/{id}`

Soft delete - data tidak dihapus permanen.

---

## Data Models

```typescript
interface SemesterRequest {
  academicYearId: number;  // Required
  name: string;            // Required (GANJIL/GENAP)
  startDate: string;       // yyyy-MM-dd
  endDate: string;         // yyyy-MM-dd
  status?: string;         // ACTIVE/INACTIVE
}

interface SemesterResponse {
  id: number;
  academicYearId: number;
  name: string;
  startDate: string;
  endDate: string;
  status: string;
  yayasanId: number;
  institutionId: number;
  createdAt: string;
  updatedAt: string;
}
```

---

## Validation Rules

- ✅ **academicYearId**: Required, must exist
- ✅ **name**: Required, max 100 chars
- ✅ **startDate**: Required, yyyy-MM-dd format
- ✅ **endDate**: Required, must be after startDate
- ✅ **status**: ACTIVE or INACTIVE only
- ✅ Semester period must be within academic year period

---

## Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| 1001 | 400 | Validation failed |
| 2001 | 404 | Semester or Academic Year not found |
| 2002 | 409 | Duplicate semester name in same academic year |

---

## Frontend Example

```typescript
// Fetch semesters
const semesters = await fetch('/api/semesters', {
  headers: { 'Authorization': `Bearer ${token}` }
}).then(r => r.json());

// Create semester
await fetch('/api/semesters', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    academicYearId: 1,
    name: 'GANJIL',
    startDate: '2025-07-01',
    endDate: '2025-12-31',
    status: 'ACTIVE'
  })
});
```

---

## Best Practices

1. **Nama Semester**: Gunakan GANJIL atau GENAP
2. **Period Validation**: Pastikan semester period dalam range academic year
3. **Status**: Set ACTIVE hanya untuk semester yang sedang berjalan
4. **Dependencies**: Cek academic year exist sebelum create semester
