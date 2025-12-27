# Sub Class API Documentation

## Overview

Sub Class API mengelola sub kelas / rombongan belajar dalam satu kelas.

**Base URL**: `/api/sub-classes`

**Features**:
- ✅ CRUD lengkap
- ✅ Terkait dengan School Class
- ✅ Auto-generate code
- ✅ Soft delete

---

## Authentication

```
Authorization: Bearer {token}
```

---

## Endpoints

### 1. List Sub Classes

**GET** `/api/sub-classes`

**Response**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "classId": 1,
      "code": "A1B2C",
      "name": "Sub Kelas A",
      "status": "ACTIVE"
    }
  ]
}
```

---

### 2. Get by ID

**GET** `/api/sub-classes/{id}`

---

### 3. Create Sub Class

**POST** `/api/sub-classes`

**Request**:
```json
{
  "classId": 1,
  "name": "Sub Kelas A",
  "status": "ACTIVE"
}
```

**Fields**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| classId | Integer | ✅ | ID kelas induk |
| code | String | ❌ | Kode sub kelas (auto-generate jika kosong) |
| name | String | ✅ | Nama sub kelas |
| status | String | ❌ | ACTIVE/INACTIVE |

---

### 4. Update

**PUT** `/api/sub-classes/{id}`

---

### 5. Delete

**DELETE** `/api/sub-classes/{id}`

---

## Data Models

```typescript
interface SubClassRequest {
  classId: number;      // Required
  code?: string;        // Optional, auto-generated
  name: string;         // Required
  status?: string;      // ACTIVE/INACTIVE
}

interface SubClassResponse {
  id: number;
  classId: number;
  code: string;
  name: string;
  status: string;
  institutionId: number;
  yayasanId: number;
}
```

---

## Validation Rules

- ✅ **classId**: Required, must exist
- ✅ **name**: Required, max 100 chars
- ✅ **code**: Auto-generate if empty (5 alphanumeric)
- ✅ **status**: ACTIVE or INACTIVE

---

## Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| 1001 | 400 | Validation failed |
| 2001 | 404 | Sub class or School class not found |
| 2002 | 409 | Duplicate code |

---

## Frontend Example

```typescript
// Create sub class
await fetch('/api/sub-classes', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    classId: 1,
    name: 'Rombel A',
    status: 'ACTIVE'
  })
});
```
