# Qrion API - Postman Collection

## 📋 Overview
Complete Postman Collection yang berisi **204 API endpoints** dari Qrion School Management System, diorganisir dalam **17 folders/modules**.

## 📦 File Location
```
Qrion_Complete_API_Collection.postman_collection.json
```

## 🚀 Cara Import ke Postman

### Option 1: Import via File
1. Buka Postman
2. Klik tombol **"Import"** di pojok kiri atas
3. Pilih tab **"File"** atau drag & drop file
4. Browse dan pilih file `Qrion_Complete_API_Collection.postman_collection.json`
5. Klik **"Import"**

### Option 2: Import via Command Line (Postman CLI)
```bash
postman collection import Qrion_Complete_API_Collection.postman_collection.json
```

## 🔧 Configuration

### Environment Variables
Collection ini menggunakan 3 variables yang sudah ter-configure:

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `{{host}}` | `http://localhost:8081` | Base URL API server |
| `{{contentType}}` | `application/json` | Content-Type header |
| `{{token}}` | _(empty)_ | JWT Bearer token untuk authentication |

### Setup Variables
1. Setelah import, buka collection
2. Klik tab **"Variables"**
3. Update `host` jika server berjalan di URL lain
4. Token akan otomatis tersimpan setelah login

## 📚 API Modules

### 1. **Authentication** (4 endpoints)
- Register Demo Institution
- Verify Registration OTP  
- Send OTP for Login
- Verify Login OTP (auto-save token)

### 2. **Authentication Demo** (4 endpoints)
- Demo register dan login flow
- OTP verification

### 3. **AcademicYear** (5 endpoints)
- List/Get/Create/Update/Delete Academic Years

### 4. **Semester** (5 endpoints)
- CRUD operations untuk semester (Ganjil/Genap)

### 5. **SchoolClass** (6 endpoints)
- Manage kelas (Level 1-12)
- Auto-generate class code

### 6. **SubClass** (6 endpoints)
- Manage sub-kelas (A, B, C, dst)

### 7. **Student** (6 endpoints)
- CRUD siswa
- Auto-generate NIS dan User account
- Link dengan parent

### 8. **Parent** (16 endpoints)
- Register parent
- OTP verification
- Link dengan students via phone number

### 9. **MBillings** (16 endpoints) ⭐ *Featured*
- **Type MONTHLY**: Tagihan berulang bulanan (SPP, dll)
- **Type GENERAL**: Tagihan sekali bayar (Buku, Seragam)
- **NEW**: `userBilled` feature untuk assign user ke billing
- Update/Delete dengan soft delete

### 10. **Billing** (36 endpoints)
- Generate billing dari MBillings
- Auto-generate monthly billing
- CRUD operations

### 11. **UserBilling** (31 endpoints)
- Assign billing ke user
- Payment tracking
- Status management

### 12. **BillingScholarship** (24 endpoints)
- Link scholarship dengan billing
- Discount management

### 13. **Scholarship** (16 endpoints)
- CRUD beasiswa
- Multiple table format support (Standard, jQuery DataTable, Ant Design)

### 14. **AutoGenerateUserBilling** (17 endpoints)
- Auto-assign billing ke users berdasarkan kriteria
- Bulk operations

### 15. **Account** (3 endpoints)
- User account management

### 16. **Seeder** (8 endpoints)
- Development tools untuk seed data

### 17. **Debug** (1 endpoint)
- Debugging utilities

## 🔐 Authentication Flow

### Step 1: Login
```http
POST {{host}}/api/demo/login-with-otp
{
  "destination": "6282284733405",
  "channel": "WHATSAPP"
}
```

### Step 2: Verify OTP
```http
POST {{host}}/api/demo/login-otp-verify
{
  "destination": "6282284733405",
  "otpCode": "123456"
}
```

**Response akan berisi `token`** yang otomatis tersimpan di collection variable!

### Step 3: Use Token
Semua endpoint yang butuh auth sudah configured dengan:
```
Authorization: Bearer {{token}}
```

## ⭐ Featured: MBillings dengan userBilled

### Create dengan userBilled
```json
POST {{host}}/api/m-billings
{
  "billingType": "MONTHLY",
  "name": "Biaya SPP",
  "amount": 500000,
  "monthlyActive": [1, 2, 3, 4, 5, 6],
  "userBilled": ["user-uuid-1", "user-uuid-2", "1", "2"]
}
```

### Update - Add/Remove users
```json
PUT {{host}}/api/m-billings/1
{
  "monthlyActive": [1, 2, 3, 4, 5, 6],
  "userBilled": ["user-uuid-3", "3", "4"]
}
```

### Update - Remove all users
```json
PUT {{host}}/api/m-billings/1
{
  "monthlyActive": [1, 2, 3, 4, 5, 6],
  "userBilled": []
}
```

### Response includes billedUsers
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "name": "Biaya SPP",
    "billedUsers": [
      {
        "id": 1,
        "uuid": "billing-user-uuid",
        "userId": 123,
        "userUuid": "user-uuid-1",
        "userName": "John Doe",
        "userEmail": "john@example.com",
        "isActive": true
      }
    ]
  }
}
```

## 📊 Table Format Support

Beberapa endpoint mendukung multiple response format:

### Standard Format (default)
```http
GET {{host}}/api/scholarships
```

### jQuery DataTable Format
```http
GET {{host}}/api/scholarships?draw=1
format: jquery-datatable
```

### Ant Design Table Format  
```http
GET {{host}}/api/scholarships
format: ant-table
```

## 🔄 Pagination

Endpoints yang support pagination:
```http
GET {{host}}/api/students?page=0&size=10&sortBy=id&sortDirection=DESC
```

Parameters:
- `page`: Page number (0-based)
- `size`: Items per page
- `sortBy`: Field name untuk sorting
- `sortDirection`: `ASC` atau `DESC`

## 🧪 Testing

### Test Sequence Recommendation:
1. **Authentication** → Login dan get token
2. **AcademicYear** → Create tahun ajaran
3. **Semester** → Create semester
4. **SchoolClass** → Create kelas
5. **SubClass** → Create sub-kelas
6. **Student** → Create siswa
7. **Parent** → Register dan verify parent
8. **MBillings** → Create master billing
9. **Billing** → Generate actual billing
10. **UserBilling** → Assign billing ke user

## 📝 Notes

- Semua endpoint sudah include proper **Authentication** configuration
- **Soft delete** digunakan di hampir semua delete operations
- Response format mengikuti standard: `{ status, message, data, errorCode }`
- Timestamp fields: `createdAt`, `updatedAt`, `deletedAt`

## 🐛 Error Handling

Standard error response:
```json
{
  "success": false,
  "message": "Error message",
  "errorCode": "ERROR_CODE",
  "data": null
}
```

Common error codes:
- `RESOURCE_NOT_FOUND` (404)
- `DUPLICATE_ENTRY` (409)
- `BUSINESS_RULE_VIOLATION` (400)
- `PERMISSION_DENIED` (403)
- `VALIDATION_ERROR` (400)

## 🔗 Related Files

- Original HTTP files: `/http/*.http`
- API Documentation: `/documentations/*.md`
- Entity Schema: `/src/main/java/com/phoenix/qrion/entities/`

## 📞 Support

Jika ada endpoint yang error atau perlu update, silakan:
1. Check original `.http` files di folder `/http`
2. Regenerate collection dengan script Python yang sama
3. Atau update manual di Postman dan export ulang

---

**Generated from**: 18 HTTP files  
**Total Endpoints**: 204  
**Last Updated**: 2025-11-25  
**Version**: 1.0
