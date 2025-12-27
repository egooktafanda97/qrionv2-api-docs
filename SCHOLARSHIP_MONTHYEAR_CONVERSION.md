# Scholarship & Billing Scholarship Monthly Conversion

## Overview
Aligned `BillingScholarship` and `MBillingScholarshipMonthly` with the `yyyy-MM` format conversion that was applied to `MBillingMonthlyActive`. This ensures consistency across all monthly billing entities.

## Changes Summary

### 1. Entity Changes

#### `MBillingScholarshipMonthly.java`
**Converted from:**
```java
@Column(nullable = false)
private Integer month;  // 1-12
```

**To:**
```java
@Column(name = "month_year", nullable = false, length = 7)
private String monthYear;  // yyyy-MM format, e.g., "2025-01"
```

**Changes:**
- Changed field name from `month` to `monthYear`
- Changed type from `Integer` to `String`
- Updated column name in table from `month` to `month_year`
- Updated unique constraint from `(m_billing_scholarship_monthly_id, month)` to `(m_billing_scholarship_monthly_id, month_year)`
- Updated validation to parse `yyyy-MM` format instead of checking 1-12 range

### 2. Database Schema Changes

#### `m_billing_scholarship_monthly` table
```sql
-- Old structure
ALTER TABLE m_billing_scholarship_monthly
CHANGE COLUMN month month INT NOT NULL,
DROP CONSTRAINT uk_mbilling_scholarship_monthly_month,
ADD UNIQUE KEY uk_mbilling_scholarship_monthly_month (m_billing_scholarship_monthly_id, month);

-- New structure
ALTER TABLE m_billing_scholarship_monthly
CHANGE COLUMN month month_year VARCHAR(7) NOT NULL,
DROP CONSTRAINT uk_mbilling_scholarship_monthly_month,
ADD UNIQUE KEY uk_mbilling_scholarship_monthly_month (m_billing_scholarship_monthly_id, month_year);

-- Example data migration (if needed):
UPDATE m_billing_scholarship_monthly
SET month_year = CONCAT('2025-', LPAD(CAST(month AS CHAR), 2, '0'))
WHERE month_year IS NULL;
```

### 3. Repository Changes

#### `MBillingScholarshipMonthlyRepository.java`

**Updated methods:**
```java
// Changed: existsByBillingScholarshipIdAndMonth()
// To: existsByBillingScholarshipIdAndMonthYear()
@Query("SELECT COUNT(m) > 0 FROM MBillingScholarshipMonthly m " +
        "WHERE m.mBillingScholarshipMonthly.id = :billingScholarshipId AND m.monthYear = :monthYear")
boolean existsByBillingScholarshipIdAndMonthYear(
        @Param("billingScholarshipId") Long billingScholarshipId,
        @Param("monthYear") String monthYear);
```

**Added:**
- Ordering by `monthYear` in `findByBillingScholarshipId()` query

### 4. Service Logic Changes

#### `BillingScholarshipServiceImpl.java`

**In `create()` method:**
```java
// OLD: Storing integer months
if (request.getMonths() != null && !request.getMonths().isEmpty()) {
    for (Integer month : request.getMonths()) {
        MBillingScholarshipMonthly monthlyRecord = MBillingScholarshipMonthly.builder()
                .mBillingScholarshipMonthly(billingScholarship)
                .month(month)  // Integer 1-12
                .build();
        mBillingScholarshipMonthlyRepository.save(monthlyRecord);
    }
}

// NEW: Converting to yyyy-MM format
if (request.getMonths() != null && !request.getMonths().isEmpty()) {
    List<MBillingMonthlyActive> activeMonthList = mBillingMonthlyActiveRepository
            .findByMBillingId(mBilling.getId());
    String referenceYearMonth = activeMonthList.isEmpty() ? 
            java.time.YearMonth.now().toString() : activeMonthList.get(0).getMonthYear();
    int referenceYear = Integer.parseInt(referenceYearMonth.substring(0, 4));

    for (Integer month : request.getMonths()) {
        String monthYear = String.format("%d-%02d", referenceYear, month);
        MBillingScholarshipMonthly monthlyRecord = MBillingScholarshipMonthly.builder()
                .mBillingScholarshipMonthly(billingScholarship)
                .monthYear(monthYear)  // String "yyyy-MM"
                .build();
        mBillingScholarshipMonthlyRepository.save(monthlyRecord);
    }
}
```

**In `update()` method:**
- Same conversion as `create()` method applied

**In `delete()` method:**
```java
// OLD: Directly get month integers
List<Integer> months = mBillingScholarshipMonthlyRepository
        .findByBillingScholarshipId(billingScholarship.getId())
        .stream()
        .map(MBillingScholarshipMonthly::getMonth)
        .collect(Collectors.toList());

// NEW: Parse monthYear strings to extract month numbers
List<Integer> months = mBillingScholarshipMonthlyRepository
        .findByBillingScholarshipId(billingScholarship.getId())
        .stream()
        .map(m -> java.time.YearMonth.parse(m.getMonthYear()).getMonthValue())
        .collect(Collectors.toList());
```

**In `toResponse()` method:**
```java
// OLD: Direct mapping
List<Integer> months = mBillingScholarshipMonthlyRepository
        .findByBillingScholarshipId(billingScholarship.getId())
        .stream()
        .map(MBillingScholarshipMonthly::getMonth)
        .sorted()
        .collect(Collectors.toList());

// NEW: Parse monthYear to extract month numbers for API response
List<Integer> months = mBillingScholarshipMonthlyRepository
        .findByBillingScholarshipId(billingScholarship.getId())
        .stream()
        .map(m -> java.time.YearMonth.parse(m.getMonthYear()).getMonthValue())
        .sorted()
        .collect(Collectors.toList());
```

**In `updateUserBillingDiscountsWithMonths()` method:**
```java
// OLD: Direct month extraction
List<Integer> oldMonths = mBillingScholarshipMonthlyRepository
        .findByBillingScholarshipId(billingScholarshipId)
        .stream()
        .map(MBillingScholarshipMonthly::getMonth)
        .collect(Collectors.toList());

// NEW: Parse monthYear strings
List<Integer> oldMonths = mBillingScholarshipMonthlyRepository
        .findByBillingScholarshipId(billingScholarshipId)
        .stream()
        .map(m -> java.time.YearMonth.parse(m.getMonthYear()).getMonthValue())
        .collect(Collectors.toList());
```

## API Contract

**Note:** The REST API still accepts and returns `List<Integer>` for months (1-12) in DTOs:
- `BillingScholarshipRequest.getMonths()` → `List<Integer>` (1-12)
- `BillingScholarshipResponse.getMonths()` → `List<Integer>` (1-12)

**Conversion happens internally in the service layer:**
1. Receives `[1, 2, 3]` from client
2. Converts to `["2025-01", "2025-02", "2025-03"]` using current year from `MBillingMonthlyActive`
3. Stores in database as `monthYear` strings
4. Converts back to `[1, 2, 3]` when returning to client

## Consistency with MBillingMonthlyActive

Both entities now follow the same pattern:
- **`MBillingMonthlyActive`**: `monthYear` (String, `yyyy-MM`) - Master months that can be billed
- **`MBillingScholarshipMonthly`**: `monthYear` (String, `yyyy-MM`) - Months that have scholarship discount

This ensures:
- Scholarships can only be applied to months that are configured in `MBillingMonthlyActive`
- Validation in `BillingScholarshipServiceImpl.create()` checks that requested scholarship months are a subset of active billing months
- Data consistency across related entities

## Testing Checklist

- [ ] Database migration applied (rename `month` → `month_year`, update constraints)
- [ ] Create scholarship with months `[1, 2, 3]` → Verify stored as `["2025-01", "2025-02", "2025-03"]`
- [ ] Update scholarship months → Verify monthYear entries updated correctly
- [ ] Delete scholarship → Verify monthYear entries cleaned up
- [ ] Fetch scholarship response → Verify months returned as `[1, 2, 3]`
- [ ] Validate month subset enforcement with `MBillingMonthlyActive`
- [ ] Test discount application logic still works month-aware

## Build Status
✅ Maven build successful
- No compile errors
- 17 pre-existing warnings (benign, unrelated to these changes)

