# Parent API Documentation

## Overview

Parent API menyediakan endpoint untuk registrasi orang tua siswa dengan sistem **verifikasi OTP**. Parent hanya bisa mendaftar jika nomor telepon mereka sudah terdaftar di data siswa (`Students.parentPhone`). Setelah registrasi, parent akan menerima kode OTP via WhatsApp untuk aktivasi akun dan otomatis terhubung dengan siswa-siswa mereka.

**Base URL:** `/api/parents`

## Authentication

- **Register** dan **Verify OTP**: Tidak memerlukan authentication
- **Get, Delete**: Memerlukan JWT Bearer Token

## Business Rules

1. **Registrasi Parent**:
   - Phone number HARUS sudah ada di `Students.parentPhone`
   - Username harus unik (lowercase, angka, underscore)
   - Phone format: `62xxx` (Indonesia international format)
   - Setelah registrasi sukses → User status `INACTIVE` → OTP dikirim via WhatsApp

2. **Verifikasi OTP**:
   - OTP berlaku 5 menit
   - Setelah verifikasi sukses:
     - User status berubah menjadi `ACTIVE`
     - Parent otomatis di-link ke **SEMUA** siswa yang memiliki `parentPhone` yang sama
     - Field `Students.parent` di-update untuk semua siswa terkait

3. **Parent-Student Relationship**:
   - Satu parent bisa memiliki banyak siswa (`@OneToMany`)
   - Siswa bisa memiliki satu parent (`@ManyToOne`)
   - Linking berdasarkan kecocokan `Parents.phone` dengan `Students.parentPhone`

## Endpoints

### 1. Register Parent

**Endpoint:** `POST /api/parents/register`

**Description:** Registrasi parent baru dengan validasi nomor telepon ke database siswa. Status awal `INACTIVE`, OTP dikirim untuk aktivasi.

**Request Body:**
```json
{
  "name": "Budi Santoso",
  "phone": "6281234567890",
  "username": "budisantoso",
  "password": "password123"
}
```

**Request Validation:**
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| name | String | Yes | Not blank, max 255 characters |
| phone | String | Yes | Format `62[0-9]{9,13}` (Indonesia +62) |
| username | String | Yes | 3-50 chars, lowercase, alphanumeric + underscore only |
| password | String | Yes | 6-100 chars |

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Registrasi berhasil! Kode OTP telah dikirim ke nomor WhatsApp Anda. Silakan verifikasi dalam 5 menit untuk mengaktifkan akun.",
  "data": {
    "id": 1,
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "userId": 10,
    "name": "Budi Santoso",
    "username": "budisantoso",
    "phone": "6281234567890",
    "userStatus": "INACTIVE",
    "totalStudents": 0,
    "createdAt": "2025-06-15T10:30:00",
    "updatedAt": "2025-06-15T10:30:00"
  }
}
```

**Error Responses:**

| Code | Message | Scenario |
|------|---------|----------|
| 400 | Nomor telepon harus dalam format internasional Indonesia (62xxx)... | Format phone salah |
| 400 | Username hanya boleh mengandung huruf kecil, angka, dan underscore | Format username salah |
| 400 | Password harus antara 6-100 karakter | Password terlalu pendek/panjang |
| 404 | Nomor telepon {phone} tidak terdaftar sebagai nomor orang tua siswa... | Phone tidak ada di Students table |
| 409 | Nomor telepon {phone} sudah terdaftar... | Parent dengan phone ini sudah ada |
| 409 | Username {username} sudah digunakan... | Username sudah dipakai |
| 500 | Gagal mengirim kode OTP... | OTP service error |

---

### 2. Verify OTP

**Endpoint:** `POST /api/parents/verify-otp`

**Description:** Verifikasi kode OTP untuk aktivasi akun parent. Setelah sukses, parent di-link ke semua siswa dengan `parentPhone` yang sama.

**Request Body:**
```json
{
  "phone": "6281234567890",
  "otpCode": "123456"
}
```

**Request Validation:**
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| phone | String | Yes | Format `62[0-9]{9,13}` |
| otpCode | String | Yes | Exactly 6 digits |

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Verifikasi berhasil! Akun Anda telah aktif dan terhubung dengan 2 siswa.",
  "data": {
    "id": 1,
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "userId": 10,
    "name": "Budi Santoso",
    "username": "budisantoso",
    "phone": "6281234567890",
    "userStatus": "ACTIVE",
    "totalStudents": 2,
    "createdAt": "2025-06-15T10:30:00",
    "updatedAt": "2025-06-15T10:35:00"
  }
}
```

**Error Responses:**

| Code | Message | Scenario |
|------|---------|----------|
| 400 | Kode OTP harus 6 digit | Format OTP salah |
| 401 | Kode OTP yang Anda masukkan salah atau sudah kadaluarsa... | OTP invalid/expired |
| 404 | Data orang tua tidak ditemukan... | Parent tidak ditemukan setelah OTP valid |

---

### 3. Get All Parents

**Endpoint:** `GET /api/parents`

**Description:** Mendapatkan daftar semua parent (tidak termasuk yang sudah dihapus).

**Headers:**
```
Authorization: Bearer {JWT_TOKEN}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data orang tua berhasil diambil",
  "data": [
    {
      "id": 1,
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "userId": 10,
      "name": "Budi Santoso",
      "username": "budisantoso",
      "gender": "LAKI_LAKI",
      "placeOfBirth": "Jakarta",
      "dateOfBirth": "1980-05-15",
      "religion": "ISLAM",
      "province": "DKI Jakarta",
      "city": "Jakarta Selatan",
      "district": "Kebayoran Baru",
      "fullAddress": "Jl. Melawai No. 123",
      "phone": "6281234567890",
      "userStatus": "ACTIVE",
      "totalStudents": 2,
      "createdAt": "2025-06-15T10:30:00",
      "updatedAt": "2025-06-15T10:35:00"
    }
  ]
}
```

---

### 4. Get Parent by ID

**Endpoint:** `GET /api/parents/{id}`

**Description:** Mendapatkan detail parent berdasarkan ID.

**Path Parameters:**
- `id` (Integer): Parent ID

**Headers:**
```
Authorization: Bearer {JWT_TOKEN}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data orang tua berhasil diambil",
  "data": {
    "id": 1,
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "userId": 10,
    "name": "Budi Santoso",
    "username": "budisantoso",
    "phone": "6281234567890",
    "userStatus": "ACTIVE",
    "totalStudents": 2,
    "createdAt": "2025-06-15T10:30:00",
    "updatedAt": "2025-06-15T10:35:00"
  }
}
```

**Error Responses:**

| Code | Message | Scenario |
|------|---------|----------|
| 404 | Data orang tua dengan ID {id} tidak ditemukan. | Parent tidak ada/sudah dihapus |

---

### 5. Get Parent by Phone

**Endpoint:** `GET /api/parents/phone/{phone}`

**Description:** Mendapatkan detail parent berdasarkan nomor telepon.

**Path Parameters:**
- `phone` (String): Parent phone number (format: `62xxx`)

**Headers:**
```
Authorization: Bearer {JWT_TOKEN}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data orang tua berhasil diambil",
  "data": {
    "id": 1,
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "userId": 10,
    "name": "Budi Santoso",
    "username": "budisantoso",
    "phone": "6281234567890",
    "userStatus": "ACTIVE",
    "totalStudents": 2,
    "createdAt": "2025-06-15T10:30:00",
    "updatedAt": "2025-06-15T10:35:00"
  }
}
```

**Error Responses:**

| Code | Message | Scenario |
|------|---------|----------|
| 404 | Data orang tua dengan nomor {phone} tidak ditemukan. | Parent tidak ada/sudah dihapus |

---

### 6. Delete Parent (Soft Delete)

**Endpoint:** `DELETE /api/parents/{id}`

**Description:** Menghapus parent (soft delete). Parent akan di-unlink dari siswa dan user account juga akan di-soft delete.

**Path Parameters:**
- `id` (Integer): Parent ID

**Headers:**
```
Authorization: Bearer {JWT_TOKEN}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Data orang tua berhasil dihapus",
  "data": null
}
```

**Error Responses:**

| Code | Message | Scenario |
|------|---------|----------|
| 404 | Data orang tua dengan ID {id} tidak ditemukan. | Parent tidak ada |

---

## Data Models

### ParentRegisterRequest
```typescript
{
  name: string;         // Required, max 255
  phone: string;        // Required, format 62[0-9]{9,13}
  username: string;     // Required, 3-50 chars, lowercase + underscore
  password: string;     // Required, 6-100 chars
}
```

### ParentVerifyOtpRequest
```typescript
{
  phone: string;        // Required, format 62[0-9]{9,13}
  otpCode: string;      // Required, exactly 6 digits
}
```

### ParentResponse
```typescript
{
  id: number;
  uuid: string;
  userId: number;
  name: string;
  username: string;
  gender?: "LAKI_LAKI" | "PEREMPUAN";
  placeOfBirth?: string;
  dateOfBirth?: string;      // ISO 8601 date
  religion?: "ISLAM" | "KRISTEN" | "KATOLIK" | "HINDU" | "BUDDHA" | "KONGHUCU";
  province?: string;
  city?: string;
  district?: string;
  fullAddress?: string;
  phone: string;
  userStatus: "ACTIVE" | "INACTIVE" | "SUSPENDED" | "DELETED";
  totalStudents: number;
  createdAt: string;         // ISO 8601 datetime
  updatedAt: string;         // ISO 8601 datetime
}
```

---

## Error Codes Reference

| Code | HTTP Status | Description |
|------|-------------|-------------|
| INVALID_INPUT | 400 | Format input tidak valid (phone, username, password) |
| MISSING_FIELD | 400 | Field required tidak diisi |
| AUTH_FAILED | 401 | OTP invalid atau expired |
| RESOURCE_NOT_FOUND | 404 | Parent atau phone tidak ditemukan |
| DUPLICATE_ENTRY | 409 | Phone atau username sudah terdaftar |
| EXTERNAL_SERVICE_ERROR | 500 | Gagal kirim OTP |
| INTERNAL_ERROR | 500 | Error server internal |

---

## Complete Workflow Example

### Step 1: Pastikan Data Siswa Ada

Sebelum parent bisa registrasi, pastikan ada siswa dengan `parentPhone` yang sesuai:

```sql
-- Cek di database
SELECT id, name, nis, parentPhone 
FROM students 
WHERE parentPhone = '6281234567890' 
  AND deleted_at IS NULL;
```

### Step 2: Register Parent

```bash
curl -X POST http://localhost:8080/api/parents/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Budi Santoso",
    "phone": "6281234567890",
    "username": "budisantoso",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Registrasi berhasil! Kode OTP telah dikirim ke nomor WhatsApp Anda...",
  "data": {
    "id": 1,
    "userStatus": "INACTIVE",
    "totalStudents": 0
  }
}
```

### Step 3: Verify OTP

Parent akan menerima OTP via WhatsApp. Gunakan kode tersebut untuk verifikasi:

```bash
curl -X POST http://localhost:8080/api/parents/verify-otp \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "6281234567890",
    "otpCode": "123456"
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Verifikasi berhasil! Akun Anda telah aktif dan terhubung dengan 2 siswa.",
  "data": {
    "id": 1,
    "userStatus": "ACTIVE",
    "totalStudents": 2
  }
}
```

### Step 4: Login & Get Data

Setelah verifikasi, parent bisa login untuk mendapatkan JWT token, lalu akses data:

```bash
# Login (gunakan endpoint authentication yang sudah ada)
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "budisantoso",
    "password": "password123"
  }'

# Get parent data
curl -X GET http://localhost:8080/api/parents/1 \
  -H "Authorization: Bearer {JWT_TOKEN}"
```

---

## Frontend Integration (React/TypeScript)

### Register Parent Form

```typescript
import { useState } from 'react';
import axios from 'axios';

interface ParentRegisterRequest {
  name: string;
  phone: string;
  username: string;
  password: string;
}

interface ApiResponse<T> {
  success: boolean;
  message: string;
  data: T;
  errorCode?: string;
}

export function ParentRegisterForm() {
  const [formData, setFormData] = useState<ParentRegisterRequest>({
    name: '',
    phone: '',
    username: '',
    password: ''
  });
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string>('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    try {
      const response = await axios.post<ApiResponse<any>>(
        'http://localhost:8080/api/parents/register',
        formData
      );

      if (response.data.success) {
        alert(response.data.message);
        // Redirect ke halaman verifikasi OTP
        window.location.href = `/verify-otp?phone=${formData.phone}`;
      }
    } catch (err: any) {
      const errorMsg = err.response?.data?.message || 'Terjadi kesalahan';
      setError(errorMsg);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="Nama Lengkap"
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        required
      />
      <input
        type="tel"
        placeholder="Nomor HP (62xxx)"
        value={formData.phone}
        onChange={(e) => setFormData({ ...formData, phone: e.target.value })}
        pattern="^62[0-9]{9,13}$"
        required
      />
      <input
        type="text"
        placeholder="Username"
        value={formData.username}
        onChange={(e) => setFormData({ ...formData, username: e.target.value })}
        pattern="^[a-z0-9_]{3,50}$"
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={formData.password}
        onChange={(e) => setFormData({ ...formData, password: e.target.value })}
        minLength={6}
        required
      />
      {error && <div className="error">{error}</div>}
      <button type="submit" disabled={loading}>
        {loading ? 'Mendaftar...' : 'Daftar'}
      </button>
    </form>
  );
}
```

### Verify OTP Form

```typescript
import { useState } from 'react';
import axios from 'axios';
import { useSearchParams } from 'react-router-dom';

interface ParentVerifyOtpRequest {
  phone: string;
  otpCode: string;
}

export function VerifyOtpForm() {
  const [searchParams] = useSearchParams();
  const phone = searchParams.get('phone') || '';
  
  const [otpCode, setOtpCode] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string>('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    try {
      const response = await axios.post<ApiResponse<any>>(
        'http://localhost:8080/api/parents/verify-otp',
        { phone, otpCode }
      );

      if (response.data.success) {
        alert(response.data.message);
        // Redirect ke halaman login
        window.location.href = '/login';
      }
    } catch (err: any) {
      const errorMsg = err.response?.data?.message || 'Kode OTP salah';
      setError(errorMsg);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <p>Kode OTP telah dikirim ke {phone}</p>
      <input
        type="text"
        placeholder="Masukkan 6 digit kode OTP"
        value={otpCode}
        onChange={(e) => setOtpCode(e.target.value)}
        pattern="[0-9]{6}"
        maxLength={6}
        required
      />
      {error && <div className="error">{error}</div>}
      <button type="submit" disabled={loading}>
        {loading ? 'Memverifikasi...' : 'Verifikasi'}
      </button>
    </form>
  );
}
```

---

## Best Practices

1. **Phone Number Format**
   - Selalu gunakan format internasional Indonesia: `62xxx`
   - Validasi di frontend sebelum kirim ke backend
   - Contoh: `0812-3456-7890` → `6281234567890`

2. **Username Guidelines**
   - Lowercase only
   - Alphanumeric + underscore
   - 3-50 characters
   - Contoh valid: `budi_santoso`, `parent123`, `ayah_andi`

3. **OTP Security**
   - OTP berlaku 5 menit
   - Jangan hardcode OTP di production
   - Implement rate limiting untuk prevent brute force
   - Log semua attempt verifikasi OTP

4. **Error Handling**
   - Tampilkan error message yang human-friendly
   - Jangan expose technical details ke user
   - Log semua error untuk debugging

5. **Testing**
   - Test dengan phone number yang ada di Students table
   - Test edge cases: duplicate phone, invalid OTP, expired OTP
   - Test linking ke multiple students

---

## FAQ

**Q: Bagaimana jika parent punya 2 anak di sekolah berbeda?**  
A: Saat ini sistem mengasumsikan parent hanya bisa terdaftar di satu yayasan/institusi. Jika butuh multi-institution, perlu enhancement.

**Q: Apa yang terjadi jika parent di-delete?**  
A: Soft delete - parent record `deletedAt` diisi, user account di-soft delete, dan students di-unlink (`Students.parent = null`).

**Q: Bagaimana cara resend OTP?**  
A: Belum ada endpoint resend OTP saat ini. Parent harus registrasi ulang atau tambahkan endpoint `/api/parents/resend-otp`.

**Q: Apakah parent bisa update profile mereka sendiri?**  
A: Belum ada endpoint update saat ini. Perlu ditambahkan `PUT /api/parents/{id}` atau `PATCH /api/parents/me`.

**Q: Bagaimana cara parent melihat daftar anak mereka?**  
A: Buat endpoint baru: `GET /api/parents/{id}/students` atau tambahkan field `students` di `ParentResponse`.

---

## Changelog

### Version 1.0.0 (2025-06-15)
- Initial release
- Register parent dengan OTP verification
- Auto-link parent ke students berdasarkan phone number
- CRUD operations untuk parent
- Standardized error handling dengan ApiErrorCode
- Indonesian error messages
