# 🔍 Addon Payment Flow - Debug Guide

## 📋 Problem Statement

**Issue:** Data masuk ke `TransactionAddon` namun tidak masuk ke `AddonInstalled` setelah payment sukses.

**Response yang diterima:**
```json
{
    "success": true,
    "message": "OK",
    "data": {
        "installed": 0,  // ❌ Harusnya > 0
        "institutionId": 1,
        "message": "Addon payment processed",
        "referenceId": "TRX-34-349",
        "transactionId": 34,
        "status": "PAID"
    }
}
```

---

## 🔄 Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. POST /api/transaction-business/buyaddons/open               │
│    ├─ Controller: TransactionBusinessController.buyaddonsOpen() │
│    └─ Service: TransactionBusinessServiceImpl.buyaddonsOpen()   │
│                                                                 │
│       ↓ calls buyAddons()                                      │
│                                                                 │
│    ┌────────────────────────────────────────────┐             │
│    │ buyAddons() Steps:                         │             │
│    │ 1. Validate Institution & PaymentMethod    │             │
│    │ 2. Fetch selected Addons                  │             │
│    │ 3. Calculate total price + tax            │             │
│    │ 4. Create TransactionBusiness (UNPAID)    │  ← Saved    │
│    │ 5. Create TransactionAddon records        │  ← Saved    │
│    │ 6. Generate reference: TRX-{id}-{random}  │             │
│    │ 7. Create PaymentGatewayLog via Ipaymu   │  ← Saved    │
│    │ 8. Update TransactionBusiness.reference   │  ← Updated  │
│    └────────────────────────────────────────────┘             │
│                                                                 │
│    Response: Payment gateway info (payment code, etc.)         │
└─────────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. User melakukan pembayaran via gateway (Ipaymu)              │
└─────────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│ 3. POST /api/ipaymu/notify/success/addons                      │
│    (Webhook dari Ipaymu)                                        │
│                                                                 │
│    ┌────────────────────────────────────────────┐             │
│    │ successByAddons() Steps:                   │             │
│    │ 1. Validate statusCode = 1 (success)      │             │
│    │ 2. Update PaymentGatewayLog                │  ← Updated  │
│    │ 3. Find TransactionBusiness by reference   │             │
│    │ 4. Mark TransactionBusiness as PAID        │  ← Updated  │
│    │ 5. Find TransactionAddon by transaction_id │  ← Query    │
│    │ 6. Install addons to AddonInstalled        │  ← Insert   │
│    │    ├─ Check if already installed           │             │
│    │    ├─ Create AddonInstalled record         │             │
│    │    └─ Increment installedCount             │             │
│    └────────────────────────────────────────────┘             │
│                                                                 │
│    Response: { installed: N, status: "PAID", ... }             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Tables

### 1. `transaction_business`
```sql
CREATE TABLE transaction_business (
    id INT PRIMARY KEY AUTO_INCREMENT,
    reference_number VARCHAR(50),  -- "TRX-34-349"
    status VARCHAR(20),            -- "UNPAID" → "PAID"
    institution_id INT,
    yayasan_id INT,
    ...
);
```

### 2. `transaction_addons`
```sql
CREATE TABLE transaction_addons (
    id INT PRIMARY KEY AUTO_INCREMENT,
    transaction_id INT,  -- FK to transaction_business.id
    addon_id INT,        -- FK to addons.id
    yayasan_id INT,
    institution_id INT,
    ...
);
```

### 3. `addon_installed`
```sql
CREATE TABLE addon_installed (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    addons_id BIGINT,           -- The addon that was installed
    institution_id INT,         -- Where it's installed
    yayasan_id INT,
    install_at TIMESTAMP,
    status VARCHAR(20),         -- "ENABLED"
    user_instaling_id INT,
    ...
);
```

### 4. `payment_gateway_log`
```sql
CREATE TABLE payment_gateway_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    reference_number VARCHAR(50),  -- Same as transaction_business.reference_number
    gateway_status VARCHAR(20),    -- "PENDING" → "SUCCESS"
    ...
);
```

---

## 🐛 Known Issues & Solutions

### Issue 1: `installed: 0` - TransactionAddon tidak ditemukan

**Root Cause:**
- Query `findByTransaction_Id()` tidak return TransactionAddon
- Kemungkinan karena lazy loading atau transaction_id tidak match

**Solution Applied:**
1. ✅ Menambahkan JOIN FETCH di `TransactionAddonRepository`:
```java
@Query("SELECT ta FROM TransactionAddon ta " +
       "LEFT JOIN FETCH ta.addon " +
       "LEFT JOIN FETCH ta.transaction " +
       "WHERE ta.transaction.id = :transactionId")
List<TransactionAddon> findByTransaction_Id(@Param("transactionId") Integer transactionId);
```

2. ✅ Menambahkan detailed logging di `successByAddons()`:
   - Log setiap step
   - Log semua TransactionAddon yang ditemukan
   - Log error yang terjadi dengan stacktrace

---

## 📊 Debugging Checklist

### Step 1: Verify TransactionBusiness Created
```sql
SELECT * FROM transaction_business 
WHERE reference_number = 'TRX-34-349';

-- Check:
-- ✓ id = 34
-- ✓ status = 'PAID'
-- ✓ institution_id not null
-- ✓ yayasan_id not null
```

### Step 2: Verify TransactionAddon Created
```sql
SELECT ta.*, a.name as addon_name
FROM transaction_addons ta
LEFT JOIN addons a ON ta.addon_id = a.id
WHERE ta.transaction_id = 34;

-- Expected: At least 1 row
-- Check:
-- ✓ addon_id not null
-- ✓ addon exists in addons table
-- ✓ institution_id matches transaction_business
```

### Step 3: Check Application Logs
```bash
# Look for these logs after payment success:
grep "[ADDONS_SUCCESS]" application.log

# Expected log sequence:
[ADDONS_SUCCESS] Received addon payment notification: referenceId=TRX-34-349, statusCode=1
[ADDONS_SUCCESS] Payment gateway log created/updated
[ADDONS_SUCCESS] Transaction resolved from log: id=34, status=PAID
[ADDONS_SUCCESS] Starting addon installation for transactionId=34
[ADDONS_SUCCESS] Found 2 transaction addons for transactionId=34
[ADDONS_SUCCESS] TransactionAddon[0]: id=45, transaction.id=34, addon.id=1, addon.name=WA Notif
[ADDONS_SUCCESS] Processing addon: addonId=1, addonName=WA Notif, institutionId=1
[ADDONS_SUCCESS] Creating AddonInstalled: addonId=1, institutionId=1, yayasanId=1
[ADDONS_SUCCESS] ✅ Successfully installed addon: id=123, addonId=1, institutionId=1
[ADDONS_SUCCESS] ✅ Addon installation completed. Total installed: 1 out of 2 addons
```

### Step 4: Verify AddonInstalled Created
```sql
SELECT * FROM addon_installed 
WHERE institution_id = 1 
ORDER BY created_at DESC 
LIMIT 10;

-- Check:
-- ✓ Record exists with addon_id that was purchased
-- ✓ status = 'ENABLED'
-- ✓ install_at is recent
```

---

## 🔧 Common Issues & Fixes

### Problem: "No transaction addons found"

**Possible Causes:**
1. TransactionAddon tidak disimpan saat `buyAddons()`
2. Transaction ID mismatch
3. Database transaction rollback

**Debug:**
```sql
-- Check if TransactionAddon was saved
SELECT COUNT(*) FROM transaction_addons WHERE transaction_id = 34;

-- If 0, check buyAddons() method in TransactionBusinessServiceImpl
-- Line 450-460: transactionAddonRepository.saveAll(taList)
```

**Fix:**
- Pastikan `transactionAddonRepository.saveAll()` dipanggil
- Pastikan tidak ada exception yang menyebabkan rollback
- Add `@Transactional(propagation = Propagation.REQUIRES_NEW)` jika perlu

---

### Problem: "TransactionAddon has null addon reference"

**Possible Causes:**
1. Lazy loading issue
2. Addon sudah dihapus dari database

**Fix:**
- ✅ Already fixed dengan JOIN FETCH di repository query
- Verifikasi addon masih ada: `SELECT * FROM addons WHERE id = ?`

---

### Problem: "Institution is NULL"

**Possible Causes:**
1. Institution dihapus setelah transaksi dibuat
2. Lazy loading issue

**Fix:**
```sql
-- Verify institution exists
SELECT * FROM institution WHERE id = 1;
```

---

## 🧪 Testing Steps

### 1. Manual Test via HTTP Client
```http
### Step 1: Login
POST http://localhost:8081/api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password"
}

### Step 2: Buy Addons
POST http://localhost:8081/api/transaction-business/buyaddons/open
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "channelId": 1,
  "addons": [1, 2],
  "paymentType": "va",
  "serviceId": 1,
  "taxPercent": 11
}

### Step 3: Simulate Payment Success (Dev Only)
POST http://localhost:8081/api/ipaymu/notify/success/addons
Content-Type: application/json

{
  "referenceId": "TRX-34-349",
  "statusCode": 1
}
```

### 2. Verify Database
```sql
-- After Step 2
SELECT * FROM transaction_business WHERE id = 34;
SELECT * FROM transaction_addons WHERE transaction_id = 34;

-- After Step 3
SELECT * FROM addon_installed WHERE institution_id = 1;
```

---

## 📝 Enhancement Applied

### Enhanced Logging in `successByAddons()`

1. **Before Processing:**
   - Log received payload
   - Log transaction details (id, institution, yayasan, status)

2. **During TransactionAddon Query:**
   - Log count of TransactionAddon found
   - Log each TransactionAddon detail (id, addon.id, addon.name)
   - Critical error if empty

3. **During Installation:**
   - Log each addon being processed
   - Log existence check result
   - Log AddonInstalled creation with all fields
   - Success confirmation with saved ID

4. **After Processing:**
   - Summary: total installed out of total addons

### Error Messages with Icons:
- ✅ Success
- ⚠️  Warning (idempotent skip)
- ❌ Critical error

---

## 🚀 Next Steps

1. **Run the application** with enhanced logging
2. **Test payment flow** via Postman/HTTP client
3. **Check logs** untuk melihat exactly di mana proses berhenti
4. **Verify database** setelah setiap step
5. **Report findings** berdasarkan log output

---

## 📞 Support

If issue persists after applying fixes:
1. Share complete logs from `[ADDONS_SUCCESS]`
2. Share SQL query results for transaction_id
3. Check if there's any exception in full application logs
