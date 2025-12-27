# Auto-Generate Billing Feature

## 📋 Overview
Ketika **MBillings** (Master Billing) dibuat, sistem akan **otomatis generate**:
1. ✅ **Billing** berdasarkan tipe billing dan konfigurasi periode
2. ✅ **UserBilling** untuk setiap user di `billedUsers`

**Semua dalam satu transaksi saat create MBillings!**

## 🔄 Flow Diagram

```
MBillings Created
       ↓
  Check Type
       ↓
   ┌───┴────┐
   ↓        ↓
GENERAL  MONTHLY
   ↓        ↓
Generate Generate
  1x     Multiple
Billing  Billings
   ↓        ↓
   └────┬───┘
        ↓
 Auto Generate
 UserBillings
 (per BilledUser)
```

## 📚 Tipe Billing

### 1. **Type GENERAL** 
Tagihan sekali bayar (Uang Buku, Seragam, Gedung, dll)

**Auto-Generate Logic:**
- Generate **1 Billing** saja
- Collect date dari `startDatePeriod` (atau hari ini jika null)
- Due date = collect date + `dueDateOffset`

**Example:**
```json
POST /api/m-billings
{
  "billingType": "GENERAL",
  "name": "Uang Buku Pelajaran",
  "amount": 350000,
  "collectDate": 10,
  "dueDateOffset": 14,
  "startDatePeriod": "2025-07-01"
}
```

**Result:** 1 Billing dibuat dengan:
- `name`: "Uang Buku Pelajaran"
- `billingCollectDate`: 2025-07-01
- `billingDueDate`: 2025-07-15 (+ 14 hari)
- `amount`: 350000

---

### 2. **Type MONTHLY**
Tagihan berulang bulanan (SPP, Uang Kegiatan, dll)

**Auto-Generate Logic:**
- Generate **multiple Billings** per bulan
- Dari `startDatePeriod` sampai `endDatePeriod`
- Jika `endDatePeriod` null → generate **1 tahun berjalan**
- Hanya generate bulan yang ada di `monthlyActive`
- Billing name format: `"{name} - {MONTH} {YEAR}"`

**Example 1: Full Year**
```json
POST /api/m-billings
{
  "billingType": "MONTHLY",
  "name": "BIAYA SPP",
  "amount": 500000,
  "collectDate": 1,
  "dueDateOffset": 7,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-12-31",
  "monthlyActive": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
}
```

**Result:** 12 Billings dibuat:
```
- BIAYA SPP - JANUARY 2025  (collect: 2025-01-01, due: 2025-01-08)
- BIAYA SPP - FEBRUARY 2025 (collect: 2025-02-01, due: 2025-02-08)
- BIAYA SPP - MARCH 2025    (collect: 2025-03-01, due: 2025-03-08)
...
- BIAYA SPP - DECEMBER 2025 (collect: 2025-12-01, due: 2025-12-08)
```

**Example 2: Skip Bulan Libur (Juli & Agustus)**
```json
POST /api/m-billings
{
  "billingType": "MONTHLY",
  "name": "Uang Kegiatan Bulanan",
  "amount": 200000,
  "collectDate": 15,
  "dueDateOffset": 5,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": null,
  "monthlyActive": [1, 2, 3, 4, 5, 6, 9, 10, 11, 12]
}
```

**Result:** 10 Billings dibuat (skip Juli=7 & Agustus=8):
```
- Uang Kegiatan Bulanan - JANUARY 2025   (collect: 2025-01-15)
- Uang Kegiatan Bulanan - FEBRUARY 2025  (collect: 2025-02-15)
...
- Uang Kegiatan Bulanan - JUNE 2025      (collect: 2025-06-15)
(Skip Juli & Agustus)
- Uang Kegiatan Bulanan - SEPTEMBER 2025 (collect: 2025-09-15)
...
- Uang Kegiatan Bulanan - DECEMBER 2025  (collect: 2025-12-15)
```

**Example 3: Semester (6 Bulan)**
```json
POST /api/m-billings
{
  "billingType": "MONTHLY",
  "name": "Biaya Semester Ganjil",
  "amount": 750000,
  "collectDate": 5,
  "dueDateOffset": 7,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-06-30",
  "monthlyActive": [1, 2, 3, 4, 5, 6]
}
```

**Result:** 6 Billings dibuat (Januari - Juni saja)

---

## 🗓️ Collect Date Logic

### Untuk GENERAL:
```
billingCollectDate = startDatePeriod (atau today jika null)
```

### Untuk MONTHLY:
```
billingCollectDate = {tahun-bulan}-{collectDate}

Example:
- Bulan: Januari 2025
- collectDate: 15
- Result: 2025-01-15

Jika collectDate > hari terakhir bulan:
- Bulan: Februari 2025 (28 hari)
- collectDate: 31
- Result: 2025-02-28 (auto-adjust)
```

---

## 📅 Due Date Logic

```
billingDueDate = billingCollectDate + dueDateOffset

Example:
- collectDate: 2025-01-01
- dueDateOffset: 7
- Result: 2025-01-08

Jika dueDateOffset = null atau 0:
- billingDueDate = billingCollectDate
```

---

## 🔗 Relasi Data

### Database Structure:
```
MBillings (Master)
    ↓ (foreign key)
Billing (Generated)
    ↓ (foreign key)
UserBilling (Assignment ke User)
```

### Field Mapping:

**MBillings → Billing:**
| MBillings Field | → | Billing Field |
|-----------------|---|---------------|
| `id` | → | `m_billing_id` (FK) |
| `yayasan` | → | `yayasan` |
| `institution` | → | `institution` |
| `name` | → | `name` (modified untuk MONTHLY) |
| `amount` | → | `amount` |

**Billing → UserBilling (Auto-Generated):**
| Billing Field | → | UserBilling Field |
|--------------|---|-------------------|
| `id` | → | `billing_id` (FK) |
| `yayasan` | → | `yayasan` |
| `institution` | → | `institution` |
| `amount` | → | `base_amount` |
| - | → | `paid_amount` = 0 |
| - | → | `payment_status` = UNPAID |

**BilledUsers → UserBilling:**
| BilledUsers | → | UserBilling |
|-------------|---|-------------|
| `id` | → | `billed_user_ref_id` (FK) |
| `user_id` | → | `billed_user_id` (FK) |

### Contoh Perhitungan Records

**Input:**
```json
{
  "billingType": "MONTHLY",
  "amount": 500000,
  "monthlyActive": [1, 2, 3],
  "userBilled": ["student1", "student2", "student3"],
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-03-31"
}
```

**Output:**
- **MBillings**: 1 record
- **Billings**: 3 records (Jan, Feb, Mar)
- **UserBillings**: 9 records (3 billings × 3 students)

**Detail UserBillings:**
```
JANUARY 2025:
- Student 1: Rp 500.000 - UNPAID
- Student 2: Rp 500.000 - UNPAID
- Student 3: Rp 500.000 - UNPAID

FEBRUARY 2025:
- Student 1: Rp 500.000 - UNPAID
- Student 2: Rp 500.000 - UNPAID
- Student 3: Rp 500.000 - UNPAID

MARCH 2025:
- Student 1: Rp 500.000 - UNPAID
- Student 2: Rp 500.000 - UNPAID
- Student 3: Rp 500.000 - UNPAID
```
| `amount` | → | `amount` |
| `collectDate` | → | `billingCollectDate` (calculated) |
| `dueDateOffset` | → | `billingDueDate` (calculated) |

---

## 🎯 Use Cases

### Use Case 1: SPP Tahunan
**Scenario:** Sekolah butuh generate SPP untuk 12 bulan penuh

```json
{
  "billingType": "MONTHLY",
  "name": "BIAYA SPP",
  "amount": 500000,
  "collectDate": 1,
  "dueDateOffset": 7,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-12-31",
  "monthlyActive": [1,2,3,4,5,6,7,8,9,10,11,12]
}
```
✅ **Result:** 12 Billing otomatis dibuat

---

### Use Case 2: SPP Skip Libur
**Scenario:** Sekolah libur Juli & Agustus, SPP hanya 10 bulan

```json
{
  "billingType": "MONTHLY",
  "name": "BIAYA SPP",
  "amount": 500000,
  "collectDate": 1,
  "dueDateOffset": 7,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": null,
  "monthlyActive": [1,2,3,4,5,6,9,10,11,12]
}
```
✅ **Result:** 10 Billing (skip bulan 7 & 8)

---

### Use Case 3: Biaya Buku Sekali Bayar
**Scenario:** Uang buku dibayar 1x di awal tahun ajaran

```json
{
  "billingType": "GENERAL",
  "name": "Uang Buku Pelajaran",
  "amount": 350000,
  "collectDate": 10,
  "dueDateOffset": 14,
  "startDatePeriod": "2025-07-01"
}
```
✅ **Result:** 1 Billing dengan due date 2025-07-15

---

### Use Case 4: Biaya Semester
**Scenario:** Split pembayaran per semester (6 bulan)

**Semester 1:**
```json
{
  "billingType": "MONTHLY",
  "name": "SPP Semester Ganjil",
  "amount": 500000,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-06-30",
  "monthlyActive": [1,2,3,4,5,6]
}
```

**Semester 2:**
```json
{
  "billingType": "MONTHLY",
  "name": "SPP Semester Genap",
  "amount": 500000,
  "startDatePeriod": "2025-07-01",
  "endDatePeriod": "2025-12-31",
  "monthlyActive": [7,8,9,10,11,12]
}
```
✅ **Result:** 6 Billing untuk masing-masing semester

---

## 🧪 Testing

### Test 1: Create GENERAL Billing
```http
POST /api/m-billings
{
  "billingType": "GENERAL",
  "name": "Test General",
  "amount": 100000,
  "startDatePeriod": "2025-01-15"
}
```

**Expected:**
- 1 Billing created
- `billingCollectDate`: 2025-01-15
- `name`: "Test General"

---

### Test 2: Create MONTHLY Billing (3 Months)
```http
POST /api/m-billings
{
  "billingType": "MONTHLY",
  "name": "Test Monthly",
  "amount": 50000,
  "collectDate": 20,
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-03-31",
  "monthlyActive": [1,2,3]
}
```

**Expected:**
- 3 Billings created:
  - Test Monthly - JANUARY 2025 (2025-01-20)
  - Test Monthly - FEBRUARY 2025 (2025-02-20)
  - Test Monthly - MARCH 2025 (2025-03-20)

---

## 📝 Notes

1. **Automatic Generation:** Billing **dan UserBilling** di-generate **langsung saat MBillings dibuat**, tidak perlu trigger manual
2. **Period Validation:** `startDatePeriod` > `endDatePeriod` tidak di-validasi, pastikan input benar
3. **Month Adjustment:** Jika `collectDate` > hari terakhir bulan, otomatis adjust ke hari terakhir
4. **Naming Convention:** 
   - GENERAL: pakai nama original
   - MONTHLY: append "- {MONTH} {YEAR}"
5. **Default Values:**
   - `startDatePeriod` null → pakai hari ini
   - `endDatePeriod` null (MONTHLY) → startDate + 1 tahun
   - `dueDateOffset` null/0 → due date = collect date
6. **UserBilling Filter:** Hanya BilledUsers dengan `deletedAt = null` yang dibuatkan UserBilling
7. **Payment Status:** Semua UserBilling dibuat dengan status `UNPAID` dan `paidAmount = 0`

---

## 🔧 Implementation Details

**Service Methods:** 
- `autoGenerateBillings()` - Line 712
- `autoGenerateUserBillings()` - Line 835

**Location:** `MBillingsServiceImpl.java`

**Invoked:** Setelah `mBillingsRepository.save()` di method `create()`

**Dependencies:**
- `BillingRepository` - untuk save Billing
- `UserBillingRepository` - untuk save UserBilling
- `BilledUsersRepository` - untuk get list BilledUsers
- `MBillingMonthlyActiveRepository` - untuk get active months

**Flow:**
1. Create MBillings → Save
2. Auto-generate Billings (GENERAL/MONTHLY)
3. Untuk setiap Billing → Auto-generate UserBillings
4. Return response lengkap

---

## ✅ Checklist Integration

- [x] MBillingsRequest: Add `startDatePeriod` & `endDatePeriod`
- [x] MBillings Entity: Support period fields
- [x] Service: Implement `autoGenerateBillings()` method
- [x] Service: Implement `autoGenerateUserBillings()` method
- [x] Service: Call auto-generate after create
- [x] UserBilling: Auto-generate untuk setiap BilledUser
- [x] HTTP File: Update examples dengan period
- [x] Documentation: Create comprehensive guide
- [x] Build: Maven compile successful

---

**Last Updated:** 2025-11-25  
**Version:** 2.0 (with UserBilling auto-generation)
