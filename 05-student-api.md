# Student API Documentation

## Overview

Student API mengelola data siswa dengan fitur lengkap termasuk auto-create user account dan wallet.

**Base URL**: `/api/students`

**Features**:
- ✅ CRUD lengkap siswa
- ✅ Auto-create User account saat create student
- ✅ Auto-create Wallet account untuk student
- ✅ Auto-generate NIS (YYYY + 6 digits)
- ✅ Auto-generate username dari nama
- ✅ Soft delete (siswa + user account)
- ✅ Custom validators dengan pesan Indonesia

---

## Authentication

```
Authorization: Bearer {token}
```

**Context dari Token** (otomatis diisi, **JANGAN kirim**):
- `yayasanId`
- `institutionId`
- `userId` (untuk relasi user)

---

## Endpoints

### 1. List Students (Paginated)

**GET** `/api/students`

**Query Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | Integer | ❌ | 0 | Page number (0-indexed) |
| size | Integer | ❌ | 10 | Items per page |
| sortBy | String | ❌ | "id" | Field to sort by |
| sortDirection | String | ❌ | "DESC" | ASC or DESC |
| draw | Integer | ❌ | null | For jQuery DataTables |

**Headers**:
| Header | Value | Description |
|--------|-------|-------------|
| format | standard / jquery-datatable / ant-table | Response format (default: standard) |

**Response (Standard Format)**:
```json
{
  "data": [
    {
      "id": 1,
      "nis": "2025123456",
      "name": "Budi Santoso",
      "gender": "L",
      "dateOfBirth": "2015-05-01",
      "placeOfBirth": "Jakarta",
      "religion": "ISLAM",
      "phone": "628123456789",
      "fullAddress": "Jl. Merdeka No. 123",
      "studentStatus": "ACTIVE",
      "userId": 10,
      "institutionId": 1,
      "yayasanId": 1
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

### 2. Advanced Search Students

**GET** `/api/students/search`

**Query Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | Integer | ❌ | Page number (0-indexed) |
| size | Integer | ❌ | Items per page |
| sortBy | String | ❌ | Field to sort by |
| sortDirection | String | ❌ | ASC or DESC |
| search | String | ❌ | Free text search across name, nis, uuid, phone, parentName |
| nis | String | ❌ | Filter by NIS (exact match) |
| name | String | ❌ | Filter by name (contains) |
| uuid | String | ❌ | Filter by UUID (exact match) |
| phone | String | ❌ | Filter by phone (contains) |
| parentName | String | ❌ | Filter by parent name (contains) |
| academicYearId | Integer | ❌ | Filter by academic year (requires enrollment) |
| classId | Integer | ❌ | Filter by class (requires enrollment) |
| subClassId | Integer | ❌ | Filter by subclass (requires enrollment) |
| draw | Integer | ❌ | For jQuery DataTables |

**Response Structure**:
Returns enriched data with related entities:
```json
{
  "data": [
    {
      "student": {
        "id": 1,
        "nis": "2025123456",
        "name": "Budi Santoso",
        "gender": "L",
        "dateOfBirth": "2015-05-01"
      },
      "institution": {
        "id": 1,
        "name": "SMA Negeri 1 Jakarta"
      },
      "yayasan": {
        "id": 1,
        "name": "Yayasan Pendidikan Indonesia"
      },
      "user": {
        "id": 10,
        "username": "budisantoso",
        "name": "Budi Santoso"
      },
      "parent": {
        "id": 5,
        "name": "Sugeng Santoso",
        "phone": "6281298765432"
      },
      "latestEnrollment": {
        "academicYearId": 1,
        "classId": 3,
        "subClassId": 5,
        "classDisplay": "10 - IPA 1"
      }
    }
  ],
  "total": 50,
  "page": 0,
  "size": 20,
  "totalPages": 3,
  "hasNext": true,
  "hasPrevious": false
}
```

---

### 3. Get Compact Student List

**GET** `/api/students/compact`

**Query Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| academicYearId | Integer | ✅ | Academic year to filter enrollments |

**Use Case**: Dropdown/select components in frontend

**Response**:
```json
[
  {
    "id": 1,
    "nama": "Budi Santoso",
    "nisn": "2025123456",
    "kelas": "10 - IPA 1",
    "status": "ACTIVE"
  },
  {
    "id": 2,
    "nama": "Siti Nurhaliza",
    "nisn": "2025123457",
    "kelas": "10 - IPA 2",
    "status": "ACTIVE"
  }
]
```

---

### 4. Get by ID

**GET** `/api/students/{id}`

---

### 3. Create Student

**POST** `/api/students`

**Request**:
```json
{
  "name": "Budi Santoso",
  "gender": "L",
  "dateOfBirth": "2015-05-01",
  "placeOfBirth": "Jakarta",
  "religion": "ISLAM",
  "phone": "628123456789",
  "parentName": "Sukarno",
  "parentPhone": "628987654321",
  "fullAddress": "Jl. Merdeka No. 123",
  "city": "Jakarta",
  "province": "DKI Jakarta",
  "district": "Menteng",
  "studentStatus": "ACTIVE"
}
```

**Fields**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | String | ✅ | Nama lengkap siswa |
| gender | String | ✅ | L (Laki-laki) atau P (Perempuan) |
| dateOfBirth | String | ✅ | Format yyyy-MM-dd, harus tanggal lampau |
| placeOfBirth | String | ✅ | Tempat lahir |
| religion | String | ✅ | ISLAM, KRISTEN, KATOLIK, HINDU, BUDDHA, KONGHUCHU |
| phone | String | ❌ | Format 62xxx (opsional) |
| parentName | String | ✅ | Nama orang tua/wali |
| parentPhone | String | ✅ | Nomor HP orang tua (format 62xxx) |
| fullAddress | String | ❌ | Alamat lengkap |
| city | String | ❌ | Kota |
| province | String | ❌ | Provinsi |
| district | String | ❌ | Kecamatan |
| studentStatus | String | ❌ | ACTIVE, INACTIVE, GRADUATED, DROPPED_OUT, SUSPENDED (default ACTIVE) |
| nis | String | ❌ | Auto-generate jika kosong (YYYY + 6 random digits) |

**Auto-Generated Fields**:
- **NIS**: Format YYYY + 6 digits random (contoh: 2025123456)
- **Username**: Dari nama, lowercase, no spaces (contoh: "Budi Santoso" → "budisantoso")
- **Password**: Default "qrion@phoenix"
- **Wallet Account**: Dibuat otomatis dengan AccountType.WALLET

**Response (201)**:
```json
{
  "success": true,
  "message": "Student created successfully",
  "data": {
    "id": 1,
    "nis": "2025123456",
    "name": "Budi Santoso",
    "username": "budisantoso"
    // ... fields lainnya
  }
}
```

**Cara Kerja Backend**:
1. Validasi request (custom validators: @ValidGender, @ValidReligion, @ValidStudentStatus)
2. Auto-generate NIS jika kosong
3. Create User account:
   - Username dari nama (lowercase, no spaces)
   - Password default: "qrion@phoenix"
   - Status: INACTIVE (perlu aktivasi)
   - Set yayasanId, institutionId dari token
4. Create Student:
   - Link ke User yang baru dibuat
   - Set all fields
5. Create Wallet Account:
   - AccountType: WALLET
   - AccountLevel: USER
   - Currency: IDR
   - Auto-generate account number
6. Return StudentResponse

---

### 4. Update Student

**PUT** `/api/students/{id}`

Request body sama dengan Create (kecuali field auto-generated).

**Note**: Update tidak mengubah User account atau Wallet.

---

### 5. Delete Student

**DELETE** `/api/students/{id}`

**Soft Delete Behavior**:
- Set `deletedAt` pada Student
- Set `deletedAt` pada User account terkait
- Data tidak dihapus permanen (masih ada di database)

**Permission Required**:
- User harus punya permission yayasan DAN institution

**Response (204)**:
```json
{
  "success": true,
  "message": "Student deleted successfully"
}
```

---

### 6. Bulk Create Students

**POST** `/api/students/bulk`

**Use Case**: Import students from Excel/CSV file

**Request**:
```json
{
  "students": [
    {
      "name": "Student 1",
      "nis": "2025001",
      "gender": "L",
      "dateOfBirth": "2015-01-15",
      "placeOfBirth": "Jakarta",
      "religion": "ISLAM",
      "parentName": "Parent 1",
      "parentPhone": "6281111111111"
    },
    {
      "name": "Student 2",
      "nis": "2025002",
      "gender": "P",
      "dateOfBirth": "2015-02-20",
      "placeOfBirth": "Bandung",
      "religion": "KRISTEN",
      "parentName": "Parent 2",
      "parentPhone": "6281222222222"
    }
  ]
}
```

**Response**:
```json
{
  "success": true,
  "message": "Bulk create completed: 2 succeeded, 0 failed out of 2 requested",
  "data": {
    "successCount": 2,
    "failedCount": 0,
    "totalRequested": 2,
    "successResults": [
      {
        "id": 1,
        "nis": "2025001",
        "name": "Student 1",
        "status": "created"
      },
      {
        "id": 2,
        "nis": "2025002",
        "name": "Student 2",
        "status": "created"
      }
    ],
    "failedResults": []
  }
}
```

**Notes**:
- Each student in the array follows the same validation rules as single create
- If one student fails, others will still be processed
- Returns detailed success and failure information

---

### 7. Bulk Update Students

**PUT** `/api/students/bulk`

**Use Case**: Update multiple students information at once

**Request**:
```json
{
  "updates": [
    {
      "id": 1,
      "name": "Updated Name 1",
      "phone": "6281999999991"
    },
    {
      "id": 2,
      "nis": "2025999",
      "studentStatus": "GRADUATED"
    },
    {
      "id": 3,
      "province": "Jawa Barat",
      "city": "Bandung"
    }
  ]
}
```

**Response**:
```json
{
  "success": true,
  "message": "Bulk update completed: 3 succeeded, 0 failed out of 3 requested",
  "data": {
    "successCount": 3,
    "failedCount": 0,
    "totalRequested": 3,
    "successResults": [
      {
        "id": 1,
        "nis": "2025001",
        "name": "Updated Name 1",
        "status": "updated"
      }
    ],
    "failedResults": []
  }
}
```

**Notes**:
- Only provide fields that need to be updated
- `id` is required for each update
- Follows same validation rules as single update

---

### 8. Bulk Delete Students

**DELETE** `/api/students/bulk`

**Use Case**: Admin selects multiple students via checkbox and deletes them

**Request**:
```json
{
  "ids": [10, 11, 12, 13, 14]
}
```

**Response**:
```json
{
  "success": true,
  "message": "Bulk delete completed: 5 succeeded, 0 failed out of 5 requested",
  "data": {
    "successCount": 5,
    "failedCount": 0,
    "totalRequested": 5,
    "successResults": [
      {
        "id": 10,
        "status": "deleted"
      },
      {
        "id": 11,
        "status": "deleted"
      }
    ],
    "failedResults": []
  }
}
```

**Notes**:
- Performs soft delete on all students
- Also soft deletes related user accounts
- Returns detailed success/failure for each ID

---

## Data Models

```typescript
interface StudentRequest {
  nis?: string;           // Optional, auto-generated
  name: string;           // Required
  gender: 'L' | 'P';      // Required: L atau P
  dateOfBirth: string;    // Required, yyyy-MM-dd, must be past date
  placeOfBirth: string;   // Required
  religion: 'ISLAM' | 'KRISTEN' | 'KATOLIK' | 'HINDU' | 'BUDDHA' | 'KONGHUCHU';  // Required
  phone?: string;         // Optional, format 62xxx
  parentName: string;     // Required
  parentPhone: string;    // Required, format 62xxx
  fullAddress?: string;
  city?: string;
  province?: string;
  district?: string;
  studentStatus?: 'ACTIVE' | 'INACTIVE' | 'GRADUATED' | 'DROPPED_OUT' | 'SUSPENDED';  // Default ACTIVE
}

interface StudentResponse {
  id: number;
  nis: string;
  name: string;
  gender: string;
  dateOfBirth: string;
  placeOfBirth: string;
  religion: string;
  phone?: string;
  parentName: string;
  parentPhone: string;
  fullAddress?: string;
  city?: string;
  province?: string;
  district?: string;
  studentStatus: string;
  userId: number;
  institutionId: number;
  yayasanId: number;
  createdAt: string;
  updatedAt: string;
  deletedAt?: string;
}
```

---

## Validation Rules

### Name Validation
- ✅ Required
- ✅ Max 255 characters

### Gender Validation (Custom Validator)
- ✅ Hanya boleh "L" (Laki-laki) atau "P" (Perempuan)
- ❌ Error message: "Jenis kelamin tidak valid. Gunakan 'L' untuk Laki-laki atau 'P' untuk Perempuan"

### Religion Validation (Custom Validator)
- ✅ Values: ISLAM, KRISTEN, KATOLIK, HINDU, BUDDHA, KONGHUCHU
- ❌ Error message: "Agama tidak valid. Gunakan: ISLAM, KRISTEN, KATOLIK, HINDU, BUDDHA, atau KONGHUCHU"

### Date of Birth Validation
- ✅ Required
- ✅ Format: yyyy-MM-dd
- ✅ Must be past date (@Past annotation)
- ❌ Error message: "Tanggal lahir harus tanggal di masa lalu"

### Phone Number Validation
- ✅ Pattern: 62xxx (Indonesian format)
- ✅ Min 10 digits, max 15 digits
- ❌ Error message: "Nomor telepon harus dalam format internasional Indonesia (62xxx)"

### Student Status Validation (Custom Validator)
- ✅ Values: ACTIVE, INACTIVE, GRADUATED, DROPPED_OUT, SUSPENDED
- ❌ Error message: "Status siswa tidak valid. Gunakan: ACTIVE, INACTIVE, GRADUATED, DROPPED_OUT, atau SUSPENDED"

---

## Error Handling

### Common Error Codes

| Error Code | HTTP | Description | Solution |
|------------|------|-------------|----------|
| 1001 | 400 | Validation failed | Cek field errors di response |
| 1002 | 400 | Malformed JSON / Invalid enum | Cek format JSON dan enum values |
| 2001 | 404 | Student not found | Pastikan ID valid |
| 2002 | 409 | Duplicate NIS | NIS sudah digunakan siswa lain |
| 3001 | 403 | Permission denied | User tidak punya akses yayasan/institution |

### Validation Error Examples

**Gender Error**:
```json
{
  "success": false,
  "message": "Validation failed",
  "errorCode": 1001,
  "errors": {
    "gender": "Jenis kelamin tidak valid. Gunakan 'L' untuk Laki-laki atau 'P' untuk Perempuan"
  }
}
```

**Religion Error**:
```json
{
  "success": false,
  "errors": {
    "religion": "Agama tidak valid. Gunakan: ISLAM, KRISTEN, KATOLIK, HINDU, BUDDHA, atau KONGHUCHU"
  }
}
```

**Date Format Error** (dari GlobalExceptionHandler):
```json
{
  "success": false,
  "message": "Format tanggal tidak valid. Gunakan format yyyy-MM-dd (contoh: 2015-05-01)",
  "errorCode": 1002
}
```

---

## Frontend Integration

### React/TypeScript Example

```typescript
// Types
interface Student {
  id: number;
  nis: string;
  name: string;
  gender: 'L' | 'P';
  dateOfBirth: string;
  religion: string;
  studentStatus: string;
}

interface CreateStudentRequest {
  name: string;
  gender: 'L' | 'P';
  dateOfBirth: string;
  placeOfBirth: string;
  religion: string;
  parentName: string;
  parentPhone: string;
  studentStatus?: string;
}

// API Service
const studentAPI = {
  getAll: async (): Promise<Student[]> => {
    const res = await fetch('/api/students', {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    const data = await res.json();
    return data.data;
  },

  create: async (student: CreateStudentRequest): Promise<Student> => {
    const res = await fetch('/api/students', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(student)
    });

    if (!res.ok) {
      const error = await res.json();
      throw error;
    }

    const data = await res.json();
    return data.data;
  },

  delete: async (id: number): Promise<void> => {
    const res = await fetch(`/api/students/${id}`, {
      method: 'DELETE',
      headers: { 'Authorization': `Bearer ${token}` }
    });

    if (!res.ok) {
      const error = await res.json();
      throw error;
    }
  }
};

// Component Example
const StudentForm: React.FC = () => {
  const [formData, setFormData] = useState({
    name: '',
    gender: 'L' as 'L' | 'P',
    dateOfBirth: '',
    placeOfBirth: '',
    religion: 'ISLAM',
    parentName: '',
    parentPhone: '',
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      const student = await studentAPI.create(formData);
      alert(`Student created: ${student.name} (NIS: ${student.nis})`);
    } catch (error: any) {
      if (error.errors) {
        // Validation errors
        Object.entries(error.errors).forEach(([field, message]) => {
          console.error(`${field}: ${message}`);
        });
      } else {
        alert(error.message);
      }
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="Nama Lengkap"
        value={formData.name}
        onChange={e => setFormData({...formData, name: e.target.value})}
        required
      />

      <select
        value={formData.gender}
        onChange={e => setFormData({...formData, gender: e.target.value as 'L' | 'P'})}
      >
        <option value="L">Laki-laki</option>
        <option value="P">Perempuan</option>
      </select>

      <input
        type="date"
        value={formData.dateOfBirth}
        onChange={e => setFormData({...formData, dateOfBirth: e.target.value})}
        required
      />

      <select
        value={formData.religion}
        onChange={e => setFormData({...formData, religion: e.target.value})}
      >
        <option value="ISLAM">Islam</option>
        <option value="KRISTEN">Kristen</option>
        <option value="KATOLIK">Katolik</option>
        <option value="HINDU">Hindu</option>
        <option value="BUDDHA">Buddha</option>
        <option value="KONGHUCHU">Konghuchu</option>
      </select>

      <button type="submit">Create Student</button>
    </form>
  );
};
```

---

## Best Practices

### 1. Gender Input
```typescript
// ❌ WRONG
gender: "MALE"  // Will fail validation

// ✅ CORRECT
gender: "L"     // For Laki-laki
gender: "P"     // For Perempuan
```

### 2. Date Format
```typescript
// ❌ WRONG
dateOfBirth: "01/05/2015"        // Wrong format
dateOfBirth: new Date()          // JavaScript Date object

// ✅ CORRECT
dateOfBirth: "2015-05-01"        // yyyy-MM-dd string
```

### 3. Phone Number
```typescript
// ❌ WRONG
phone: "0812345678"              // Must start with 62
parentPhone: "+62 812-345-678"   // No spaces/dashes

// ✅ CORRECT
phone: "628123456789"
parentPhone: "628987654321"
```

### 4. NIS Generation
```typescript
// Let backend auto-generate
const request = {
  name: "Budi",
  // nis: undefined or omit
};

// Backend will generate: "2025123456"
```

### 5. Error Handling
```typescript
try {
  await studentAPI.create(data);
} catch (error: any) {
  if (error.errorCode === 1001) {
    // Validation errors - show field-specific messages
    Object.entries(error.errors).forEach(([field, msg]) => {
      showFieldError(field, msg);
    });
  } else if (error.errorCode === 2002) {
    // Duplicate NIS
    alert('NIS sudah digunakan');
  } else {
    // Generic error
    alert(error.message);
  }
}
```

---

## FAQ

**Q: Apakah NIS harus diisi manual?**  
A: Tidak. Sistem akan auto-generate NIS dengan format YYYY + 6 digits random jika tidak diisi.

**Q: Apa password default untuk user account siswa?**  
A: Password default adalah "qrion@phoenix". Sebaiknya user ganti password setelah login pertama.

**Q: Kenapa harus format 62xxx untuk nomor HP?**  
A: Sistem menggunakan format internasional Indonesia untuk konsistensi dan kompatibilitas dengan WhatsApp OTP.

**Q: Apa yang terjadi saat delete student?**  
A: Soft delete - Student.deletedAt dan User.deletedAt di-set. Data masih ada di database, tidak dihapus permanen.

**Q: Apakah bisa menghapus user account tanpa menghapus student?**  
A: Tidak. Delete student akan cascade delete user account juga.

**Q: Gender selain L/P tidak bisa?**  
A: Saat ini sistem hanya support L (Laki-laki) dan P (Perempuan). Ini sesuai standar Dapodik Indonesia.

**Q: Wallet account dibuat kapan?**  
A: Otomatis saat create student. Tidak perlu request manual.
