# Implementation Guide - Billing Scholarship Month-Based Update

## Quick Start

### How to Use the Enhanced Update Feature

#### 1. Update Scholarship for Specific Months Only

**Scenario**: Scholarship was created for 12 months (Jan-Dec) with 50% discount, but you want to adjust the first 3 months to 40% discount.

**API Call**:
```bash
curl -X PUT http://localhost:8081/api/billing-scholarships/1 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "scholarshipId": 1,
    "mBillingId": 1,
    "discountType": "PERCENTAGE",
    "discountValue": 40,
    "maxDiscountAmount": 500000,
    "notes": "Adjusted discount for Q1 2025",
    "months": [1, 2, 3]
  }'
```

**What Happens**:
1. ✅ MBillingScholarshipMonthly updated to [1, 2, 3]
2. ✅ UserBilling for Jan, Feb, Mar updated to 40% discount
3. ✅ UserBilling for Apr-Dec remain at 50% discount
4. ✅ Any PAID or locked billings are skipped

---

## Detailed Flow Examples

### Example 1: Progressive Discount Reduction

**Situation**:
- Master Billing: 12 months
- Current Scholarship: 50% for all months
- Goal: Reduce to 35% for Q4 only

**Implementation**:

```java
// 1. UPDATE months [10, 11, 12] with 35%
PUT /api/billing-scholarships/1
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 35,
  "months": [10, 11, 12]  // October, November, December
}

// Result:
// - Oct-Dec billing: 35% discount ✅
// - Jan-Sep billing: 50% discount ✅
// - PAID billings: unchanged ✅
// - Locked billings: unchanged ✅
```

---

### Example 2: Mid-Year Adjustment with Multiple Updates

**Situation**: Adjust different quarters to different rates

**Step 1**: Original setup (all months 50%)
```
Months [1-12] → 50% discount
All users affected
```

**Step 2**: Q1 rate reduction (Jan-Mar to 40%)
```
PUT /api/billing-scholarships/1
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 40,
  "months": [1, 2, 3]
}

Result:
- Jan-Mar: 40% ✅
- Apr-Dec: 50% ✅
```

**Step 3**: Q2 adjustment (Apr-Jun to 45%)
```
PUT /api/billing-scholarships/1
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 45,
  "months": [4, 5, 6]
}

Result:
- Jan-Mar: 40% (unchanged) ✅
- Apr-Jun: 45% ✅
- Jul-Dec: 50% ✅
```

**Step 4**: Continue for remaining quarters...

**Final State**:
```
Q1 (Jan-Mar):  40% discount
Q2 (Apr-Jun):  45% discount
Q3 (Jul-Sep):  50% discount
Q4 (Oct-Dec):  35% discount
```

---

### Example 3: Fixed Amount Discount Update

**Scenario**: Change from percentage to fixed amount for first 2 months

**Request**:
```json
PUT /api/billing-scholarships/2
{
  "scholarshipId": 2,
  "mBillingId": 1,
  "discountType": "FIXED_AMOUNT",
  "discountValue": 500000,
  "maxDiscountAmount": 500000,
  "notes": "Special discount for early enrollment",
  "months": [1, 2]
}
```

**Effect**:
- Each billing in Jan & Feb gets flat Rp 500,000 discount
- Other months maintain their percentage
- PAID/locked billings skipped

---

## Database State Changes

### Before Update
```sql
-- MBillingScholarshipMonthly (Original)
┌─────┬───────────────────┬───────┐
│ id  │ billing_scholarship_id │ month │
├─────┼───────────────────┼───────┤
│ 1   │ 1                 │ 1     │
│ 2   │ 1                 │ 2     │
│ 3   │ 1                 │ 3     │
│ 4   │ 1                 │ 4     │
│ 5   │ 1                 │ 5     │
...
│ 12  │ 1                 │ 12    │
└─────┴───────────────────┴───────┘

-- UserBilling (Original)
┌──────┬────────┬──────────┬────────────────┐
│ id   │ month  │ discount │ discount_value │
├──────┼────────┼──────────┼────────────────┤
│ 101  │ 1      │ 50%      │ 500000         │
│ 102  │ 2      │ 50%      │ 500000         │
│ 103  │ 3      │ 50%      │ 500000         │
...
└──────┴────────┴──────────┴────────────────┘
```

### After UPDATE months [1, 2, 3] with 40%
```sql
-- MBillingScholarshipMonthly (Updated)
┌─────┬───────────────────┬───────┐
│ id  │ billing_scholarship_id │ month │
├─────┼───────────────────┼───────┤
│ 1   │ 1                 │ 1     │  ✅ Kept
│ 2   │ 1                 │ 2     │  ✅ Kept
│ 3   │ 1                 │ 3     │  ✅ Kept
│ 4   │ 1                 │ 4     │  ❌ Deleted
│ 5   │ 1                 │ 5     │  ❌ Deleted
...
│ 12  │ 1                 │ 12    │  ❌ Deleted
└─────┴───────────────────┴───────┘
-- (Deleted old records [4-12], kept [1-3])

-- UserBilling (Updated - Selective)
┌──────┬────────┬──────────┬────────────────┐
│ id   │ month  │ discount │ discount_value │
├──────┼────────┼──────────┼────────────────┤
│ 101  │ 1      │ 40%      │ 400000         │  ✅ Updated
│ 102  │ 2      │ 40%      │ 400000         │  ✅ Updated
│ 103  │ 3      │ 40%      │ 400000         │  ✅ Updated
│ 104  │ 4      │ 50%      │ 500000         │  ⏭️ Skipped (wrong month)
│ 105  │ 5      │ 50%      │ 500000         │  ⏭️ Skipped (wrong month)
...
│ 112  │ 12     │ 50%      │ 500000         │  ⏭️ Skipped (wrong month)
└──────┴────────┴──────────┴────────────────┘
```

---

## Error Handling

### Case 1: Invalid Month Range
```
PUT /api/billing-scholarships/1
{
  "months": [1, 2, 13]  // ❌ Invalid month
}

Response: 400 Bad Request
Message: "Month must be between 1-12"
```

### Case 2: Months Not in Active Range
```
PUT /api/billing-scholarships/1
{
  "months": [10, 11, 12]  // But MBilling only has 6 months
}

Response: 400 Bad Request
Message: "Months [10, 11, 12] not in active MBilling months [1-6]"
```

### Case 3: Scholarship Not Found
```
PUT /api/billing-scholarships/999
{
  "scholarshipId": 999  // ❌ Doesn't exist
}

Response: 404 Not Found
Message: "Scholarship not found with id: 999"
```

### Case 4: Permission Denied
```
User from Yayasan A tries to update:
  Scholarship from Yayasan B

Response: 403 Forbidden
Message: "BillingScholarship tidak ditemukan atau bukan bagian dari 
           yayasan/institusi Anda"
```

---

## Logging Examples

### Successful Update
```
[INFO] PUT /api/billing-scholarships/1 - Update billing scholarship
[INFO] Processing month-based update: Found 24 total unpaid UserBillings 
       for MBilling ID: 1, targetMonths: [1, 2, 3]
[INFO] Updated UserBilling ID 101: Month=1, discount 500000.00 -> 400000.00
[INFO] Updated UserBilling ID 102: Month=2, discount 500000.00 -> 400000.00
[INFO] Updated UserBilling ID 103: Month=3, discount 500000.00 -> 400000.00
[DEBUG] Skipping UserBilling ID 104 - billing month 4 not in target months [1, 2, 3]
[DEBUG] Skipping UserBilling ID 105 - billing month 5 not in target months [1, 2, 3]
...
[WARN] Skipping UserBilling ID 110 - already PAID or locked
[WARN] Skipping UserBilling ID 111 - already PAID or locked
[INFO] Scholarship 'Merit Scholarship' month-based update complete: 
       Updated=3, SkippedWrongMonth=19, SkippedPaidOrLocked=2
[INFO] Updated billing scholarship id=1: discountType=PERCENTAGE, 
       discountValue=40, months=[1, 2, 3]
```

### Edge Case: No Months Specified (Fallback)
```
[INFO] PUT /api/billing-scholarships/1 - Update billing scholarship
[INFO] No months specified, updating all unpaid UserBillings 
       for scholarship ID: 1
[INFO] Successfully updated 23 UserBillings for scholarship ID: 1
```

### Warning: No Matching Records
```
[WARN] No UserBilling records were updated for scholarship 'Merit Scholarship' 
       with months [7, 8, 9]. 
       Check if months match billing collect dates.
```

---

## Testing Queries

### Verify Update Was Applied

```sql
-- Check MBillingScholarshipMonthly
SELECT * FROM m_billing_scholarship_monthly 
WHERE billing_scholarship_id = 1
ORDER BY month;
-- Expected: [1, 2, 3]

-- Check UserBilling updates
SELECT id, billing_collect_date, discount_value, discount_program, updated_at
FROM user_billings
WHERE m_billing_id = 1 
  AND EXTRACT(MONTH FROM billing_collect_date) IN (1, 2, 3)
  AND payment_status != 'PAID'
ORDER BY billing_collect_date;
-- Expected: All have new discount_value = 400000
```

### Verify Non-Matching Were Skipped

```sql
-- Check UserBilling NOT updated (months 4-12)
SELECT id, billing_collect_date, discount_value
FROM user_billings
WHERE m_billing_id = 1 
  AND EXTRACT(MONTH FROM billing_collect_date) IN (4, 5, 6, 7, 8, 9, 10, 11, 12)
  AND payment_status != 'PAID'
ORDER BY billing_collect_date;
-- Expected: All have old discount_value = 500000
```

---

## Performance Tips

### For Large-Scale Updates (>1000 records):

**Current approach** (efficient for < 1000):
```java
// Individual saves in loop
userBillingRepository.save(ub);
```

**Recommended for large updates**:
```java
// Batch save
List<UserBilling> toUpdate = new ArrayList<>();
for (UserBilling ub : allUserBillings) {
    if (shouldUpdate(ub)) {
        ub.setDiscountValue(newValue);
        toUpdate.add(ub);
    }
}
// Batch save all at once
userBillingRepository.saveAll(toUpdate);
```

**Expected Performance**:
- 100 records: ~500ms
- 1000 records: ~5s
- 10000 records: ~50s (recommend batch approach)

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No records updated | Month mismatch | Check billing_collect_date in database |
| Only some updated | Some PAID/locked | Expected behavior - skipped intentionally |
| Old discount still present | Caching | Clear cache or restart application |
| Permission denied | Wrong yayasan | Verify user's yayasan/institution assignment |
| Months not updating | Validation failed | Check months are subset of active months |

---

## Related Operations

### After UPDATE: Check Related Data

```java
// 1. Verify BillingScholarship
GET /api/billing-scholarships/1
// Should return: months: [1, 2, 3]

// 2. Check affected UserBillings
GET /api/user-billings?mBillingId=1&status=UNPAID

// 3. Verify MBillings remains unchanged
GET /api/mbillings/1
// Should have all 12 months in MBillingMonthlyActive
```

### Revert Operation (If Needed)

```json
PUT /api/billing-scholarships/1
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "discountType": "PERCENTAGE",
  "discountValue": 50,
  "months": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]  // Revert to all months
}
```

---

## FAQ

**Q: Does this affect already PAID billings?**  
A: No, PAID and locked billings are automatically skipped.

**Q: Can I update different months to different values?**  
A: Yes, make separate UPDATE calls for each month group.

**Q: What if billing has no collect_date?**  
A: It's skipped with a debug log message.

**Q: Does this create new MBillingScholarshipMonthly records?**  
A: Old ones are deleted, new ones created for specified months.

**Q: Can I update a scholarship used by multiple MBillings?**  
A: No, update is specific to one BillingScholarship (scholarship-MBilling pair).

**Q: Is there a limit to how many updates I can make?**  
A: No technical limit, but large batches may be slow (optimize with batch saves).
