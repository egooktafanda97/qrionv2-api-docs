# Teacher API Documentation

## Overview

Teacher API mengelola data guru/staff dengan fitur lengkap termasuk auto-create user account.

**Base URL**: `/api/teachers`

**Features**:
- ✅ CRUD lengkap untuk guru/staff
- ✅ Auto-create User account
- ✅ Auto-generate NIP (optional)
- ✅ Auto-generate username dari nama
- ✅ Position-based (Kepala Sekolah, Guru, Staff, dll)
- ✅ Soft delete (teacher + user account)
- ✅ Custom validators

---

## Authentication

```
Authorization: Bearer {token}
```

**Context dari Token** (otomatis diisi):
- `yayasanId`
- `institutionId`
- `userId`

---

## Endpoints

### 1. List Teachers

**GET** `/api/teachers`

**Response**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "nip": "198501012010011001",
      "name": "Dr. Ahmad Sukirman",
      "gender": "L",
      "dateOfBirth": "1985-01-01",
      "placeOfBirth": "Jakarta",
      "religion": "ISLAM",
      "phone": "628123456789",
      "positionId": 1,
      "positionName": "Kepala Sekolah",
      "isTeacher": true,
      "userId": 5
    }
  ]
}
```

---

### 2. Get by ID

**GET** `/api/teachers/{id}`

---

### 3. Create Teacher

**POST** `/api/teachers`

**Request**:
```json
{
  "name": "Dr. Ahmad Sukirman",
  "gender": "L",
  "dateOfBirth": "1985-01-01",
  "placeOfBirth": "Jakarta",
  "religion": "ISLAM",
  "phone": "628123456789",
  "positionId": 1,
  "isTeacher": true,
  "fullAddress": "Jl. Pendidikan No. 10",
  "city": "Jakarta",
  "province": "DKI Jakarta",
  "district": "Menteng"
}
```

**Fields**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| nip | String | ❌ | Nomor Induk Pegawai (auto-generate jika kosong) |
| name | String | ✅ | Nama lengkap |
| gender | String | ✅ | L (Laki-laki) atau P (Perempuan) |
| dateOfBirth | String | ✅ | Format yyyy-MM-dd, harus tanggal lampau |
| placeOfBirth | String | ✅ | Tempat lahir |
| religion | String | ✅ | ISLAM, KRISTEN, KATOLIK, HINDU, BUDDHA, KONGHUCHU |
| phone | String | ❌ | Format 62xxx |
| positionId | Integer | ✅ | ID posisi/jabatan (dari m_positions) |
| isTeacher | Boolean | ✅ | true = Guru, false = Staff |
| fullAddress | String | ❌ | Alamat lengkap |
| city | String | ❌ | Kota |
| province | String | ❌ | Provinsi |
| district | String | ❌ | Kecamatan |

**Auto-Generated Fields**:
- **NIP**: Auto-generate jika kosong (18 digits)
- **Username**: Dari nama, lowercase, no spaces
- **Password**: Default "qrion@phoenix"
- **User Status**: INACTIVE (perlu aktivasi)

**Response (201)**:
```json
{
  "success": true,
  "message": "Teacher created successfully",
  "data": {
    "id": 1,
    "nip": "198501012010011001",
    "name": "Dr. Ahmad Sukirman",
    "username": "ahmadsukirman"
  }
}
```

**Cara Kerja Backend**:
1. Validasi request (@ValidGender, @ValidReligion)
2. Auto-generate NIP jika kosong
3. Create User account:
   - Username dari nama (lowercase, no spaces)
   - Password: "qrion@phoenix"
   - Status: INACTIVE
4. Create Teacher dengan link ke User
5. Return TeacherResponse

---

### 4. Update Teacher

**PUT** `/api/teachers/{id}`

Request body sama dengan Create.

---

### 5. Delete Teacher

**DELETE** `/api/teachers/{id}`

**Soft Delete Behavior**:
- Set `deletedAt` pada Teacher
- Set `deletedAt` pada User account
- Data tidak dihapus permanen

---

## Data Models

```typescript
interface TeacherRequest {
  nip?: string;           // Optional, auto-generated
  name: string;           // Required
  gender: 'L' | 'P';      // Required
  dateOfBirth: string;    // yyyy-MM-dd, past date
  placeOfBirth: string;   // Required
  religion: string;       // Required
  phone?: string;         // Format 62xxx
  positionId: number;     // Required
  isTeacher: boolean;     // Required
  fullAddress?: string;
  city?: string;
  province?: string;
  district?: string;
}

interface TeacherResponse {
  id: number;
  nip: string;
  name: string;
  gender: string;
  dateOfBirth: string;
  placeOfBirth: string;
  religion: string;
  phone?: string;
  positionId: number;
  positionName: string;
  isTeacher: boolean;
  userId: number;
  institutionId: number;
  yayasanId: number;
  fullAddress?: string;
  city?: string;
  province?: string;
  district?: string;
  createdAt: string;
  updatedAt: string;
}
```

---

## Validation Rules

- ✅ **name**: Required, max 255 chars
- ✅ **gender**: L atau P (custom validator)
- ✅ **dateOfBirth**: Required, yyyy-MM-dd, must be past
- ✅ **religion**: Enum validation (custom validator)
- ✅ **phone**: Pattern 62xxx, min 10 max 15 digits
- ✅ **positionId**: Required, must exist in m_positions
- ✅ **isTeacher**: Required boolean

---

## Available Positions

Positions yang tersedia di sistem (m_positions table):

| ID | Position Name | Description |
|----|---------------|-------------|
| 1 | Kepala Sekolah | Principal |
| 2 | Wakil Kepala Sekolah | Vice Principal |
| 3 | Guru Kelas | Homeroom Teacher |
| 4 | Guru Mata Pelajaran | Subject Teacher |
| 5 | Wali Kelas | Class Guardian |
| 6 | Guru BK | Guidance Counselor |
| 7 | Staff TU | Administrative Staff |
| 8 | Staff Perpustakaan | Library Staff |
| 9 | Staff Kebersihan | Cleaning Staff |
| 10 | Satpam | Security |

---

## Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| 1001 | 400 | Validation failed |
| 2001 | 404 | Teacher or Position not found |
| 2002 | 409 | Duplicate NIP |
| 3001 | 403 | Permission denied |

---

## Frontend Example

```typescript
interface Teacher {
  id: number;
  nip: string;
  name: string;
  gender: 'L' | 'P';
  positionName: string;
  isTeacher: boolean;
}

const teacherAPI = {
  getAll: async (): Promise<Teacher[]> => {
    const res = await fetch('/api/teachers', {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    const data = await res.json();
    return data.data;
  },

  create: async (teacher: TeacherRequest): Promise<Teacher> => {
    const res = await fetch('/api/teachers', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(teacher)
    });

    if (!res.ok) throw await res.json();
    
    const data = await res.json();
    return data.data;
  }
};

// Usage
const newTeacher = await teacherAPI.create({
  name: 'Dr. Ahmad Sukirman',
  gender: 'L',
  dateOfBirth: '1985-01-01',
  placeOfBirth: 'Jakarta',
  religion: 'ISLAM',
  phone: '628123456789',
  positionId: 1,  // Kepala Sekolah
  isTeacher: true
});
```

---

## Best Practices

### 1. Position Selection
```typescript
// Fetch available positions first
const positions = await fetch('/api/positions').then(r => r.json());

// Use position ID in teacher creation
const teacher = {
  positionId: positions.find(p => p.name === 'Guru Kelas').id
};
```

### 2. isTeacher Flag
- **true**: Untuk Guru (mengajar di kelas)
- **false**: Untuk Staff (TU, Perpustakaan, dll)

### 3. NIP Format
- Auto-generate jika kosong
- Jika manual, gunakan format standar (18 digits)

---

## FAQ

**Q: Perbedaan isTeacher true vs false?**  
A: true = Guru (bisa mengajar), false = Staff (tidak mengajar)

**Q: Position mana untuk Kepala Sekolah?**  
A: positionId = 1, dengan isTeacher = true

**Q: Apakah NIP wajib diisi?**  
A: Tidak, sistem akan auto-generate jika kosong.

**Q: Password default untuk teacher?**  
A: "qrion@phoenix" - harus diganti setelah login pertama.

**Q: Soft delete teacher menghapus user juga?**  
A: Ya, cascade soft delete ke user account.
