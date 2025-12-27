# Auto Generate User Billing

## Overview
**SUDAH TERINTEGRASI KE MBILLINGS - TIDAK PERLU ENDPOINT TERPISAH!**

Ketika membuat `MBillings`, sistem akan otomatis:
1. ✅ Generate `Billing` berdasarkan type (GENERAL/MONTHLY)
2. ✅ Generate `UserBilling` untuk setiap `BilledUser` yang aktif
3. ✅ Semua dilakukan dalam satu transaksi saat create MBillings

## Flow Otomatis

```
Create MBillings
    ↓
Auto Generate Billings (GENERAL/MONTHLY)
    ↓
Auto Generate UserBillings (untuk setiap BilledUser)
```

## Cara Penggunaan

Cukup gunakan endpoint MBillings yang sudah ada:

```http
POST http://localhost:8081/api/v1/mbillings
Authorization: Bearer {{accessToken}}
Content-Type: application-json

{
  "billingType": "MONTHLY",
  "name": "SPP Tahun Ajaran 2024/2025",
  "amount": 500000,
  "collectDate": 10,
  "dueDateOffset": 5,
  "isAutoGenerate": true,
  "isActive": true,
  "monthlyActive": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
  "userBilled": [
    "{{student1_uuid}}",
    "{{student2_uuid}}"
  ],
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-12-31"
}
```

## Hasil Otomatis

Setelah request di atas, sistem akan membuat:

### 1. Master Billing (MBillings)
- 1 record master billing dengan metadata

### 2. Billing Records  
- GENERAL: 1 billing
- MONTHLY: 12 billing (Januari - Desember 2025)

### 3. UserBilling Records
- Untuk **setiap** billing yang dibuat
- **Untuk setiap** user di `billedUsers`
- Contoh: 12 billing × 2 students = **24 UserBilling records**

### Detail UserBilling yang Dibuat

Setiap UserBilling memiliki:
- `billing` → FK ke Billing yang baru dibuat
- `billedUserRef` → FK ke BilledUsers
- `billedUser` → FK ke Users (student)
- `baseAmount` → Amount dari Billing
- `paidAmount` → 0 (default)
- `paymentStatus` → UNPAID (default)
- `yayasan`, `institution` → Inherited dari MBillings

## Filter BilledUser

Hanya BilledUsers yang aktif yang akan dibuatkan UserBilling:
```java
billedUser.deletedAt == null
```

## Contoh Lengkap

### Input:
```json
{
  "billingType": "MONTHLY",
  "name": "SPP",
  "amount": 500000,
  "monthlyActive": [1, 2, 3],
  "userBilled": ["student1", "student2"],
  "startDatePeriod": "2025-01-01",
  "endDatePeriod": "2025-03-31"
}
```

### Output Otomatis:

**MBillings**: 1 record
```
- SPP (master)
```

**Billings**: 3 records
```
- SPP - JANUARY 2025
- SPP - FEBRUARY 2025  
- SPP - MARCH 2025
```

**UserBillings**: 6 records (3 billings × 2 students)
```
- [JANUARY] Student 1 - Rp 500.000 - UNPAID
- [JANUARY] Student 2 - Rp 500.000 - UNPAID
- [FEBRUARY] Student 1 - Rp 500.000 - UNPAID
- [FEBRUARY] Student 2 - Rp 500.000 - UNPAID
- [MARCH] Student 1 - Rp 500.000 - UNPAID
- [MARCH] Student 2 - Rp 500.000 - UNPAID
```

## Implementasi

Lihat `MBillingsServiceImpl.java`:
- Method: `autoGenerateBillings()` - Line 712
- Method: `autoGenerateUserBillings()` - Line 835
- Dipanggil otomatis setelah `create()` - Line 150

## Catatan

- ⚠️ **TIDAK ADA endpoint terpisah** untuk generate UserBilling
- ✅ Semua otomatis saat create MBillings
- ✅ Transaksi terintegrasi dan konsisten
- ✅ Filter otomatis hanya BilledUser aktif
