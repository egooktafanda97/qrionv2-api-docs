# Critical Business Logic Implementation - Billing System

## Overview
Implementasi business logic penting untuk data integrity dan audit trail pada sistem billing.

## Business Rules yang Diimplementasikan

### 1. Cascade Delete BilledUsers → UserBilling
**Service**: `BilledUsersService` & `BilledUsersServiceImpl`

**Rule**: Jika user di-remove dari tabel `BilledUsers` dengan `mBilling` tertentu, maka pada `UserBilling` wajib hilang juga.

**Implementation**:
- Method: `softDeleteBilledUser(Long billedUserId)`
- Method: `hardDeleteBilledUser(Long billedUserId)`
- Validation: `canDeleteBilledUser(Long billedUserId)`

**Protection**:
- ❌ Tidak bisa delete jika ada `UserBilling` dengan `lockedUpdate = true`
- ✅ Cascade soft/hard delete ke semua `UserBilling` terkait

```java
// Usage example
billedUsersService.softDeleteBilledUser(billedUserId);
```

---

### 2. Cascade Delete Users → BilledUsers & UserBilling
**Service**: `UsersCascadeService` & `UsersCascadeServiceImpl`

**Rule**: Jika `Users` dihapus, maka pada `BilledUsers` dan `UserBilling` wajib hilang juga.

**Implementation**:
- Method: `softDeleteUserWithCascade(Integer userId)`
- Validation: `canDeleteUser(Integer userId)`

**Cascade Order**:
1. Soft delete semua `UserBilling` → `deletedAt = now()`
2. Soft delete semua `BilledUsers` → `deletedAt = now()`
3. Soft delete `Users` → `deletedAt = now()`

**Protection**:
- ❌ Tidak bisa delete jika ada `UserBilling` dengan `lockedUpdate = true`

```java
// Usage example
usersCascadeService.softDeleteUserWithCascade(userId);
```

---

### 3. Lock UserBilling - Prevent Updates/Deletes
**Rule**: Jika `UserBilling` pada `paidAmount` tidak sama dengan 0 **DAN** `paymentStatus` sama dengan `PAID`, maka `lockedUpdate = true`. Semua terkait user dan tagihan ini tidak dapat diubah atau di-delete.

**Implementation** (auto handled by business logic):
- Di `UserBillingServiceImpl` atau `PaymentService` saat payment:
```java
// When payment is made
if (userBilling.getPaidAmount().compareTo(BigDecimal.ZERO) > 0 
    && userBilling.getPaymentStatus() == PaymentStatus.PAID) {
    userBilling.setLockedUpdate(true);
}
```

**Validation Queries**:
```java
// Check if UserBilling is locked
userBillingRepository.existsLockedByBilledUserRefId(billedUserRefId);
userBillingRepository.existsLockedByUserId(userId);
```

---

### 4. Cascade Update MBillings → Billing → UserBilling
**Service**: `MBillingsCascadeService` & `MBillingsCascadeServiceImpl`

**Rule**: Jika `MBillings` di-update nominal-nya, maka pada `Billing` dan `UserBilling` harus ikut terubah **HANYA** pada:
- Bulan yang akan datang (`billingCollectDate > CURRENT_DATE`)
- Status pembayaran masih open (`paidAmount = 0` AND `paymentStatus = UNPAID`)
- `lockedUpdate = false`

**Implementation**:
- Method: `updateAmountCascade(Long mBillingId, BigDecimal newAmount)`
- Count updatable: `countUpdatableBillings(Long mBillingId)`

**Update Flow**:
1. Update `MBillings.amount = newAmount`
2. Find all unlocked future `Billing` records
3. Update each `Billing.amount = newAmount`
4. Find unlocked `UserBilling` for each billing
5. Update `UserBilling.baseAmount = newAmount`
6. Database auto-calculate: `netAmount` & `remainingAmount`

```java
// Usage example
int updated = mBillingsCascadeService.updateAmountCascade(mBillingId, newAmount);
log.info("Updated {} records", updated);
```

---

## Repository Query Methods Added

### UserBillingRepository
```java
// Find by billedUserRef (BilledUsers)
List<UserBilling> findByBilledUserRefId(Long billedUserRefId);

// Check locked records
boolean existsLockedByBilledUserRefId(Long billedUserRefId);
boolean existsLockedByUserId(Integer userId);

// Find by User ID
List<UserBilling> findByBilledUserId(Integer userId);

// Find by Billing ID
List<UserBilling> findByBillingId(Long billingId);

// Find unlocked (updatable) by Billing ID
List<UserBilling> findUnlockedByBillingId(Long billingId);
```

### BilledUsersRepository
```java
// Find by User ID (Integer for Users entity)
List<BilledUsers> findByUserIdInt(Integer userId);
```

### BillingRepository
```java
// Find unlocked future billings
List<Billing> findUnlockedFutureBillingsByMBillingId(Long mBillingId);

// Count unlocked billings
int countUnlockedFutureBillingsByMBillingId(Long mBillingId);
```

---

## Usage in Controllers

### Delete BilledUser
```java
@DeleteMapping("/billed-users/{id}")
public ResponseEntity<ApiResponse<Void>> deleteBilledUser(@PathVariable Long id) {
    // Check if can delete
    if (!billedUsersService.canDeleteBilledUser(id)) {
        throw new ApiException(ApiErrorCode.BUSINESS_LOGIC_ERROR,
            "Cannot delete - has paid UserBilling");
    }
    
    // Soft delete with cascade
    billedUsersService.softDeleteBilledUser(id);
    
    return ResponseEntity.ok(new ApiResponse<>("Success", null));
}
```

### Delete User
```java
@DeleteMapping("/users/{id}")
public ResponseEntity<ApiResponse<Void>> deleteUser(@PathVariable Integer id) {
    // Check if can delete
    if (!usersCascadeService.canDeleteUser(id)) {
        throw new ApiException(ApiErrorCode.BUSINESS_LOGIC_ERROR,
            "Cannot delete - has paid billings");
    }
    
    // Soft delete with cascade
    usersCascadeService.softDeleteUserWithCascade(id);
    
    return ResponseEntity.ok(new ApiResponse<>("Success", null));
}
```

### Update MBilling Amount
```java
@PutMapping("/mbillings/{id}/amount")
public ResponseEntity<ApiResponse<Map<String, Integer>>> updateAmount(
        @PathVariable Long id,
        @RequestBody UpdateAmountRequest request) {
    
    // Check how many can be updated
    int updatable = mBillingsCascadeService.countUpdatableBillings(id);
    
    // Update with cascade
    int updated = mBillingsCascadeService.updateAmountCascade(id, request.getNewAmount());
    
    Map<String, Integer> result = Map.of(
        "updatableBillings", updatable,
        "updatedRecords", updated
    );
    
    return ResponseEntity.ok(new ApiResponse<>("Success", result));
}
```

---

## Data Integrity Protection

### ✅ Protected Operations
1. **Cannot delete BilledUser** if any related UserBilling is locked (paid)
2. **Cannot delete User** if any related UserBilling is locked (paid)
3. **Cannot update amount** on past billings or paid billings
4. **Cascade deletes** maintain referential integrity

### ⚠️ Important Notes
- Always use service methods, NOT direct repository delete
- `lockedUpdate = true` prevents ALL modifications
- Future billings only (`billingCollectDate > CURRENT_DATE`)
- Soft delete preserves audit trail

---

## Database Columns Used

### UserBilling
- `locked_update` BOOLEAN - Prevents updates/deletes when true
- `paid_amount` DECIMAL - Non-zero when payment made
- `payment_status` ENUM - UNPAID, PARTIAL, PAID, OVERPAID
- `billed_user_ref_id` FK - Reference to BilledUsers
- `billed_user_id` FK - Reference to Users
- `deleted_at` TIMESTAMP - Soft delete marker

### Billing
- `locked_update` BOOLEAN - Prevents updates when true
- `billing_collect_date` DATE - Used for future billing check
- `deleted_at` TIMESTAMP - Soft delete marker

### BilledUsers
- `deleted_at` TIMESTAMP - Soft delete marker
- `user_id` FK - Reference to Users

---

## Testing Scenarios

### Scenario 1: Delete Unpaid BilledUser
```
1. Create MBilling with amount 1,000,000
2. Add 4 users to BilledUsers
3. Auto-generate Billing & UserBilling
4. Delete one BilledUser (unpaid)
✅ Result: BilledUser & all related UserBilling deleted
```

### Scenario 2: Delete Paid BilledUser
```
1. Create MBilling with amount 1,000,000
2. Add 4 users to BilledUsers
3. Auto-generate Billing & UserBilling
4. Process payment for one user (paidAmount = 1,000,000, status = PAID)
5. Try to delete that BilledUser
❌ Result: Error - "Cannot delete - has paid UserBilling"
```

### Scenario 3: Update Future MBilling Amount
```
1. Create MBilling MONTHLY from Dec 2025 - Nov 2026
2. Already in Feb 2026
3. Update amount from 1,000,000 to 1,500,000
✅ Result: Only Mar 2026 - Nov 2026 billings updated (future only)
```

### Scenario 4: Update MBilling with Paid Billings
```
1. Create MBilling MONTHLY from Dec 2025 - Nov 2026  
2. User paid for Mar 2026 billing
3. Update amount from 1,000,000 to 1,500,000
✅ Result: Mar 2026 NOT updated (locked), Apr-Nov 2026 updated
```

---

## Error Messages

### Delete Blocked
```json
{
  "error": "BUSINESS_LOGIC_ERROR",
  "message": "Tidak dapat menghapus BilledUser karena ada UserBilling yang sudah terbayar (locked)"
}
```

```json
{
  "error": "BUSINESS_LOGIC_ERROR",
  "message": "Tidak dapat menghapus User karena ada UserBilling yang sudah terbayar (locked)"
}
```

### Not Found
```json
{
  "error": "RESOURCE_NOT_FOUND",
  "message": "BilledUser tidak ditemukan"
}
```

---

## Summary

✅ **Requirement 1**: BilledUsers delete → cascade UserBilling delete  
✅ **Requirement 2**: Users delete → cascade BilledUsers & UserBilling delete  
✅ **Requirement 3**: Paid UserBilling → locked, cannot edit/delete  
✅ **Requirement 4**: MBilling update → cascade to future unlocked billings  

Semua business logic sudah terimplementasi dengan protection dan validation yang proper! 🎉
