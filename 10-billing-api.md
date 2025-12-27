# Billing API Documentation

## Overview

Billing API mengelola master billing template untuk institusi. Billing adalah template tagihan yang dapat diaplikasikan ke banyak siswa secara bersamaan.

**Base URL**: `/api/billing`

**Features**:
- ✅ CRUD lengkap billing
- ✅ Monthly billing generation (auto-create untuk semua siswa)
- ✅ General billing (manual assignment)
- ✅ Auto-calculate total dengan discount dan scholarship
- ✅ Payment status tracking
- ✅ Add users (students) to billing
- ✅ Paginated listing dengan filter
- ✅ Multiple response formats (standard, jQuery DataTable, Ant Design)

**Billing Types**:
- **MONTHLY**: Tagihan bulanan (SPP, uang makan, dll) - auto-assign ke semua siswa aktif
- **GENERAL**: Tagihan insidental (seragam, buku, study tour, dll) - manual assignment

---

## Authentication

```
Authorization: Bearer {token}
```

**Context dari Token** (otomatis diisi):
- `yayasanId`
- `institutionId`
- `userId` (release user)

---

## Endpoints

### 1. List Billings (Paginated)

**GET** `/api/billing`

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

**Response**:
```json
{
  "data": [
    {
      "id": 1,
      "uuid": "550e8400-e29b-41d4-a716-446655440000",
      "billingName": "SPP Bulan Januari 2025",
      "academicYearId": 1,
      "academicYearName": "2024/2025",
      "billCategory": "MONTHLY",
      "month": 1,
      "year": 2025,
      "total": 500000,
      "dueDate": "2025-01-10",
      "releaseDate": "2025-01-01",
      "status": "ACTIVE",
      "yayasanId": 1,
      "institutionId": 1
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

---

### 2. Get Billing by ID

**GET** `/api/billing/{id}`

**Path Parameters**:
- `id` (Long): Billing ID

**Response**:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "billingName": "SPP Bulan Januari 2025",
    "description": "Pembayaran SPP untuk bulan Januari",
    "academicYearId": 1,
    "billCategory": "MONTHLY",
    "month": 1,
    "year": 2025,
    "total": 500000,
    "dueDate": "2025-01-10",
    "releaseDate": "2025-01-01",
    "status": "ACTIVE",
    "mBillings": [
      {
        "id": 1,
        "billingName": "Uang Sekolah",
        "nominal": 400000
      },
      {
        "id": 2,
        "billingName": "Uang Makan",
        "nominal": 100000
      }
    ]
  }
}
```

---

### 3. Get Billing by UUID

**GET** `/api/billing/uuid/{uuid}`

**Path Parameters**:
- `uuid` (String): Billing UUID

**Response**: Same as Get by ID

---

### 4. Create Billing (Monthly)

**POST** `/api/billing`

**Request (Monthly Billing)**:
```json
{
  "billingName": "SPP Bulan Januari 2025",
  "description": "Pembayaran SPP untuk bulan Januari",
  "academicYearId": 1,
  "billCategory": "MONTHLY",
  "month": 1,
  "year": 2025,
  "dueDate": "2025-01-10",
  "releaseDate": "2025-01-01",
  "mBillings": [
    {
      "id": 1,
      "nominal": 400000
    },
    {
      "id": 2,
      "nominal": 100000
    }
  ]
}
```

**Request (General Billing)**:
```json
{
  "billingName": "Seragam Olahraga",
  "description": "Pembelian seragam olahraga tahun ajaran baru",
  "academicYearId": 1,
  "billCategory": "GENERAL",
  "dueDate": "2025-02-15",
  "releaseDate": "2025-01-20",
  "mBillings": [
    {
      "id": 5,
      "nominal": 250000
    }
  ]
}
```

**Fields**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| billingName | String | ✅ | Nama billing (max 255 chars) |
| description | String | ❌ | Deskripsi detail billing |
| academicYearId | Integer | ✅ | ID tahun ajaran |
| billCategory | String | ✅ | MONTHLY or GENERAL |
| month | Integer | ✅* | 1-12 (required if MONTHLY) |
| year | Integer | ✅* | YYYY (required if MONTHLY) |
| dueDate | String | ✅ | Due date (yyyy-MM-dd) |
| releaseDate | String | ✅ | Release date (yyyy-MM-dd) |
| mBillings | Array | ✅ | Array of m_billing items with id and nominal |

**Response (201)**:
```json
{
  "success": true,
  "message": "Billing created successfully",
  "data": {
    "id": 1,
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "billingName": "SPP Bulan Januari 2025",
    "total": 500000
  }
}
```

**Business Logic**:
1. Validasi request (bill category, dates, m_billings)
2. Calculate total dari semua m_billings
3. Auto-generate UUID
4. Set yayasanId, institutionId dari token
5. Set releaseUserId dari token
6. Set status = ACTIVE
7. Create Billing record
8. **Jika MONTHLY**: Auto-create UserBilling untuk semua siswa aktif di academic year
9. Return BillingResponse

---

### 5. Update Billing

**PUT** `/api/billing/{id}`

**Request**: Same as Create

**Notes**:
- Update hanya mengubah data Billing, tidak mengubah UserBilling yang sudah ada
- Untuk menambah user ke billing, gunakan endpoint `/api/billing/{id}/add-users`

---

### 6. Delete Billing

**DELETE** `/api/billing/{id}`

**Soft Delete Behavior**:
- Set `deletedAt` pada Billing
- Set `deletedAt` pada semua UserBilling terkait
- Data tidak dihapus permanen

**Response (204)**:
```json
{
  "success": true,
  "message": "Billing deleted successfully"
}
```

---

### 7. Generate Monthly Billing

**POST** `/api/billing/generate-monthly`

**Use Case**: Generate tagihan SPP bulanan untuk semua siswa

**Request**:
```json
{
  "academicYearId": 1,
  "month": 2,
  "year": 2025,
  "billingName": "SPP Bulan Februari 2025",
  "description": "Pembayaran SPP untuk bulan Februari",
  "dueDate": "2025-02-10",
  "releaseDate": "2025-02-01",
  "mBillings": [
    {
      "id": 1,
      "nominal": 400000
    },
    {
      "id": 2,
      "nominal": 100000
    }
  ]
}
```

**Response**:
```json
{
  "success": true,
  "message": "Monthly billing generated successfully for 150 students",
  "data": {
    "id": 5,
    "billingName": "SPP Bulan Februari 2025",
    "totalStudents": 150,
    "totalAmount": 75000000
  }
}
```

**Business Logic**:
1. Validasi month/year combination belum ada
2. Create Billing with MONTHLY category
3. Query all active students in academic year
4. For each student:
   - Check scholarship eligibility
   - Calculate discount amount
   - Create UserBilling with correct amount
5. Return summary

---

### 8. Get Payment Status by Billing ID

**GET** `/api/billing/{id}/payment-status`

**Response**:
```json
{
  "success": true,
  "data": {
    "billingId": 1,
    "billingName": "SPP Bulan Januari 2025",
    "totalStudents": 150,
    "paid": 120,
    "unpaid": 25,
    "partial": 5,
    "totalAmount": 75000000,
    "paidAmount": 60000000,
    "unpaidAmount": 15000000,
    "percentage": 80.0
  }
}
```

---

### 9. Get Payment Status by Billing UUID

**GET** `/api/billing/uuid/{uuid}/payment-status`

**Response**: Same as Get Payment Status by ID

---

### 10. Add Users to Billing

**POST** `/api/billing/{id}/add-users`

**Use Case**: Assign general billing ke siswa tertentu

**Request**:
```json
{
  "userIds": [10, 11, 12, 13, 14, 15]
}
```

**Response**:
```json
{
  "success": true,
  "message": "6 users added to billing successfully",
  "data": {
    "billingId": 3,
    "addedCount": 6,
    "skippedCount": 0,
    "failedCount": 0
  }
}
```

**Business Logic**:
1. Validate billing exists dan tidak deleted
2. For each userId:
   - Check if user already has this billing
   - If not, create UserBilling
   - Apply scholarship if eligible
3. Return summary

---

## Data Models

### BillingRequest
```typescript
interface BillingRequest {
  billingName: string;        // Required, max 255
  description?: string;       // Optional
  academicYearId: number;     // Required
  billCategory: 'MONTHLY' | 'GENERAL';  // Required
  month?: number;             // Required if MONTHLY (1-12)
  year?: number;              // Required if MONTHLY (YYYY)
  dueDate: string;            // Required, yyyy-MM-dd
  releaseDate: string;        // Required, yyyy-MM-dd
  mBillings: Array<{          // Required, min 1 item
    id: number;               // MBilling ID
    nominal: number;          // Amount for this item
  }>;
}
```

### BillingResponse
```typescript
interface BillingResponse {
  id: number;
  uuid: string;
  billingName: string;
  description?: string;
  academicYearId: number;
  academicYearName?: string;
  billCategory: 'MONTHLY' | 'GENERAL';
  month?: number;
  year?: number;
  total: number;
  dueDate: string;
  releaseDate: string;
  status: 'ACTIVE' | 'INACTIVE' | 'COMPLETED';
  yayasanId: number;
  institutionId: number;
  releaseUserId: number;
  mBillings?: Array<MBillingItem>;
  createdAt: string;
  updatedAt: string;
  deletedAt?: string;
}
```

### BillingPaymentStatusResponse
```typescript
interface BillingPaymentStatusResponse {
  billingId: number;
  billingName: string;
  totalStudents: number;
  paid: number;
  unpaid: number;
  partial: number;
  totalAmount: number;
  paidAmount: number;
  unpaidAmount: number;
  percentage: number;
}
```

---

## Business Rules

### 1. Billing Creation Rules
- billingName cannot be empty
- billCategory must be MONTHLY or GENERAL
- If MONTHLY: month (1-12) and year (YYYY) are required
- If GENERAL: month and year must be null
- dueDate must be >= releaseDate
- releaseDate must be >= today
- mBillings array must have at least 1 item
- Each mBilling must have valid id and positive nominal

### 2. Monthly Billing Rules
- Cannot create duplicate monthly billing for same month/year/academicYear
- Auto-creates UserBilling for ALL active students in academic year
- Applies scholarship discount automatically if student has active scholarship
- Sets paymentStatus = UNPAID by default

### 3. General Billing Rules
- Does NOT auto-create UserBilling
- Must manually assign users via add-users endpoint
- Can be assigned to specific students only
- Useful for optional items (seragam, buku, study tour)

### 4. Scholarship Integration
- System automatically checks if student has active scholarship
- If yes, applies discount percentage to billing amount
- Scholarship discount is calculated per UserBilling, not per Billing
- Formula: `finalAmount = totalAmount - (totalAmount * discountPercentage / 100)`

### 5. Payment Status Calculation
- **UNPAID**: paidAmount = 0
- **PARTIAL**: 0 < paidAmount < totalAmount
- **PAID**: paidAmount >= totalAmount
- Percentage = (totalPaidAmount / totalBillingAmount) × 100

---

## Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| 3000 | 404 | Billing not found |
| 3001 | 404 | Academic year not found |
| 3002 | 404 | MBilling not found |
| 1006 | 409 | Duplicate monthly billing (same month/year/academicYear) |
| 1000 | 400 | Invalid bill category |
| 1000 | 400 | Month/year required for MONTHLY billing |
| 1000 | 400 | Month/year must be null for GENERAL billing |
| 1002 | 400 | Invalid date format |
| 1005 | 400 | Due date must be >= release date |
| 1005 | 400 | Release date must be >= today |
| 1001 | 400 | mBillings array cannot be empty |

---

## Related Modules

- **User Billing**: Individual student billing records (11-user-billing-api.md)
- **M-Billings**: Master billing item templates (15-m-billings-api.md)
- **Billing Scholarship**: Scholarship discounts on billing (16-billing-scholarship-api.md)
- **Transaction Biller**: Payment processing (12-transaction-biller-api.md)
- **Academic Year**: Academic year management (01-academic-year-api.md)

---

## Common Use Cases

### Use Case 1: Create Monthly SPP
```
1. POST /api/billing/generate-monthly
2. System creates Billing with MONTHLY category
3. System auto-creates UserBilling for all active students
4. System applies scholarship discounts
5. Students can see their bills in mobile app
```

### Use Case 2: Create General Billing for Specific Students
```
1. POST /api/billing (with GENERAL category)
2. POST /api/billing/{id}/add-users (select specific students)
3. System creates UserBilling only for selected students
4. Selected students can see the bill
```

### Use Case 3: Check Payment Status
```
1. GET /api/billing/{id}/payment-status
2. View how many students have paid
3. Calculate percentage completion
4. Identify students who haven't paid
```

---

## Notes

- Billing adalah **template**, UserBilling adalah **instance per siswa**
- MONTHLY billing = auto-assign ke semua siswa
- GENERAL billing = manual assign
- Scholarship discount diterapkan di level UserBilling, bukan Billing
- Soft delete digunakan untuk audit trail
- UUID digunakan untuk public-facing identifiers
- releaseDate mengontrol kapan billing visible untuk siswa
- dueDate mengontrol kapan billing overdue

---

**Last Updated**: December 22, 2025  
**Version**: 1.0  
**Status**: Production Ready
