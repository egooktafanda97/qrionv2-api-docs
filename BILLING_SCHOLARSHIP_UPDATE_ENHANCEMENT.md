# Billing Scholarship Update Enhancement - Month-Based Selection

## Overview
Enhanced the `BillingScholarshipController` update endpoint to support **month-based filtering** when updating scholarship discounts. This allows users to selectively apply discount updates only to specific months instead of all months.

## Problem Statement
**Before**: When updating a billing scholarship, the discount value would be applied to ALL unpaid UserBillings regardless of month:
```
PUT /api/billing-scholarships/{id}
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 50,
  "months": [1, 2]  // Selected months, but ignored during update
}
```

Even if scholarship was created with 5 months, updating it would apply discount to all unpaid billings, not just months 1-2.

**After**: Now the update respects the month selection and ONLY updates UserBillings that match:
- The scholarship 
- AND the specified months
- AND are not already PAID/locked

## Solution Architecture

### Key Components Modified

#### 1. **BillingScholarshipServiceImpl.update()**
Changed to call a new month-aware update method:

```java
@Override
@Transactional
public BillingScholarshipResponse update(Long id, BillingScholarshipRequest request) {
    // ... validation ...
    
    // UPDATE: Now passes months to the update method
    updateUserBillingDiscountsWithMonths(
        scholarship.getId(),
        mBilling.getId(),
        request.getDiscountValue(),
        scholarship.getName(),
        request.getMonths(),  // NEW: Month selection parameter
        yayasanId,
        institutionId);
    
    return toResponse(billingScholarship);
}
```

#### 2. **New Method: updateUserBillingDiscountsWithMonths()**
Implemented month-aware discount update logic:

```java
/**
 * Update discount value di UserBilling dengan MONTH filtering
 * CRITICAL: Hanya update UserBilling yang:
 * 1. Belum PAID dan belum locked
 * 2. Bulannya MASUK dalam scholarship months yang diberikan
 */
private void updateUserBillingDiscountsWithMonths(
    Long scholarshipId, 
    Long mBillingId,
    BigDecimal newDiscountValue, 
    String scholarshipName,
    List<Integer> targetMonths,  // [1, 2, 3] = Jan, Feb, Mar
    Integer yayasanId, 
    Integer institutionId)
```

### Update Logic Flow

```
┌─ Update BillingScholarship months [1,2]
├─ Delete old month records from MBillingScholarshipMonthly
├─ Insert new month records [1,2]
│
└─ Update UserBilling discounts:
   ├─ Get ALL unpaid UserBillings for this MBilling
   │
   └─ For each UserBilling:
      ├─ SKIP if PAID or locked
      ├─ SKIP if month NOT in [1,2]
      ├─ UPDATE discount if month in [1,2]
      └─ Log the action
```

### Example Scenarios

#### Scenario 1: Create with 5 months, Update for 2 months only
```
CREATE:
- Scholarship created for months [1, 2, 3, 4, 5]
- All Jan-May billings get discount

UPDATE:
PUT /api/billing-scholarships/123
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 30,  // Changed from 50 to 30
  "months": [1, 2]       // Update ONLY Jan & Feb
}

RESULT:
- MBillingScholarshipMonthly: [1, 2] (updated)
- UserBilling Jan: discount updated to 30% ✅
- UserBilling Feb: discount updated to 30% ✅
- UserBilling Mar-May: discount stays 50% ✅
- UserBilling PAID status: skipped ✅
```

#### Scenario 2: Different discount for different periods
```
CREATE Month 1-12 with 50% discount

UPDATE 1: Months [1,2,3] with 40% discount
UPDATE 2: Months [4,5,6] with 35% discount
UPDATE 3: Months [7-12] with 25% discount

RESULT: Flexible discount structure per period
```

## Implementation Details

### Method Signature
```java
private void updateUserBillingDiscountsWithMonths(
    Long scholarshipId,              // Scholarship ID
    Long mBillingId,                 // Master Billing ID
    BigDecimal newDiscountValue,     // New discount amount/percentage
    String scholarshipName,          // For logging & discount program name
    List<Integer> targetMonths,      // [1,2,3] = which months to update
    Integer yayasanId,               // Permission check
    Integer institutionId)           // Permission check
```

### Processing Logic

```
1. NULL/EMPTY months handling:
   - If targetMonths is null/empty → call original updateUserBillingDiscounts
   - Apply discount to ALL unpaid UserBillings
   
2. Get unpaid UserBillings:
   - Query: findUnpaidByMBillingIdAndYayasanAndInstitution
   - Filters: UNPAID status + permission check
   
3. Filter loop:
   - For each UserBilling:
     ├─ Check PAID/locked status → SKIP if true
     ├─ Check billing month exists → SKIP if null
     ├─ Extract month from billing_collect_date
     ├─ Check month in targetMonths → SKIP if not match
     └─ UPDATE discount if all checks pass
     
4. Logging:
   - Track: updated_count, skipped_wrong_month, skipped_paid_locked
   - Warn if no records updated
```

### Sample Logs

```
[INFO] Processing month-based update: Found 24 total unpaid UserBillings 
       for MBilling ID: 5, targetMonths: [1, 2]
[INFO] Updated UserBilling ID 101: Month=1, discount 50.00 -> 30.00
[INFO] Updated UserBilling ID 102: Month=2, discount 50.00 -> 30.00
[DEBUG] Skipping UserBilling ID 103 - billing month 3 not in target months [1, 2]
[WARN] Skipping UserBilling ID 104 - already PAID or locked
[INFO] Scholarship 'Merit Scholarship' month-based update complete: 
       Updated=2, SkippedWrongMonth=20, SkippedPaidOrLocked=2
```

## API Endpoint

### PUT /api/billing-scholarships/{id}

**Request:**
```json
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 30,
  "maxDiscountAmount": 500000,
  "notes": "Updated for first two months only",
  "months": [1, 2]  // CRITICAL: Only update these months
}
```

**Response:**
```json
{
  "success": true,
  "message": "Potongan beasiswa berhasil diperbarui",
  "data": {
    "id": 1,
    "scholarshipId": 1,
    "scholarshipName": "Merit Scholarship",
    "mBillingId": 1,
    "mBillingName": "Monthly Billing 2024",
    "months": [1, 2],
    "discountType": "PERCENTAGE",
    "discountValue": 30,
    "maxDiscountAmount": 500000,
    "notes": "Updated for first two months only",
    "createdAt": "2025-11-28T10:00:00Z",
    "updatedAt": "2025-11-28T13:45:00Z"
  }
}
```

## Data Flow

### MBillingScholarshipMonthly Table
Tracks which months have the scholarship:

```sql
UPDATE m_billing_scholarship_monthly
SET months = [1, 2]  -- Updated from [1,2,3,4,5]
WHERE billing_scholarship_id = 1
```

### UserBilling Table
Selective discount updates based on month match:

```sql
-- UPDATED (month matches [1, 2]):
UPDATE user_billings 
SET discount_value = 30, 
    discount_program = 'Beasiswa - Merit Scholarship',
    updated_at = NOW()
WHERE user_billing_id IN (101, 102)  -- Jan, Feb billings

-- NOT UPDATED (month not in [1, 2]):
-- user_billing_id: 103, 104, ... (Mar, Apr, etc.)
-- discount_value stays at: 50
```

## Benefits

| Aspect | Before | After |
|--------|--------|-------|
| **Month Selection** | Ignored on update | Respected ✅ |
| **Discount Scope** | Applied to all unpaid | Applied only to matching months ✅ |
| **Flexibility** | Cannot modify specific months | Can update different months with different rates ✅ |
| **User Control** | Limited | Full control over scholarship duration ✅ |
| **Data Consistency** | All or nothing | Granular control ✅ |

## Edge Cases Handled

1. ✅ **No months provided**: Falls back to updating all unpaid UserBillings
2. ✅ **Empty months list**: Same as above
3. ✅ **PAID/Locked billings**: Automatically skipped
4. ✅ **Billing with no collect date**: Skipped with warning
5. ✅ **User not in scholarship**: Not affected (filtering happens elsewhere)
6. ✅ **Month outside active months**: Handled by validation (not filtered here)
7. ✅ **Multiple updates**: Each update only affects specified months

## Testing Recommendations

### Test Case 1: Basic Month-Based Update
```
1. Create scholarship with months [1,2,3,4,5]
2. Create UserBillings for all 12 months
3. UPDATE scholarship to months [1,2] with new discount
4. Verify: Jan & Feb updated, Mar-Dec unchanged
```

### Test Case 2: Partial Update with Locked Records
```
1. Create billings with discount 50
2. Lock some March billings
3. UPDATE to months [1,2,3] with discount 30
4. Verify: Jan & Feb updated, Mar (locked) skipped
```

### Test Case 3: Multiple Sequential Updates
```
1. Create scholarship months [1-12], discount 50
2. UPDATE months [1-3] to 40
3. UPDATE months [4-6] to 35
4. UPDATE months [7-12] to 25
5. Verify: Each group has correct discount
```

### Test Case 4: Remove All Months
```
1. UPDATE with empty months array
2. Verify: All unpaid billings get new discount (fallback behavior)
```

## Backward Compatibility

✅ **Fully backward compatible**:
- Old API calls without months → Falls back to original behavior
- Existing workflows unaffected
- No database migration needed

## Performance Considerations

- **Query**: `findUnpaidByMBillingIdAndYayasanAndInstitution` - indexed
- **Loop operations**: Handled efficiently (batch updates recommended for large datasets)
- **Index recommendations**: 
  ```sql
  CREATE INDEX idx_user_billings_mbilling_status 
  ON user_billings(m_billing_id, payment_status)
  
  CREATE INDEX idx_billing_collect_date 
  ON billings(billing_collect_date)
  ```

## Deployment Notes

1. No database schema changes required
2. New method is internal (private) - no API changes
3. Existing endpoints remain compatible
4. Recommended: Add monitoring for update operations

## Related Documentation

- `SCHOLARSHIP_SCHEMA_EXPLANATION.md` - Schema details
- `SCHOLARSHIP_RELATIONSHIP_DIAGRAM.md` - Relationship diagram
- `documentations/08-scholarship-api.md` - Full API documentation
