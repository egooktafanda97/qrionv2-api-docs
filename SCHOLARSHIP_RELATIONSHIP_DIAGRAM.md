# Diagram Relasi Beasiswa (Scholarship Relationship)

## Overview
Dokumen ini menjelaskan hubungan antara entity-entity yang terkait dengan sistem beasiswa (scholarship).

## Entity Relationship Diagram

```
┌─────────────────────┐
│   Scholarship       │  Master data program beasiswa
│   (m_scholarship)   │
└──────────┬──────────┘
           │
           │ 1:N (One scholarship can be applied to many billings)
           │
           ▼
┌─────────────────────────────┐
│  BillingScholarship         │  Mapping: scholarship → billing dengan diskon
│  (billing_scholarship)      │  - Menentukan billing mana yang dapat diskon
├─────────────────────────────┤  - Menentukan tipe diskon (PERCENTAGE/FIXED)
│ - scholarship_id (FK)       │  - Menentukan nilai diskon
│ - m_billing_id (FK)         │  - Dapat membatasi untuk bulan tertentu
│ - discountType              │
│ - discountValue             │
│ - maxDiscountAmount         │
└──────────┬──────────────────┘
           │
           │ 1:N (One billing scholarship can apply to many months)
           │
           ▼
┌──────────────────────────────────┐
│  MBillingScholarshipMonthly      │  Bulan-bulan yang mendapat diskon
│  (m_billing_scholarship_monthly) │  - Untuk MONTHLY billing type
├──────────────────────────────────┤  - Satu record per bulan
│ - m_billing_scholarship_id (FK)  │  - Contoh: [1,2,3] = Jan, Feb, Mar
│ - month (1-12)                   │
└──────────────────────────────────┘


┌─────────────────────┐       ┌─────────────────────┐
│   Students          │       │   Scholarship       │
│   (students)        │       │   (m_scholarship)   │
└──────────┬──────────┘       └──────────┬──────────┘
           │                             │
           │ N:1                     1:N │
           │                             │
           └──────────┬──────────────────┘
                      │
                      ▼
           ┌─────────────────────────────┐
           │  StudentScholarship         │  Siswa yang menerima beasiswa
           │  (student_scholarship)      │  - Menghubungkan siswa dengan scholarship
           ├─────────────────────────────┤  - System query BillingScholarship by scholarship_id
           │ - student_id (FK)           │    untuk dapat list billing yang dapat diskon
           │ - scholarship_id (FK)       │
           │ - awarded_date              │
           └─────────────────────────────┘
```

## Alur Kerja (Workflow)

### 1. Setup Program Beasiswa untuk Billing

**Admin membuat aturan beasiswa:**

```
1. Admin pilih Scholarship (misal: "Beasiswa Prestasi")
2. Admin pilih MBilling yang akan dapat diskon (misal: "SPP Bulanan")
3. Admin atur diskon:
   - discountType: PERCENTAGE / FIXED
   - discountValue: 50.00 (berarti 50% atau Rp 50.000)
   - maxDiscountAmount: 500000 (opsional, untuk PERCENTAGE)
4. Admin pilih bulan yang berlaku: [1, 2, 3, 4, 5, 6]
   → System create 6 records di m_billing_scholarship_monthly
```

**Database Result:**
```sql
-- billing_scholarship
id | scholarship_id | m_billing_id | discount_type | discount_value | max_discount_amount
1  | 1              | 1            | PERCENTAGE    | 50.00          | 500000

-- m_billing_scholarship_monthly
id | m_billing_scholarship_monthly_id | month
1  | 1                                 | 1
2  | 1                                 | 2
3  | 1                                 | 3
4  | 1                                 | 4
5  | 1                                 | 5
6  | 1                                 | 6
```

### 2. Memberikan Beasiswa ke Siswa

**Admin memberikan beasiswa ke siswa:**

```
1. Admin pilih Student (misal: "Budi")
2. Admin pilih Scholarship (misal: "Beasiswa Prestasi")
3. System create StudentScholarship record
4. System otomatis akan query semua BillingScholarship dengan scholarship_id=1
```

**Database Result:**
```sql
-- student_scholarship
id | student_id | scholarship_id | awarded_date
1  | 123        | 1              | 2025-01-15

-- System akan query:
-- SELECT * FROM billing_scholarship WHERE scholarship_id = 1
-- Hasilnya bisa dapat multiple billing discounts (SPP, Uang Bangunan, dll)
```

### 3. Perhitungan Diskon saat Generate User Billing

**System auto-generate user billing dengan diskon:**

```
1. System generate user_billing untuk siswa Budi (billing_id = 1, bulan = January)
2. System cek: apakah Budi punya StudentScholarship?
   → Yes: scholarship_id = 1
3. System query BillingScholarship:
   SELECT * FROM billing_scholarship 
   WHERE scholarship_id = 1 AND m_billing_id = 1
   → Found: billing_scholarship.id = 1
4. System ambil discount rules:
   - discountType = PERCENTAGE
   - discountValue = 50.00
   - months = [1, 2, 3, 4, 5, 6] (dari m_billing_scholarship_monthly)
5. System cek: apakah bulan ini (January = 1) termasuk dalam months?
   → Yes, month 1 ada di list
6. System apply diskon:
   - Original: Rp 1.000.000
   - Discount 50%: Rp 500.000
   - Final: Rp 500.000
```

## Keuntungan Arsitektur Ini

### ✅ Flexible
- Satu scholarship bisa diterapkan ke banyak billing (SPP, Uang Bangunan, dll)
- Setiap billing bisa punya aturan diskon berbeda
- Diskon bisa dibatasi untuk bulan tertentu (MONTHLY billing)

### ✅ Reusable
- BillingScholarship bisa digunakan untuk banyak siswa
- Tidak perlu create aturan diskon berulang-ulang

### ✅ Maintainable
- Update diskon cukup di BillingScholarship
- Semua siswa yang menggunakan scholarship itu akan otomatis dapat update

### ✅ Auditability
- Setiap siswa punya history scholarship yang diterima
- Tanggal pemberian beasiswa tercatat (awarded_date)

## Contoh Use Case

### Use Case 1: Beasiswa Penuh (100%)

```sql
-- BillingScholarship untuk beasiswa penuh SPP
scholarship_id: 2 (Beasiswa Yatim)
m_billing_id: 1 (SPP Bulanan)
discount_type: PERCENTAGE
discount_value: 100.00
months: [1,2,3,4,5,6,7,8,9,10,11,12] -- semua bulan
```

### Use Case 2: Diskon Tetap per Bulan

```sql
-- BillingScholarship untuk diskon fixed
scholarship_id: 3 (Beasiswa KIP)
m_billing_id: 1 (SPP Bulanan)
discount_type: FIXED
discount_value: 250000.00
months: [1,2,3,4,5,6] -- semester 1 saja
```

### Use Case 3: Diskon Persentase dengan Max

```sql
-- BillingScholarship untuk diskon persentase dengan batas
scholarship_id: 1 (Beasiswa Prestasi)
m_billing_id: 1 (SPP Bulanan)
discount_type: PERCENTAGE
discount_value: 50.00
max_discount_amount: 500000.00 -- maksimal diskon Rp 500.000
months: [1,2,3,4,5,6,7,8,9,10,11,12]
```

**Contoh Perhitungan:**
- SPP Rp 800.000 → Diskon 50% = Rp 400.000 ✅ (di bawah max)
- SPP Rp 1.500.000 → Diskon 50% = Rp 750.000 ❌ → Capped at Rp 500.000

## Migration Notes

### Perubahan dari Versi Sebelumnya

**REMOVED:**
```java
// StudentScholarship.java - DIHAPUS
@Column(name = "amount", precision = 15, scale = 2)
private BigDecimal amount;
```

**NOT ADDED (Purposely):**
```java
// StudentScholarship.java - TIDAK PERLU field ini
// @ManyToOne(optional = true)
// @JoinColumn(name = "billing_scholarship_id")
// private BillingScholarship billingScholarship;
```

**WHY?**
- `amount` field tidak flexible (fixed amount untuk semua billing)
- **Tidak perlu** `billingScholarship` reference karena:
  - 1 Scholarship bisa punya BANYAK BillingScholarship mappings
  - System akan **query dynamically** saat generate billing:
    ```sql
    SELECT * FROM billing_scholarship 
    WHERE scholarship_id = ? AND m_billing_id = ?
    ```
  - Lebih flexible: scholarship bisa diterapkan ke billing baru tanpa update StudentScholarship
  - Amount dihitung dynamic per billing type dan per bulan

### Database Migration

```sql
-- 1. Drop amount column (tidak diperlukan lagi)
ALTER TABLE student_scholarship 
DROP COLUMN IF EXISTS amount;

-- 2. TIDAK PERLU add billing_scholarship_id
-- Karena system akan query BillingScholarship by scholarship_id secara dynamic

-- 3. Pastikan sudah ada index untuk query performance
CREATE INDEX IF NOT EXISTS idx_billing_scholarship_scholarship_id 
ON billing_scholarship(scholarship_id);

CREATE INDEX IF NOT EXISTS idx_billing_scholarship_scholarship_billing 
ON billing_scholarship(scholarship_id, m_billing_id);
```

## API Endpoints

### BillingScholarship Management

```http
# Create BillingScholarship
POST /api/billing-scholarships
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 50.00,
  "maxDiscountAmount": 500000.00,
  "months": [1, 2, 3, 4, 5, 6],
  "notes": "Diskon 50% untuk semester 1"
}

# Update BillingScholarship
PUT /api/billing-scholarships/{id}
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 75.00,
  "months": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
}

# Get BillingScholarship by Scholarship
GET /api/billing-scholarships/scholarship/{scholarshipId}

# Get BillingScholarship by Billing
GET /api/billing-scholarships/billing/{mBillingId}
```

## Summary

**Entity Roles:**

1. **Scholarship**: Master data program beasiswa
2. **BillingScholarship**: Aturan diskon untuk billing tertentu (mapping scholarship → billing)
3. **MBillingScholarshipMonthly**: Bulan-bulan yang mendapat diskon
4. **StudentScholarship**: Siswa yang menerima beasiswa (hanya simpan scholarship_id)

**Key Relationships:**

- `Scholarship` → 1:N → `BillingScholarship` (satu beasiswa bisa untuk banyak billing)
- `BillingScholarship` → 1:N → `MBillingScholarshipMonthly` (satu aturan untuk banyak bulan)
- `StudentScholarship` → N:1 → `Scholarship` (banyak siswa bisa dapat scholarship yang sama)
- **Query Pattern**: System query `BillingScholarship` by `scholarship_id` untuk dapat discount rules

**Benefits:**

✅ Flexible discount rules per billing type  
✅ Month-specific discounts for MONTHLY billing  
✅ Reusable discount configurations  
✅ Easy to maintain and audit  
✅ Dynamic calculation based on rules  

---

*Last Updated: 2025-11-26*  
*Author: System Documentation*
