# Scholarship Discount Refactoring Summary

## Overview
Nilai diskon dipindahkan dari entity **Scholarship** ke **BillingScholarship** untuk memberikan fleksibilitas dalam pengaturan diskon per billing assignment.

## Architectural Decision
**Rationale**: Discount configuration should be at assignment level (BillingScholarship), not master level (Scholarship)

**Benefits**:
1. Same scholarship can have different discount rates for different billings
2. More flexible discount management per billing assignment
3. Separates concerns: Scholarship = program definition, BillingScholarship = specific assignment with discount

## Changes Made

### 1. Entity Changes ✅

#### Scholarship.java (REMOVED)
```java
// REMOVED 3 fields:
@Column(name = "discount_type", length = 20, nullable = false)
private String discountType;

@Column(name = "discount_value", precision = 15, scale = 2, nullable = false)
private BigDecimal discountValue;

@Column(name = "max_discount_amount", precision = 15, scale = 2)
private BigDecimal maxDiscountAmount;
```

#### BillingScholarship.java (ADDED)
```java
// ADDED 3 fields:
@Column(name = "discount_type", length = 20, nullable = false)
private String discountType; // PERCENTAGE or FIXED_AMOUNT

@Column(name = "discount_value", precision = 15, scale = 2, nullable = false)
private BigDecimal discountValue;

@Column(name = "max_discount_amount", precision = 15, scale = 2)
private BigDecimal maxDiscountAmount;
```

### 2. DTO Changes ✅

#### ScholarshipRequest.java (REMOVED)
- Removed `discountType` field
- Removed `discountValue` field  
- Removed `maxDiscountAmount` field
- Removed `java.math.BigDecimal` import (unused)

#### BillingScholarshipRequest.java (ADDED)
```java
// ADDED discount validation
@NotBlank(message = "Tipe diskon wajib diisi (PERCENTAGE atau FIXED_AMOUNT)")
@Pattern(regexp = "^(PERCENTAGE|FIXED_AMOUNT)$", message = "Tipe diskon harus PERCENTAGE atau FIXED_AMOUNT")
@JsonProperty("discountType")
private String discountType;

@NotNull(message = "Nilai diskon wajib diisi")
@DecimalMin(value = "0.01", message = "Nilai diskon minimal 0.01")
@JsonProperty("discountValue")
private BigDecimal discountValue;

@JsonProperty("maxDiscountAmount")
private BigDecimal maxDiscountAmount;
```

### 3. Database Migration ✅

**File**: `V1.3__move_discount_to_billing_scholarship.sql`

**Operations**:
1. Add discount columns to `billing_scholarship` table
2. Add CHECK constraints for validation:
   - `discount_type` IN ('PERCENTAGE', 'FIXED_AMOUNT')
   - `discount_value` >= 0
   - `max_discount_amount` IS NULL OR >= 0
3. Migrate existing data from `m_scholarship` to `billing_scholarship`
4. Drop discount columns from `m_scholarship` table
5. Add column comments for documentation

## Remaining Tasks

### 4. Response DTO Changes ⚠️ PENDING
**File**: `BillingScholarshipResponse.java`
- [ ] Add `discountType` field
- [ ] Add `discountValue` field
- [ ] Add `maxDiscountAmount` field

### 5. Service Layer Changes ⚠️ PENDING
**File**: `ScholarshipServiceImpl.java`

**Method**: `assignToMBilling()` (lines 250-368)
- [ ] Remove discount field access from Scholarship
- [ ] Get discount values from BillingScholarshipRequest instead
- [ ] Pass discount values to BillingScholarship builder
- [ ] Remove discount validation from scholarship creation/update methods

### 6. Application Restart ⚠️ PENDING
- [ ] Kill process on port 8081
- [ ] Run `./mvnw spring-boot:run -DskipTests`
- [ ] Verify Hibernate creates new schema
- [ ] Check for migration errors

### 7. Integration Testing ⚠️ PENDING
- [ ] Test scholarship creation (without discount fields)
- [ ] Test scholarship assignment to billing (with discount fields)
- [ ] Test monthly scholarship allocation
- [ ] Verify subset validation still works

## Database Schema Changes

### billing_scholarship Table
```sql
ALTER TABLE billing_scholarship
    ADD COLUMN discount_type VARCHAR(20) NOT NULL DEFAULT 'PERCENTAGE',
    ADD COLUMN discount_value NUMERIC(15,2) NOT NULL DEFAULT 0,
    ADD COLUMN max_discount_amount NUMERIC(15,2);
```

### m_scholarship Table
```sql
ALTER TABLE m_scholarship
    DROP COLUMN discount_type,
    DROP COLUMN discount_value,
    DROP COLUMN max_discount_amount;
```

## API Usage Changes

### Before (OLD API)
```json
// POST /api/scholarships
{
  "name": "Beasiswa Prestasi",
  "description": "...",
  "discountType": "PERCENTAGE",
  "discountValue": 50,
  "maxDiscountAmount": 500000,
  "startDate": "2025-01-01",
  "endDate": "2025-12-31"
}
```

### After (NEW API)
```json
// POST /api/scholarships (no discount)
{
  "name": "Beasiswa Prestasi",
  "description": "...",
  "startDate": "2025-01-01",
  "endDate": "2025-12-31"
}

// POST /api/billing-scholarships (with discount)
{
  "scholarshipId": 1,
  "mBillingId": 1,
  "months": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
  "discountType": "PERCENTAGE",
  "discountValue": 50,
  "maxDiscountAmount": 500000,
  "notes": "Diskon 50% untuk tagihan bulanan"
}
```

## Compilation Status
- ✅ BUILD SUCCESS
- ⚠️ Lombok warnings (expected - Lombok generates accessors)
- ⚠️ Some IDE errors (Lombok processor issues - compile with Maven works)

## Next Steps Priority
1. **HIGH**: Update BillingScholarshipResponse DTO (add discount fields)
2. **HIGH**: Update ScholarshipServiceImpl.assignToMBilling() - use new discount location
3. **HIGH**: Restart application - apply changes
4. **MEDIUM**: Testing - verify functionality

## Breaking Changes
⚠️ **API Breaking Change**: Clients creating scholarships must now:
1. Create scholarship WITHOUT discount (POST /api/scholarships)
2. Assign to billing WITH discount (POST /api/billing-scholarships)

## Migration Notes
- Existing data will be migrated automatically by V1.3 migration
- No manual data migration required
- Application must be restarted after pulling these changes
