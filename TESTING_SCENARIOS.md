## Monthly Active Year-Month Format - Testing Guide

### Test Scenario 1: Create MBillings with Monthly Type and Auto-Generate

**Description**: Create a monthly billing for January-March 2024 with only January and March active, verify that auto-generation creates billings ONLY for those months.

**Setup**:
1. Create a Yayasan (ID: 1)
2. Create an Institution (ID: 1) under Yayasan
3. Create Academic Year (ID: 1) with 2024 dates

**Request**: POST /api/m-billings
```json
{
  "name": "Biaya Uji Coba Semester 1 2024",
  "description": "Billing untuk bulan Januari dan Maret saja",
  "amount": 750000,
  "billingType": "MONTHLY",
  "isAutoGenerate": true,
  "startDatePeriod": "2024-01-01",
  "endDatePeriod": "2024-03-31",
  "academicYearId": 1,
  "collectDate": 10,
  "dueDateOffset": 7,
  "monthlyActive": ["2024-01", "2024-03"],
  "billedUsers": ["user-uuid-1", "user-uuid-2"],
  "institutionId": 1
}
```

**Expected Response**: 
```json
{
  "id": 123,
  "uuid": "...",
  "name": "Biaya Uji Coba Semester 1 2024",
  "amount": 750000,
  "billingType": "MONTHLY",
  "monthlyActive": ["2024-01", "2024-03"],
  "isAutoGenerate": true,
  "startDatePeriod": "2024-01-01",
  "endDatePeriod": "2024-03-31"
}
```

**Expected Behavior**:
- ✅ monthlyActive saved as ["2024-01", "2024-03"]
- ✅ Auto-generation creates exactly 2 Billing records:
  - Billing 1: name = "Biaya Uji Coba Semester 1 2024 - January 2024", billingCollectDate = "2024-01-10"
  - Billing 2: name = "Biaya Uji Coba Semester 1 2024 - March 2024", billingCollectDate = "2024-03-10"
- ✅ No billing created for February 2024

**Validation Steps**:
```java
// In database, check:
SELECT * FROM m_billing_monthly_active WHERE m_billing_id = 123;
// Expected: 2 rows with monthYear = "2024-01" and "2024-03"

SELECT * FROM billing WHERE m_billing_id = 123;
// Expected: 2 rows with billing_collect_date = "2024-01-10" and "2024-03-10"
```

---

### Test Scenario 2: Create with Empty Monthly Active (Generate All Months)

**Description**: Create monthly billing without specifying monthlyActive, auto-generation should create billing for all months in date range.

**Request**: POST /api/m-billings
```json
{
  "name": "Biaya Operasional Q1 2024",
  "amount": 500000,
  "billingType": "MONTHLY",
  "isAutoGenerate": true,
  "startDatePeriod": "2024-01-01",
  "endDatePeriod": "2024-03-31",
  "collectDate": 5,
  "dueDateOffset": 3,
  "monthlyActive": [],
  "billedUsers": ["user-uuid-1"]
}
```

**Expected Behavior**:
- ✅ Auto-generation creates 3 Billing records (Jan, Feb, Mar)
- ✅ monthlyActive in response shows ["2024-01", "2024-02", "2024-03"]

**Database Check**:
```sql
SELECT COUNT(*) FROM billing WHERE m_billing_id = 124;
-- Expected: 3 rows
```

---

### Test Scenario 3: Update Monthly Active

**Description**: Update an existing MBillings to change active months.

**Request**: PATCH /api/m-billings/123/monthly-active
```json
{
  "mBillingsId": 123,
  "monthlyActive": ["2024-01", "2024-02"]  // Changed from ["2024-01", "2024-03"]
}
```

**Expected Response**:
```json
{
  "id": 123,
  "monthlyActive": ["2024-01", "2024-02"]
}
```

**Expected Behavior**:
- ✅ Old MBillingMonthlyActive records for "2024-03" are deleted
- ✅ New record for "2024-02" is created
- ✅ "2024-01" record remains

**Database Check**:
```sql
SELECT monthYear FROM m_billing_monthly_active WHERE m_billing_id = 123;
-- Expected: "2024-01", "2024-02" (no "2024-03")
```

---

### Test Scenario 4: Validation - Invalid Format

**Description**: Attempt to create billing with invalid month format.

**Request**: POST /api/m-billings
```json
{
  "name": "Invalid Billing",
  "amount": 500000,
  "billingType": "MONTHLY",
  "monthlyActive": ["01", "02", "2024-13", "invalid"]
}
```

**Expected Response**: 400 Bad Request
```json
{
  "errorCode": "BUSINESS_RULE_VIOLATION",
  "message": "Format bulan harus yyyy-MM"
}
```

---

### Test Scenario 5: Validation - Empty Monthly Active for MONTHLY Type

**Description**: Create MONTHLY billing without providing monthlyActive (and isAutoGenerate = false).

**Request**: POST /api/m-billings
```json
{
  "name": "Manual Billing",
  "amount": 500000,
  "billingType": "MONTHLY",
  "isAutoGenerate": false,
  "monthlyActive": null
}
```

**Expected Response**: 400 Bad Request
```json
{
  "errorCode": "BUSINESS_RULE_VIOLATION",
  "message": "Bulan aktif harus diisi"
}
```

---

### Test Scenario 6: Generate Specific Month Billing

**Description**: Generate billing for a specific month using generateMonthlyBilling endpoint.

**Request**: POST /api/m-billings/123/generate-monthly
```json
{
  "mBillingsId": 123,
  "year": 2024,
  "month": 1
}
```

**Service Flow**:
1. Convert year/month to "2024-01"
2. Query MBillingMonthlyActive with monthYear = "2024-01"
3. If empty, throw error "Bulan 01 tidak aktif untuk 'Biaya Uji Coba Semester 1 2024'"
4. If exists, create billing record

**Expected Response**: 200 OK
```json
{
  "id": 456,
  "name": "Biaya Uji Coba Semester 1 2024 - Bulan Januari 2024",
  "billingCollectDate": "2024-01-10"
}
```

**Expected Behavior** (if month is NOT active):
```json
{
  "errorCode": "STATE_CONFLICT",
  "message": "Bulan 01 tidak aktif untuk 'Biaya Uji Coba Semester 1 2024'"
}
```

---

### Test Scenario 7: Scholarship with Selective Months

**Description**: Create scholarship that applies only to specific months of a monthly billing.

**Request**: POST /api/billing-scholarships
```json
{
  "name": "Beasiswa Bulan Ganjil",
  "discountValue": 100000,
  "maxDiscountAmount": 100000,
  "billingScholarshipType": "PERCENTAGE",
  "mBillingId": 123,
  "months": [1, 3],  // January and March only (matches active "2024-01", "2024-03")
  "students": ["student-uuid-1", "student-uuid-2"]
}
```

**Service Flow**:
1. Fetch MBillingMonthlyActive for MBilling 123
2. Extract month values: convert "2024-01" → 1, "2024-03" → 3
3. Validate requested months [1, 3] are subset of active [1, 3] ✅
4. Create scholarship and monthly records

**Expected Response**: 201 Created
```json
{
  "id": 789,
  "name": "Beasiswa Bulan Ganjil",
  "months": [1, 3]
}
```

**Error Case** (requesting month not in activeYearMonths):
```json
{
  "errorCode": "BUSINESS_RULE_VIOLATION",
  "message": "Bulan tidak valid: [2]. Bulan yang tersedia: [1, 3]"
}
```

---

### Test Scenario 8: Year Boundary - December to January

**Description**: Create billing spanning December 2023 to January 2024.

**Request**: POST /api/m-billings
```json
{
  "name": "Biaya Transisi Tahun",
  "amount": 600000,
  "billingType": "MONTHLY",
  "isAutoGenerate": true,
  "startDatePeriod": "2023-12-01",
  "endDatePeriod": "2024-01-31",
  "monthlyActive": ["2023-12", "2024-01"],
  "collectDate": 15,
  "dueDateOffset": 5
}
```

**Expected Behavior**:
- ✅ Two billings created:
  - "Biaya Transisi Tahun - December 2023" (2023-12-15)
  - "Biaya Transisi Tahun - January 2024" (2024-01-15)
- ✅ monthYear in database correctly distinguishes year: "2023-12" vs "2024-01"

---

### Manual Testing Checklist

- [ ] **Field Type Validation**: monthlyActive is List<String>, not List<Integer>
  - POST request with monthlyActive as integer array should fail
  - POST request with monthlyActive as string array should succeed

- [ ] **Format Validation**: YearMonth.parse() enforces "yyyy-MM"
  - Valid: "2024-01", "2023-12", "2025-06"
  - Invalid: "01", "2024-1", "2024/01", "12/2024"

- [ ] **Auto-Generation Logic**: 
  - activeYearMonths correctly populated from monthYear field
  - Iteration uses YearMonth object, not integer
  - Only months in activeYearMonths list generate billing

- [ ] **Month Extraction for Scholarships**:
  - YearMonth.parse(monthYear).getMonthValue() returns correct 1-12
  - Validation compares extracted month number against requested months

- [ ] **Database Integrity**:
  - m_billing_monthly_active.month_year contains valid "yyyy-MM" strings
  - Unique constraint on (m_billing_id, month_year) prevents duplicates
  - Ordering by month_year produces chronological results

---

### Performance Tests (Optional)

**Test**: Create billing with 12 months, verify auto-generation completes quickly

```java
// Time this operation
POST /api/m-billings with monthlyActive of all 12 months of 2024
Expected: Completes in < 2 seconds
```

**Test**: Query active months for large MBilling IDs

```sql
SELECT COUNT(*) FROM m_billing_monthly_active 
WHERE m_billing_id IN (SELECT id FROM m_billings LIMIT 1000);
-- Should use index on m_billing_id for fast lookup
```

---

### Troubleshooting Guide

**Issue**: "Format bulan harus yyyy-MM"
- **Cause**: monthlyActive contains invalid format
- **Fix**: Ensure format is exactly "yyyy-MM" (e.g., "2024-01" not "2024-1" or "01")

**Issue**: "Method findByMBillingMonthlyAndMonth not found"
- **Cause**: Code still using old method name with Integer parameter
- **Fix**: Use findByMBillingMonthlyAndMonthYear with String parameter

**Issue**: Auto-generate creates billing for all months despite monthlyActive restriction
- **Cause**: activeYearMonths list not populated or condition not checked
- **Fix**: Verify autoGenerateBillings checks `activeYearMonths.contains(ymStr)`

**Issue**: Scholarship month validation fails unexpectedly
- **Cause**: getMonth() method called on MBillingMonthlyActive (doesn't exist)
- **Fix**: Use YearMonth.parse(monthYear).getMonthValue() to extract month

**Issue**: Duplicate billing created for same year-month
- **Cause**: Unique constraint on (m_billing_id, month_year) not enforced
- **Fix**: Verify database schema has correct unique constraint
