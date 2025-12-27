# MBillings Schema Documentation

## Overview
`MBillings` adalah master billing yang mendukung 2 tipe tagihan: **MONTHLY** dan **GENERAL**.

## Diagram Relasi

```
┌─────────────────────────────────────────────┐
│            MBillings                         │
│  ┌────────────────────────────────────────┐ │
│  │ id: Long                               │ │
│  │ uuid: String                           │ │
│  │ yayasan_id: FK → Yayasan               │ │
│  │ institution_id: FK → Institution       │ │
│  │ billingType: Enum (MONTHLY/GENERAL) ★  │ │
│  │ name: String                           │ │
│  │ description: String                    │ │
│  │ amount: BigDecimal                     │ │
│  │ collectDate: Integer (1-31)            │ │
│  │ dueDateOffset: Integer                 │ │
│  │ isAutoGenerate: Boolean                │ │
│  │ isActive: Boolean                      │ │
│  │ createdAt: LocalDateTime               │ │
│  │ updatedAt: LocalDateTime               │ │
│  │ deletedAt: LocalDateTime               │ │
│  └────────────────────────────────────────┘ │
└──────────────┬──────────────────────────────┘
               │
               │ Relasi HANYA jika billingType = MONTHLY
               │ (OneToMany)
               ▼
┌──────────────────────────────────────────────┐
│      MBillingMonthlyActive                   │
│  ┌────────────────────────────────────────┐  │
│  │ id: Long                               │  │
│  │ uuid: String                           │  │
│  │ m_billing_id: FK → MBillings   │  │
│  │ month: Integer (1-12)                  │  │
│  │   1 = Januari, 2 = Februari, dst       │  │
│  │ createdAt: LocalDateTime               │  │
│  │ updatedAt: LocalDateTime               │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Unique Constraint:                          │
│  (m_billing_id, month)               │
└──────────────────────────────────────────────┘
```

## Tipe Billing

### 1. MONTHLY (Tagihan Bulanan Berulang)

**Karakteristik:**
- ✅ Untuk tagihan yang berulang setiap bulan (SPP, Uang Kegiatan Bulanan, dll)
- ✅ **WAJIB** memiliki `monthlyActive` (array bulan 1-12)
- ✅ **WAJIB** berelasi dengan tabel `m_billing_monthly_active`
- ✅ Digunakan sebagai acuan untuk generate tagihan bulanan ke tabel `billings`
- ✅ Field `collectDate` dan `dueDateOffset` digunakan untuk menentukan tanggal tagihan

**Contoh Use Case:**
- SPP Bulanan (12 bulan)
- Uang Kegiatan Ekstrakurikuler (10 bulan, skip libur)
- Biaya Semester (6 bulan per semester)

**Contoh Request:**
```json
{
  "billingType": "MONTHLY",
  "name": "BIAYA SPP",
  "description": "Biaya SPP per bulan",
  "amount": 500000,
  "collectDate": 1,
  "dueDateOffset": 7,
  "isAutoGenerate": true,
  "monthlyActive": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
}
```

**Data di `m_billing_monthly_active`:**
```
┌────┬─────────────────────┬───────┐
│ id │ m_billing_id│ month │
├────┼─────────────────────┼───────┤
│ 1  │ 1 (BIAYA SPP)       │   1   │ → Januari
│ 2  │ 1 (BIAYA SPP)       │   2   │ → Februari
│ 3  │ 1 (BIAYA SPP)       │   3   │ → Maret
│... │ ...                 │  ...  │
│ 12 │ 1 (BIAYA SPP)       │  12   │ → Desember
└────┴─────────────────────┴───────┘
```

---

### 2. GENERAL (Tagihan Parsial Sekali Bayar)

**Karakteristik:**
- ✅ Untuk tagihan yang **TIDAK berulang** / sekali bayar (Uang Buku, Seragam, Gedung, dll)
- ❌ **TIDAK BOLEH** memiliki `monthlyActive`
- ❌ **TIDAK** berelasi dengan tabel `m_billing_monthly_active`
- ✅ Biasanya untuk tagihan tahunan atau parsial
- ⚠️ Field `collectDate` dan `dueDateOffset` opsional (tidak digunakan untuk generate bulanan)

**Contoh Use Case:**
- Uang Buku Pelajaran (1x per tahun ajaran)
- Uang Seragam (1x saat masuk sekolah)
- Uang Gedung (1x saat pendaftaran)
- Biaya Study Tour (1x per event)

**Contoh Request:**
```json
{
  "billingType": "GENERAL",
  "name": "Uang Buku Pelajaran",
  "description": "Biaya buku pelajaran tahun ajaran 2025/2026",
  "amount": 350000,
  "isAutoGenerate": false
}
```

**Data di `m_billing_monthly_active`:**
```
KOSONG - Tidak ada data karena bukan tagihan bulanan
```

---

## Validasi Business Rules

### Create MBillings

| Field | MONTHLY | GENERAL | Validasi |
|-------|---------|---------|----------|
| `billingType` | ✅ Wajib | ✅ Wajib | Harus MONTHLY atau GENERAL |
| `name` | ✅ Wajib | ✅ Wajib | Tidak boleh kosong |
| `amount` | ✅ Wajib | ✅ Wajib | > 0 |
| `monthlyActive` | ✅ **WAJIB** (array 1-12) | ❌ **TIDAK BOLEH ADA** | Jika MONTHLY: wajib dan 1-12; Jika GENERAL: harus null/empty |
| `collectDate` | ✅ Opsional | ⚠️ Tidak digunakan | 1-31 |
| `dueDateOffset` | ✅ Opsional | ⚠️ Tidak digunakan | >= 0 |
| `isAutoGenerate` | ✅ Wajib | ✅ Wajib | true/false |

### Contoh Validasi Error

**Error 1: MONTHLY tanpa monthlyActive**
```json
{
  "billingType": "MONTHLY",
  "name": "SPP",
  "amount": 500000,
  "isAutoGenerate": true
  // monthlyActive TIDAK ADA ❌
}
```
**Response:**
```json
{
  "success": false,
  "message": "Untuk billing MONTHLY, bulan aktif harus diisi"
}
```

**Error 2: GENERAL dengan monthlyActive**
```json
{
  "billingType": "GENERAL",
  "name": "Uang Buku",
  "amount": 350000,
  "isAutoGenerate": false,
  "monthlyActive": [1, 2, 3] // ❌ TIDAK BOLEH ADA
}
```
**Response:**
```json
{
  "success": false,
  "message": "Untuk billing GENERAL, tidak boleh ada bulan aktif (ini bukan tagihan bulanan)"
}
```

---

## Response Structure

### MONTHLY Response
```json
{
  "id": 1,
  "uuid": "550e8400-...",
  "billingType": "MONTHLY",
  "name": "BIAYA SPP",
  "amount": 500000,
  "monthlyActive": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
  "collectDate": 1,
  "dueDateOffset": 7,
  "isAutoGenerate": true,
  "isActive": true,
  "createdAt": "2025-11-24T10:00:00",
  "updatedAt": "2025-11-24T10:00:00"
}
```

### GENERAL Response
```json
{
  "id": 2,
  "uuid": "660e8400-...",
  "billingType": "GENERAL",
  "name": "Uang Buku Pelajaran",
  "amount": 350000,
  "monthlyActive": null,  // ← SELALU NULL untuk GENERAL
  "collectDate": null,
  "dueDateOffset": null,
  "isAutoGenerate": false,
  "isActive": true,
  "createdAt": "2025-11-24T10:00:00",
  "updatedAt": "2025-11-24T10:00:00"
}
```

---

## Migration Database

Jika Anda upgrade dari versi lama, pastikan:

1. ✅ Tambahkan kolom `billing_type` di tabel `m_billings`
```sql
ALTER TABLE m_billings 
ADD COLUMN billing_type VARCHAR(50) NOT NULL DEFAULT 'MONTHLY';
```

2. ✅ Update existing data (semua data lama dianggap MONTHLY)
```sql
UPDATE m_billings SET billing_type = 'MONTHLY' WHERE billing_type IS NULL;
```

3. ✅ Pastikan foreign key constraint sudah benar
```sql
ALTER TABLE m_billing_monthly_active
ADD CONSTRAINT fk_mbilling_monthly_active 
FOREIGN KEY (m_billing_id) REFERENCES m_billings(id);
```

---

## Best Practices

### ✅ DO (Lakukan)
- Gunakan **MONTHLY** untuk tagihan yang berulang setiap bulan
- Gunakan **GENERAL** untuk tagihan sekali bayar
- Set `isAutoGenerate = true` untuk MONTHLY agar bisa auto-generate ke tabel `billings`
- Set `isAutoGenerate = false` untuk GENERAL karena biasanya dibuat manual

### ❌ DON'T (Jangan)
- Jangan buat GENERAL dengan `monthlyActive` (akan ditolak sistem)
- Jangan buat MONTHLY tanpa `monthlyActive` (akan ditolak sistem)
- Jangan lupa set `billingType` (wajib diisi)

---

## Alur Generate Billing Bulanan

```
┌─────────────────────┐
│ MBillings (MONTHLY) │
│ + monthlyActive     │
└──────────┬──────────┘
           │
           │ Process Generate
           │ (via Scheduler/Manual)
           ▼
┌──────────────────────┐
│ Cek monthlyActive:   │
│ - Bulan 1,2,3...12   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Generate Billing     │
│ untuk bulan aktif    │
│ ke tabel: billings   │
└──────────────────────┘
```

**Contoh:**
- Jika `monthlyActive = [1, 2, 3, 4, 5, 6, 9, 10, 11, 12]` (skip Juli & Agustus)
- Maka sistem hanya akan generate billing untuk 10 bulan tersebut
- Bulan 7 (Juli) dan 8 (Agustus) tidak akan di-generate

---

## Kesimpulan

| Aspek | MONTHLY | GENERAL |
|-------|---------|---------|
| **Tujuan** | Tagihan bulanan berulang | Tagihan sekali bayar |
| **Relasi m_billing_monthly_active** | ✅ Ada | ❌ Tidak ada |
| **monthlyActive** | ✅ Wajib | ❌ Tidak boleh |
| **Generate Otomatis** | ✅ Ya (via scheduler) | ❌ Tidak (manual) |
| **Contoh** | SPP, Uang Kegiatan | Buku, Seragam, Gedung |

---

**Last Updated:** 2025-11-24  
**Version:** 1.0
