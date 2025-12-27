# Monthly Active Year-Month Format Upgrade Guide

## Overview
Konversi `monthlyActive` dari format integer (1-12) menjadi format string "yyyy-MM" (e.g., "2024-01", "2024-02") untuk mendukung year-month range yang lebih spesifik dalam billing system.

## Changes Made

### 1. **Entity Changes**

#### MBillingMonthlyActive.java
- **Sebelum**: `@Column(name = "month", nullable = false) private Integer month;`
- **Sesudah**: `@Column(name = "month_year", length = 7, nullable = false) private String monthYear;`
- Unique constraint: `@UniqueConstraint(columnNames = {"m_billing_id", "month_year"})`

#### MBillingsRequest.java
```java
// Sebelum
private List<Integer> monthlyActive;  // 1-12

// Sesudah
private List<String> monthlyActive;  // "yyyy-MM" format (e.g., ["2024-01", "2024-02"])
```

#### MBillingsResponse.java
```java
// Sebelum
private List<Integer> monthlyActive;

// Sesudah
private List<String> monthlyActive;  // Sorted list of "yyyy-MM" strings
```

#### MBillingMonthlyActiveUpdateRequest.java
```java
// Sebelum
private List<Integer> monthlyActive;

// Sesudah
private List<String> monthlyActive;  // "yyyy-MM" format
```

### 2. **Repository Updates**

#### MBillingMonthlyActiveRepository.java
```java
// Query parameter updated from Integer month to String monthYear
@Query("SELECT m FROM MBillingMonthlyActive m WHERE m.mBillings = :mBillings AND m.monthYear = :monthYear")
Optional<MBillingMonthlyActive> findByMBillingsAndMonthYear(
    @Param("mBillings") MBillings mBillings,
    @Param("monthYear") String monthYear);

@Query("SELECT m FROM MBillingMonthlyActive m WHERE m.mBillings = :mBillings ORDER BY m.monthYear")
List<MBillingMonthlyActive> findByMBillingMonthly(@Param("mBillings") MBillings mBillings);

@Query("SELECT m FROM MBillingMonthlyActive m WHERE m.mBillings = :mBillings AND m.monthYear = :monthYear")
List<MBillingMonthlyActive> findByMBillingMonthlyAndMonthYear(
    @Param("mBillings") MBillings mBillings,
    @Param("monthYear") String monthYear);

@Query("SELECT COUNT(m) > 0 FROM MBillingMonthlyActive m WHERE m.mBillings = :mBillings AND m.monthYear = :monthYear")
boolean existsByMBillingMonthlyAndMonthYear(
    @Param("mBillings") MBillings mBillings,
    @Param("monthYear") String monthYear);
```

### 3. **Service Logic Updates**

#### MBillingsServiceImpl.java

**validateMonthlyActive()** - Validates "yyyy-MM" format
```java
private void validateMonthlyActive(List<String> monthlyActive) {
    if (monthlyActive == null) {
        return;
    }
    if (monthlyActive.isEmpty()) {
        throw new ApiException(ApiErrorCode.BUSINESS_RULE_VIOLATION, "Bulan aktif harus diisi");
    }
    for (String ym : monthlyActive) {
        try {
            YearMonth.parse(ym);  // Validates "yyyy-MM" format
        } catch (Exception e) {
            throw new ApiException(ApiErrorCode.BUSINESS_RULE_VIOLATION, "Format bulan harus yyyy-MM");
        }
    }
}
```

**autoGenerateBillings()** - MONTHLY type auto-generation
```java
if (mBillings.getBillingType() == BillingTypeEnum.MONTHLY) {
    LocalDate startDate = mBillings.getStartDatePeriod() != null
            ? mBillings.getStartDatePeriod()
            : LocalDate.of(now.getYear(), 1, 1);
    LocalDate endDate = mBillings.getEndDatePeriod() != null
            ? mBillings.getEndDatePeriod()
            : LocalDate.of(now.getYear(), 12, 31);

    // Fetch active year-months as List<String> (e.g., ["2024-01", "2024-02"])
    List<MBillingMonthlyActive> monthlyActives = mBillingMonthlyActiveRepository.findByMBillingMonthly(mBillings);
    List<String> activeYearMonths = monthlyActives.stream()
            .map(MBillingMonthlyActive::getMonthYear)
            .collect(Collectors.toList());

    // If empty, generate all months in start-end range
    if (activeYearMonths.isEmpty()) {
        YearMonth startYm = YearMonth.from(startDate);
        YearMonth endYm = YearMonth.from(endDate);
        YearMonth ym = startYm;
        while (!ym.isAfter(endYm)) {
            activeYearMonths.add(ym.toString());  // Adds "2024-01", "2024-02", etc.
            ym = ym.plusMonths(1);
        }
    }

    // Iterate through period and generate billing ONLY for active year-months
    YearMonth startYm = YearMonth.from(startDate);
    YearMonth endYm = YearMonth.from(endDate);
    YearMonth ym = startYm;
    while (!ym.isAfter(endYm)) {
        String ymStr = ym.toString();  // "2024-01", "2024-02", etc.
        if (activeYearMonths.contains(ymStr)) {
            // Generate billing for this month
            LocalDate billingCollectDate = ym.atDay(collectDay);
            LocalDate billingDueDate = calculateDueDate(billingCollectDate, mBillings.getDueDateOffset());
            // ... create billing record
        }
        ym = ym.plusMonths(1);
    }
}
```

#### BillingServiceImpl.java
```java
// When generating billing for specific month
String yearMonthStr = String.format("%d-%02d", targetYear, targetMonth);  // e.g., "2024-01"
List<MBillingMonthlyActive> activeMonths = mBillingMonthlyActiveRepository
        .findByMBillingMonthlyAndMonthYear(mBilling, yearMonthStr);

if (activeMonths.isEmpty()) {
    throw new ApiException(..., "Bulan tidak aktif untuk '" + mBilling.getName() + "'");
}
```

#### BillingScholarshipServiceImpl.java & ScholarshipServiceImpl.java
```java
// Convert monthYear string to month number for validation
List<Integer> activeMonths = activeMonthsRecords.stream()
        .map(m -> {
            try {
                return java.time.YearMonth.parse(m.getMonthYear()).getMonthValue();
            } catch (Exception e) {
                return null;
            }
        })
        .filter(m -> m != null)
        .collect(Collectors.toList());

// Validate scholarship months against active months
List<Integer> invalidMonths = request.getMonths().stream()
        .filter(month -> !activeMonths.contains(month))
        .collect(Collectors.toList());
```

## API Usage Examples

### Create MBillings with Monthly Active
**POST /api/m-billings**

```json
{
  "name": "Billing Bulan Januari - Maret 2024",
  "amount": 500000,
  "billingType": "MONTHLY",
  "isAutoGenerate": true,
  "startDatePeriod": "2024-01-01",
  "endDatePeriod": "2024-03-31",
  "collectDate": 5,
  "dueDateOffset": 7,
  "monthlyActive": ["2024-01", "2024-02", "2024-03"],
  "billedUsers": ["user-uuid-1", "user-uuid-2"]
}
```

### Update Monthly Active
**PATCH /api/m-billings/{id}/monthly-active**

```json
{
  "mBillingsId": 123,
  "monthlyActive": ["2024-01", "2024-02", "2024-04"]  // Note: excluded "2024-03"
}
```

### Validation Rules
1. **Format**: Must be valid "yyyy-MM" format (e.g., "2024-01")
2. **Not Empty**: monthlyActive list tidak boleh kosong jika billing type MONTHLY
3. **Unique**: Tidak boleh ada duplikat month untuk satu MBillings
4. **Consistency**: Ketika auto-generate, hanya month yang ada di activeYearMonths yang akan generate billing

## Database Migration

### Schema Changes
```sql
-- Rename column from 'month' to 'month_year'
ALTER TABLE m_billing_monthly_active 
RENAME COLUMN month TO month_year;

-- Update data (example for 2024)
UPDATE m_billing_monthly_active 
SET month_year = CONCAT('2024-', LPAD(month_year, 2, '0'))
WHERE month_year REGEXP '^[0-9]+$';

-- Update unique constraint
ALTER TABLE m_billing_monthly_active 
DROP CONSTRAINT uk_mbilling_month,
ADD CONSTRAINT uk_mbilling_month_year UNIQUE (m_billing_id, month_year);
```

## Testing Checklist

### Unit Tests
- [ ] `validateMonthlyActive()` accepts valid "yyyy-MM" format
- [ ] `validateMonthlyActive()` rejects invalid formats
- [ ] `autoGenerateBillings()` generates billing only for active year-months
- [ ] `autoGenerateBillings()` generates all months if activeYearMonths is empty

### Integration Tests
- [ ] Create MBillings with monthlyActive as List<String>
- [ ] Update monthlyActive with new list
- [ ] Generate monthly billing respects activeYearMonths
- [ ] Scholarship validation uses extracted month numbers from monthYear

### Manual Tests
1. **Create with Multiple Months**
   - POST with monthlyActive = ["2024-01", "2024-02", "2024-03"]
   - Verify data saved with correct format

2. **Auto-Generate Respects Active Months**
   - Create MONTHLY billing with startDatePeriod = "2024-01-01", endDatePeriod = "2024-12-31"
   - Set monthlyActive = ["2024-01", "2024-03", "2024-06"]
   - Trigger auto-generate
   - Verify billing created only for Jan, Mar, Jun

3. **Empty Active Months Auto-Generate All**
   - Create MONTHLY billing with startDatePeriod = "2024-01-01", endDatePeriod = "2024-03-31"
   - Set monthlyActive = []
   - Trigger auto-generate
   - Verify billing created for Jan, Feb, Mar

## Rollback Plan

If issues occur:

1. Revert database column back to integer month:
```sql
ALTER TABLE m_billing_monthly_active 
RENAME COLUMN month_year TO month;
```

2. Update entity to use `Integer month` instead of `String monthYear`

3. Revert all service logic changes to use integer comparisons

## Known Limitations

- **Year-specific**: Format is "yyyy-MM", so months are now year-specific (2024-01 vs 2025-01)
- **Backward Compatibility**: Old API clients sending integer months (1-12) will fail validation
- **Database**: Requires column rename or migration script

## Future Enhancements

1. Add quarter-based billing (Q1, Q2, etc.)
2. Support custom date ranges per billing cycle
3. Bulk update of activeYearMonths with date range
4. Calendar UI for month selection
