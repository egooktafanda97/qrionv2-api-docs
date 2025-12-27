# School Class API Documentation

## Overview

School Class API mengelola kelas di sekolah (contoh: Kelas 1, Kelas 2, dst).

**Base URL**: `/api/school-classes`

**Features**:
- ✅ CRUD lengkap
- ✅ Auto-generate code jika tidak diisi
- ✅ Level-based class (1, 2, 3, dst)
- ✅ Soft delete

---

## Authentication

```
Authorization: Bearer {token}
```

---

## Endpoints

### 1. List School Classes

**GET** `/api/school-classes`

**Response**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "code": "1A",
      "name": "Kelas 1A",
      "level": 1,
      "status": "ACTIVE",
      "institutionId": 1,
      "yayasanId": 1
    }
  ]
}
```

---

### 2. Get by ID

**GET** `/api/school-classes/{id}`

---

### 3. Create School Class

**POST** `/api/school-classes`

**Request (with code)**:
```json
{
  "code": "1A",
  "name": "Kelas 1A",
  "level": 1,
  "status": "ACTIVE"
}
```

**Request (auto-generate code)**:
```json
{
  "name": "Kelas II",
  "level": 2
}
```

**Fields**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| code | String | ❌ | Kode kelas (auto-generate jika kosong) |
| name | String | ✅ | Nama kelas |
| level | Integer | ✅ | Level/tingkat kelas (1, 2, 3, dst) |
| status | String | ❌ | ACTIVE/INACTIVE (default INACTIVE) |

---

### 4. Update

**PUT** `/api/school-classes/{id}`

Request body sama dengan Create.

---

### 5. Delete

**DELETE** `/api/school-classes/{id}`

Soft delete.

---

## Data Models

```typescript
interface SchoolClassRequest {
  code?: string;        // Optional, auto-generated if empty
  name: string;         // Required
  level: number;        // Required (1, 2, 3, ...)
  status?: string;      // ACTIVE/INACTIVE
}

interface SchoolClassResponse {
  id: number;
  code: string;
  name: string;
  level: number;
  status: string;
  institutionId: number;
  yayasanId: number;
  createdAt: string;
  updatedAt: string;
}
```

---

## Validation Rules

- ✅ **name**: Required, max 50 chars
- ✅ **level**: Required, must be positive integer
- ✅ **code**: Max 50 chars, auto-generate if empty (5 alphanumeric chars)
- ✅ **status**: ACTIVE or INACTIVE only

---

## Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| 1001 | 400 | Validation failed |
| 2001 | 404 | School class not found |
| 2002 | 409 | Duplicate code |

---

## Frontend Example

```typescript
// Create class with auto-generated code
await fetch('/api/school-classes', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Kelas 1A',
    level: 1,
    status: 'ACTIVE'
  })
});
```

---

## Best Practices

1. **Auto-generate Code**: Kosongkan field `code` untuk generate otomatis
2. **Level**: Gunakan angka sesuai tingkat (1 untuk kelas 1, dst)
3. **Naming**: Gunakan nama yang jelas (contoh: "Kelas 1A", "Grade 1")
