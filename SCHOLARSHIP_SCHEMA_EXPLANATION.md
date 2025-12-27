# Skema Beasiswa (Scholarship Schema)

## Overview
Sistem beasiswa yang fleksibel dengan dukungan untuk billing monthly dan general, termasuk pengaturan bulan-bulan tertentu yang mendapat beasiswa.

## Entity Relationship Diagram

```
Scholarship (Master Beasiswa)
    ↓ (1:M)
BillingScholarship (Relasi Scholarship-MBilling)
    ↓ (M:1)                     ↓ (1:M - hanya jika MBilling.billingType = MONTHLY)
MBillings                  MBillingScholarshipMonthly
    ↓ (1:M)                     
MBillingMonthlyActive
```

## Entities

### 1. Scholarship (Master Beasiswa)
**Table:** `scholarships`

**Purpose:** Master data beasiswa yang dapat diberikan kepada siswa/user

**Fields:**
- `id` (PK)
- `uuid` (unique)
- `yayasan_id` (FK → Yayasan)
- `institution_id` (FK → Institution, nullable)
- `name` (varchar 200) - Nama beasiswa (contoh: "Beasiswa Prestasi 50%")
- `description` (varchar 1000) - Deskripsi beasiswa
- `discount_type` (enum: PERCENTAGE/FIXED_AMOUNT) - Tipe diskon
- `discount_value` (decimal 15,2) - Nilai diskon (% atau nominal)
- `max_discount_amount` (decimal 15,2, nullable) - Batas maksimal diskon
- `is_active` (boolean)
- `notes` (varchar 2000)
- `created_at`, `updated_at`, `deleted_at`

**Example Data:**
```json
{
  "name": "Beasiswa Prestasi 50%",
  "discount_type": "PERCENTAGE",
  "discount_value": 50.00,
  "max_discount_amount": 5000000.00
}
```

### 2. BillingScholarship (Relasi Scholarship - MBilling)
**Table:** `billing_scholarships`

**Purpose:** Menghubungkan beasiswa dengan master billing tertentu

**Fields:**
- `id` (PK)
- `uuid` (unique)
- `scholarship_id` (FK → Scholarship) - Beasiswa yang digunakan
- `m_billing_id` (FK → MBillings) - Master billing target
- `is_active` (boolean)
- `notes` (varchar 2000)
- `created_at`, `updated_at`, `deleted_at`

**Unique Constraint:** `(scholarship_id, m_billing_id)` - Satu scholarship tidak boleh duplikat untuk MBilling yang sama

**Business Logic:**
- Cek `MBillings.billingType`:
  - Jika `MONTHLY` → harus ada relasi ke `MBillingScholarshipMonthly`
  - Jika `GENERAL` → beasiswa berlaku untuk semua billing

### 3. MBillingScholarshipMonthly (Bulan-bulan Beasiswa)
**Table:** `m_billing_scholarship_monthly`

**Purpose:** Menentukan bulan-bulan mana saja yang mendapat beasiswa (khusus billing MONTHLY)

**Fields:**
- `id` (PK)
- `uuid` (unique)
- `billing_scholarship_id` (FK → BillingScholarship)
- `month` (integer 1-12) - Bulan yang mendapat beasiswa
- `created_at`, `updated_at`

**Unique Constraint:** `(billing_scholarship_id, month)` - Tidak boleh duplikat bulan untuk BillingScholarship yang sama

**ATURAN VALIDASI PENTING:**

#### Rule 1: Subset dari MBillingMonthlyActive
Bulan yang diset di `MBillingScholarshipMonthly` **HARUS** merupakan subset dari `MBillingMonthlyActive`.

**Contoh Valid:**
```
MBillingMonthlyActive: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
MBillingScholarshipMonthly bisa:
  - [1, 2, 3, 4, 5] ✓
  - [6, 7, 8, 9, 10, 11, 12] ✓
  - [1, 3, 5, 7, 9, 11] ✓
  - [2, 4, 6] ✓
  - [1] ✓
  - [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12] ✓ (semua bulan)
```

**Contoh Tidak Valid:**
```
MBillingMonthlyActive: [1, 2, 3, 4, 5]
MBillingScholarshipMonthly:
  - [1, 2, 3, 4, 5, 6] ✗ (bulan 6 tidak ada di MBillingMonthlyActive)
  - [10, 11, 12] ✗ (semua bulan tidak ada di MBillingMonthlyActive)
```

#### Rule 2: Validasi Range Bulan
Month harus antara 1-12 (divalidasi di `@PrePersist` dan `@PreUpdate`)

## Use Cases

### Use Case 1: Beasiswa untuk Billing General
**Scenario:** Beasiswa 30% untuk SPP Tahunan

```sql
-- MBillings
billing_type = 'GENERAL'
amount = 10000000

-- BillingScholarship dibuat
-- MBillingScholarshipMonthly TIDAK DIBUAT (karena general)

-- Hasil: Diskon 30% berlaku untuk seluruh billing
```

### Use Case 2: Beasiswa untuk Billing Monthly - Semua Bulan
**Scenario:** Beasiswa 50% untuk SPP Bulanan selama 1 tahun penuh

```sql
-- MBillingMonthlyActive: [1,2,3,4,5,6,7,8,9,10,11,12]

-- MBillingScholarshipMonthly dibuat untuk bulan:
[1,2,3,4,5,6,7,8,9,10,11,12]

-- Hasil: Diskon 50% berlaku untuk 12 bulan
```

### Use Case 3: Beasiswa untuk Billing Monthly - Sebagian Bulan
**Scenario:** Beasiswa 100% untuk semester 1 saja (bulan 1-6)

```sql
-- MBillingMonthlyActive: [1,2,3,4,5,6,7,8,9,10,11,12]

-- MBillingScholarshipMonthly dibuat untuk bulan:
[1,2,3,4,5,6]

-- Hasil: 
-- - Bulan 1-6: Diskon 100% (GRATIS)
-- - Bulan 7-12: Tidak ada diskon (bayar normal)
```

### Use Case 4: Beasiswa untuk Bulan Tertentu
**Scenario:** Beasiswa 25% hanya untuk bulan ganjil

```sql
-- MBillingMonthlyActive: [1,2,3,4,5,6,7,8,9,10,11,12]

-- MBillingScholarshipMonthly dibuat untuk bulan:
[1,3,5,7,9,11]

-- Hasil:
-- - Bulan ganjil: Diskon 25%
-- - Bulan genap: Tidak ada diskon
```

## Business Logic Flow

### Ketika Membuat BillingScholarship:

```java
if (mBillings.getBillingType() == BillingTypeEnum.MONTHLY) {
    // 1. Ambil semua bulan aktif dari MBillingMonthlyActive
    List<Integer> activeMonths = mBillingMonthlyActive.getMonths();
    
    // 2. Validasi input bulan scholarship
    List<Integer> scholarshipMonths = request.getMonths();
    
    for (Integer month : scholarshipMonths) {
        if (!activeMonths.contains(month)) {
            throw new ValidationException(
                "Bulan " + month + " tidak ada di MBillingMonthlyActive. " +
                "Bulan yang tersedia: " + activeMonths
            );
        }
    }
    
    // 3. Create MBillingScholarshipMonthly untuk setiap bulan
    for (Integer month : scholarshipMonths) {
        MBillingScholarshipMonthly monthlyScholarship = new MBillingScholarshipMonthly();
        monthlyScholarship.setBillingScholarship(billingScholarship);
        monthlyScholarship.setMonth(month);
        save(monthlyScholarship);
    }
} else {
    // GENERAL billing - tidak perlu MBillingScholarshipMonthly
    // Beasiswa berlaku untuk seluruh billing
}
```

### Ketika Menghitung Diskon UserBilling:

```java
// Cek apakah user punya beasiswa untuk billing ini
BillingScholarship scholarship = findScholarshipForUserAndBilling(userId, billingId);

if (scholarship != null) {
    if (mBilling.getBillingType() == BillingTypeEnum.MONTHLY) {
        // Cek apakah bulan ini ada di scholarship months
        Integer currentMonth = billing.getMonth();
        boolean hasScholarshipThisMonth = scholarshipMonthlyRepository
            .existsByBillingScholarshipIdAndMonth(scholarship.getId(), currentMonth);
        
        if (hasScholarshipThisMonth) {
            // Apply discount
            BigDecimal discount = calculateDiscount(
                billing.getAmount(),
                scholarship.getScholarship().getDiscountType(),
                scholarship.getScholarship().getDiscountValue(),
                scholarship.getScholarship().getMaxDiscountAmount()
            );
            
            userBilling.setDiscountValue(discount);
        }
    } else {
        // GENERAL billing - langsung apply discount
        BigDecimal discount = calculateDiscount(...);
        userBilling.setDiscountValue(discount);
    }
}
```

## Database Indexes

Untuk performa optimal:

```sql
-- Scholarship
CREATE INDEX idx_scholarship_yayasan ON scholarships(yayasan_id);
CREATE INDEX idx_scholarship_institution ON scholarships(institution_id);
CREATE INDEX idx_scholarship_active ON scholarships(is_active);

-- BillingScholarship
CREATE INDEX idx_billing_scholarship_scholarship ON billing_scholarships(scholarship_id);
CREATE INDEX idx_billing_scholarship_mbilling ON billing_scholarships(m_billing_id);
CREATE INDEX idx_billing_scholarship_active ON billing_scholarships(is_active);

-- MBillingScholarshipMonthly
CREATE INDEX idx_billing_scholarship_monthly_bs ON m_billing_scholarship_monthly(billing_scholarship_id);
CREATE INDEX idx_billing_scholarship_monthly_month ON m_billing_scholarship_monthly(month);
```

## Migration Considerations

1. **Foreign Key Constraints:**
   - Pastikan MBillings sudah ada sebelum membuat BillingScholarship
   - Cascade delete untuk MBillingScholarshipMonthly ketika BillingScholarship dihapus

2. **Data Integrity:**
   - Validasi bulan di application level sebelum insert
   - Constraint di database: `month BETWEEN 1 AND 12`
   - Unique constraint untuk mencegah duplikasi

3. **Soft Delete:**
   - Gunakan `deleted_at` untuk soft delete
   - Query selalu filter `deleted_at IS NULL`

## Next Steps - Implementation

1. ✅ Create entities (DONE)
2. Create repositories
3. Create DTOs (Request/Response)
4. Create services dengan business logic validation
5. Create controllers untuk CRUD operations
6. Create migration files
7. Write unit tests untuk validasi rules
8. Create documentation untuk API endpoints
