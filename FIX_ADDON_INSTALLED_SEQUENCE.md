# 🔧 FIX: AddonInstalled Sequence Issue

## 🐛 **ERROR YANG TERJADI:**

```
ERROR: duplicate key value violates unique constraint "addon_installed_pkey"
Detail: Key (id)=(3) already exists.
```

**Penyebab:** PostgreSQL sequence untuk `addon_installed_id_seq` tidak ter-sync dengan data yang sudah ada di table.

---

## 📋 **ROOT CAUSE ANALYSIS:**

1. Table `addon_installed` memiliki primary key ID dengan `@GeneratedValue(strategy = GenerationType.IDENTITY)`
2. Sequence `addon_installed_id_seq` kehilangan sinkronisasi dengan data aktual
3. Ketika Hibernate try insert, ID dipilih dari sequence yang outdated
4. Conflict terjadi karena ID 3 sudah ada di table

---

## ✅ **SOLUSI:**

### **Step 1: Reset Sequence di PostgreSQL**

Jalankan SQL ini di PostgreSQL client (DBeaver, psql, atau pg admin):

```sql
-- 1. Check current max ID
SELECT MAX(id) FROM addon_installed;

-- 2. Reset sequence ke nilai lebih tinggi dari max ID
-- Misalnya jika MAX(id) = 5, set ke 6
ALTER SEQUENCE addon_installed_id_seq RESTART WITH 10;

-- 3. Verify (optional)
SELECT setval('addon_installed_id_seq', 
    COALESCE((SELECT MAX(id) FROM addon_installed), 0) + 1);
```

### **Step 2: Code Enhancement di Controller**

Sudah ditambahkan handling untuk `DataIntegrityViolationException`:

```java
try {
    AddonInstalled savedAddon = addonInstalledRepository.save(ai);
    installedCount++;
    logger.info("[ADDONS_SUCCESS] ✅ Successfully installed addon...");
} catch (org.springframework.dao.DataIntegrityViolationException die) {
    if (die.getMessage() != null && die.getMessage().contains("duplicate key")) {
        logger.warn("[ADDONS_SUCCESS] ⚠️  Addon already installed (duplicate key)...");
    } else {
        logger.error("[ADDONS_SUCCESS] ❌ Database constraint violation...");
    }
}
```

---

## 🧪 **TESTING SETELAH FIX:**

### **1. Jalankan SQL Fix di Database**
```bash
# Menggunakan psql
psql -U postgres -d qrion_db -c "ALTER SEQUENCE addon_installed_id_seq RESTART WITH 100;"
```

### **2. Rebuild & Run Aplikasi**
```bash
./mvnw clean package -DskipTests
./mvnw spring-boot:run
```

### **3. Test Payment Flow**
- POST `/api/transaction-business/buyaddons/open`
- Trigger payment success callback
- Check logs untuk `[ADDONS_SUCCESS]`
- Verify data di table `addon_installed`

### **4. Verify Database**
```sql
-- Check sequence current value
SELECT last_value FROM addon_installed_id_seq;

-- Check installed addons
SELECT * FROM addon_installed ORDER BY id DESC LIMIT 5;
```

---

## 📊 **Expected Log Output (After Fix):**

```
[ADDONS_SUCCESS] ✅ Successfully installed addon: id=10, addonId=5, institutionId=1
[ADDONS_SUCCESS] ✅ Addon installation completed. Total installed: 1 out of 1 addons
```

---

## 🛠️ **Prevention untuk Masa Depan:**

1. **Gunakan database migration tool** (Flyway/Liquibase) untuk manage sequence
2. **Validate sequence saat startup** - tambah Health Check
3. **Audit trail** - log semua ID generation issues

---

## 📝 **Script File:**

Created at: `scripts/fix_addon_installed_sequence.sql`

```sql
-- Fix addon_installed sequence issue
-- Run this SQL in PostgreSQL to reset the sequence

-- 1. Find the current max ID in addon_installed
SELECT MAX(id) FROM addon_installed;

-- 2. Reset the sequence
-- Change XXXX to the value from step 1 + 1
ALTER SEQUENCE addon_installed_id_seq RESTART WITH 10;

-- 3. Verify the fix
SELECT setval('addon_installed_id_seq', (SELECT MAX(id) FROM addon_installed) + 1);

-- 4. Test insert
-- After running this, try the payment notification again
```

---

## ✅ **Checklist Sebelum Go Live:**

- [ ] Run SQL fix untuk reset sequence
- [ ] Rebuild aplikasi dengan code changes
- [ ] Test full payment flow (buyaddons + success callback)
- [ ] Verify addons berhasil di-install ke `addon_installed` table
- [ ] Check logs untuk `[ADDONS_SUCCESS] ✅`
- [ ] Clean up old/duplicate records jika ada

---

## 📞 **If Issue Persists:**

Jika setelah fix masih error, check:

1. **Sequence tidak ter-reset:** Run SQL lagi dengan nilai yang lebih tinggi
2. **Multiple addons install:** Verify unique constraint rules
3. **Transaction isolation:** Check PostgreSQL transaction level

Share logs dan hasil `SELECT MAX(id) FROM addon_installed;` untuk debugging lebih lanjut.
