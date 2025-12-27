# Withdrawal & Transit API

> **API Version:** v1  
> **Last Updated:** December 24, 2025  
> **Base URL:** `/api/withdrawals`

---

## Overview

The Withdrawal & Transit API provides endpoints for managing institution-level financial transfers:

1. **Withdrawal** - Transfer funds from BILLING account to external (e.g., to bank)
2. **Transit** - Internal transfer from BILLING account to BILLING_CASH account (for cash management)
3. **History** - View all withdrawal and transit transactions with pagination

Both operations:
- Create invoice entries for audit trail
- Record double-entry journal transactions
- Update account balances atomically
- Require valid authentication and authorization

---

## Authentication

All endpoints require Bearer token authentication with the user bound to a Yayasan and Institution.

```http
Authorization: Bearer {token}
```

The authenticated user's institution determines which accounts are accessible.

---

## Endpoints

### 1. Request OTP for Withdrawal

Request an OTP code (sent via WhatsApp) before performing withdrawal or transit operations.

**Endpoint:**
```
POST /api/withdrawals/otp/request
GET /api/withdrawals/otp/request
```

**Authentication:** Required (Bearer token)

**Request Body:** None

**Response:**
```typescript
{
  success: boolean;
  message: string;  // "OTP sent" or "Failed to send OTP"
  timestamp: string;
  data: string;
}
```

**Status Codes:**
| Code | Message | Cause |
|------|---------|-------|
| 200 | OTP sent | Successfully sent OTP to user's telephone |
| 401 | Authentication required | No valid Bearer token |
| 400 | No telephone bound to user | User account has no phone number |

**Example:**
```bash
curl -X POST http://localhost:8081/api/withdrawals/otp/request \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

### 2. Withdraw from BILLING Account

Perform a withdrawal from the institution's BILLING account to external destination (e.g., bank transfer).

**Endpoint:**
```
POST /api/withdrawals/billing
```

**Authentication:** Required (Bearer token)

**Request Body:**
```typescript
{
  otp_code: string;        // OTP from /otp/request endpoint
  amount: number;          // Amount in rupiah (must be positive)
}
```

**Validation Rules:**
- `otp_code`: Required, must match a valid/non-expired OTP for user's telephone
- `amount`: Required, must be positive number, must not exceed available balance in BILLING account

**Response:**
```typescript
{
  success: boolean;
  message: string;  // "Withdrawal processed"
  timestamp: string;
  data: {
    id: number;
    uuid: string;
    yayasanId: number;
    institutionId: number;
    invoiceNumber: string;
    timestampTxn: string;  // ISO datetime
    debitAccountId: number;  // BILLING account
    creditAccountId: null;   // External withdrawal has no credit account
    amount: number;
    feeAmount: number;
    providerFee: number;
    feeGlAccountId: null;
    debitStartBalance: number;  // BILLING balance before
    debitEndBalance: number;    // BILLING balance after
    creditStartBalance: null;
    creditEndBalance: null;
    transactionType: "WITHDRAWAL";
    description: string;  // "Institution withdrawal"
    status: "SUCCESS";
    systemSource: "CORE";
    userInitiatorId: number;
    createdAt: string;
    updatedAt: string;
  }
}
```

**Status Codes:**
| Code | Error Code | Message | Cause |
|------|-----------|---------|-------|
| 200 | - | Withdrawal processed | Success |
| 400 | INVALID_INPUT | Amount must be positive | Invalid amount |
| 400 | INVALID_INPUT | Invalid or expired OTP | OTP verification failed |
| 400 | INVALID_INPUT | No telephone bound to user | User has no phone |
| 400 | INSUFFICIENT_BALANCE | Insufficient available balance | Not enough funds |
| 401 | AUTH_FAILED | Authentication required | Invalid/missing token |
| 404 | RESOURCE_NOT_FOUND | Institution BILLING account not found | Account doesn't exist |

**Example:**
```bash
# Step 1: Request OTP
curl -X POST http://localhost:8081/api/withdrawals/otp/request \
  -H "Authorization: Bearer YOUR_TOKEN"

# Step 2: Perform withdrawal
curl -X POST http://localhost:8081/api/withdrawals/billing \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "otp_code": "123456",
    "amount": 5000000
  }'
```

---

### 3. Transit from BILLING to BILLING_CASH

Transfer funds internally from BILLING account to BILLING_CASH account. This operation records double-entry journal entries and creates an invoice.

**Endpoint:**
```
POST /api/withdrawals/transit
```

**Authentication:** Required (Bearer token)

**Request Body:**
```typescript
{
  amount: number;              // Required: transfer amount in rupiah
  description?: string;        // Optional: transaction description
  referenceNumber?: string;    // Optional: external reference for tracking
}
```

**Validation Rules:**
- `amount`: Required, must be positive, must not exceed BILLING account available balance
- `description`: Optional, max 255 characters
- `referenceNumber`: Optional, max 50 characters

**Response:**
```typescript
{
  success: boolean;
  message: string;  // "Transit from BILLING to BILLING_CASH processed"
  timestamp: string;
  data: {
    id: number;
    uuid: string;
    yayasanId: number;
    institutionId: number;
    referenceNumber?: string;
    invoiceNumber: string;           // INV-TRN-{timestamp}-{random}
    timestampTxn: string;            // ISO datetime
    debitAccountId: number;          // BILLING account (money out)
    creditAccountId: number;         // BILLING_CASH account (money in)
    amount: number;
    feeAmount: 0;
    providerFee: 0;
    feeGlAccountId: null;
    debitStartBalance: number;       // BILLING balance before
    debitEndBalance: number;         // BILLING balance after
    creditStartBalance: number;      // BILLING_CASH balance before
    creditEndBalance: number;        // BILLING_CASH balance after
    transactionType: "TRANSIT";
    description: string;
    status: "SUCCESS";
    systemSource: "CORE";
    userInitiatorId: number;
    createdAt: string;
    updatedAt: string;
  }
}
```

**Status Codes:**
| Code | Error Code | Message | Cause |
|------|-----------|---------|-------|
| 200 | - | Transit processed | Success |
| 400 | INVALID_INPUT | Amount must be positive | Invalid amount |
| 400 | INVALID_INPUT | User must be bound to yayasan and institution | User context missing |
| 400 | INSUFFICIENT_BALANCE | Insufficient available balance in BILLING account | Not enough funds |
| 401 | AUTH_FAILED | Authentication required | Invalid/missing token |
| 404 | RESOURCE_NOT_FOUND | Institution BILLING account not found | BILLING account missing |
| 404 | RESOURCE_NOT_FOUND | Institution BILLING_CASH account not found | BILLING_CASH account missing |

**Transit Process Flow:**

```
User Request (amount: 2,500,000)
    ↓
Validate Authentication & User Context
    ↓
Find BILLING Account (type: BILLING, level: INSTITUTION)
    ↓
Find BILLING_CASH Account (type: BILLING_CASH, level: INSTITUTION)
    ↓
Check BILLING Account Balance (available ≥ amount)
    ↓
Update Balances (both accounts atomically):
  - Debit BILLING: -2,500,000
  - Credit BILLING_CASH: +2,500,000
    ↓
Generate Invoice (INV-TRN-{timestamp}-{random})
    ↓
Create TransactionJournal Entry:
  - Debit: BILLING (2,500,000)
  - Credit: BILLING_CASH (2,500,000)
  - Status: SUCCESS
  - Type: TRANSIT
    ↓
Link Invoice to Journal
    ↓
Log Operation & Return Response
```

**Example:**
```bash
curl -X POST http://localhost:8081/api/withdrawals/transit \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 2500000,
    "description": "Daily cash transfer to BILLING_CASH",
    "referenceNumber": "TRX-20250115-001"
  }'
```

---

### 4. Get Withdrawal & Transit History

Retrieve paginated history of all withdrawal and transit transactions.

**Endpoint:**
```
GET /api/withdrawals/history
```

**Authentication:** Required (Bearer token)

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 0 | Page number (0-indexed) |
| size | integer | 10 | Items per page |
| sortBy | string | "id" | Field to sort by (id, timestampTxn, createdAt, amount) |
| sortDirection | string | "DESC" | Sort direction (ASC or DESC) |
| draw | integer | - | jQuery DataTable draw counter (for jquery-datatable format) |
| format | string | "standard" | Response format: standard, jquery-datatable, ant-table |

**Response - Standard Format:**
```typescript
{
  success: boolean;
  message: string;
  timestamp: string;
  data: {
    data: TransactionJournal[];
    total: number;             // Total records
    page: number;              // Current page (0-indexed)
    size: number;              // Page size
    totalPages: number;        // Total number of pages
    hasNext: boolean;
    hasPrevious: boolean;
  }
}
```

**Response - jQuery DataTable Format:**
```typescript
{
  draw: number;                // From request
  recordsTotal: number;        // Total records
  recordsFiltered: number;     // Filtered records
  data: TransactionJournal[];
}
```

**Response - Ant Table Format:**
```typescript
{
  data: TransactionJournal[];
  success: boolean;
  total: number;
  current: number;             // Current page (1-indexed)
  pageSize: number;
}
```

**TransactionJournal Item:**
```typescript
{
  id: number;
  uuid: string;
  yayasanId: number;
  institutionId: number;
  referenceNumber?: string;
  invoiceNumber: string;
  timestampTxn: string;
  debitAccountId: number;
  creditAccountId?: number;
  amount: number;
  feeAmount: number;
  providerFee: number;
  feeGlAccountId?: number;
  debitStartBalance: number;
  debitEndBalance: number;
  creditStartBalance?: number;
  creditEndBalance?: number;
  transactionType: "WITHDRAWAL" | "TRANSIT";
  description: string;
  status: "SUCCESS" | "FAILED";
  systemSource: "CORE";
  userInitiatorId: number;
  createdAt: string;
  updatedAt: string;
}
```

**Status Codes:**
| Code | Message | Cause |
|------|---------|-------|
| 200 | Success | Retrieved history |
| 401 | Authentication required | Invalid/missing token |

**Examples:**

```bash
# Get first 10 records (default)
curl http://localhost:8081/api/withdrawals/history \
  -H "Authorization: Bearer YOUR_TOKEN"

# Get page 2 with 25 records per page
curl "http://localhost:8081/api/withdrawals/history?page=1&size=25" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Sort by transaction time (newest first)
curl "http://localhost:8081/api/withdrawals/history?sortBy=timestampTxn&sortDirection=DESC" \
  -H "Authorization: Bearer YOUR_TOKEN"

# jQuery DataTable format
curl "http://localhost:8081/api/withdrawals/history?format=jquery-datatable&draw=1" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Ant Design Table format
curl "http://localhost:8081/api/withdrawals/history?format=ant-table&page=0&size=20" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## Data Models

### TransitBillingRequest
```typescript
interface TransitBillingRequest {
  /** Transfer amount in rupiah (must be positive) */
  amount: number;
  
  /** Optional: Transaction description */
  description?: string;
  
  /** Optional: External reference number for tracking */
  referenceNumber?: string;
}
```

### WithdrawBillingRequest
```typescript
interface WithdrawBillingRequest {
  /** OTP code from /otp/request endpoint */
  otp_code: string;
  
  /** Withdrawal amount in rupiah (must be positive) */
  amount: number;
}
```

### TransactionJournal
```typescript
interface TransactionJournal {
  /** Primary key */
  id: number;
  
  /** Unique identifier (UUID v4) */
  uuid: string;
  
  /** Yayasan (foundation) ID */
  yayasanId: number;
  
  /** Institution ID */
  institutionId: number;
  
  /** Optional external reference number */
  referenceNumber?: string;
  
  /** Generated invoice number for audit trail */
  invoiceNumber: string;
  
  /** Transaction timestamp (ISO format) */
  timestampTxn: string;
  
  /** Debit account ID (money out) */
  debitAccountId: number;
  
  /** Credit account ID (money in, nullable for external withdrawals) */
  creditAccountId?: number;
  
  /** Transaction amount in rupiah */
  amount: number;
  
  /** Fee amount deducted (if any) */
  feeAmount: number;
  
  /** Provider fee (gateway charge) */
  providerFee: number;
  
  /** GL account ID for fees (nullable) */
  feeGlAccountId?: number;
  
  /** Starting balance in debit account */
  debitStartBalance: number;
  
  /** Ending balance in debit account */
  debitEndBalance: number;
  
  /** Starting balance in credit account */
  creditStartBalance?: number;
  
  /** Ending balance in credit account */
  creditEndBalance?: number;
  
  /** Transaction type: "WITHDRAWAL" or "TRANSIT" */
  transactionType: "WITHDRAWAL" | "TRANSIT";
  
  /** Transaction description */
  description: string;
  
  /** Status: "SUCCESS" or "FAILED" */
  status: "SUCCESS" | "FAILED";
  
  /** System source: "CORE" */
  systemSource: "CORE";
  
  /** User ID who initiated transaction */
  userInitiatorId: number;
  
  /** Creation timestamp */
  createdAt: string;
  
  /** Last update timestamp */
  updatedAt: string;
}
```

---

## Business Logic

### Account Types
- **BILLING** - Institution's main billing/revenue account
- **BILLING_CASH** - Institution's cash holding account

Both must exist at INSTITUTION level for transit operations.

### Double-Entry Bookkeeping
Transit operations follow accounting principles:
```
Debit:   BILLING Account          (e.g., 2,500,000)
Credit:  BILLING_CASH Account     (e.g., 2,500,000)
```

Both debit and credit accounts must have sufficient balance/limits.

### Balance Updates
- **Available Balance**: Day-to-day spendable amount
- **Ledger Balance**: Total account balance including pending transactions
- Both updated atomically in single transaction

### Invoice Generation
- Auto-generated: `INV-TRN-{YYYYMMDDHHMMSS}-{RANDOM:1000-9999}`
- Status: Immediately set to "PAID"
- Linked to TransactionJournal for audit trail
- Non-editable once created

### Transactional Integrity
All operations are atomic - either fully succeed or fully rollback. No partial transactions.

---

## Error Codes

| Code | HTTP | Message | Resolution |
|------|------|---------|-----------|
| AUTH_FAILED | 401 | Authentication required | Provide valid Bearer token |
| INVALID_INPUT | 400 | Amount must be positive | Ensure amount > 0 |
| INVALID_INPUT | 400 | Invalid or expired OTP | Request new OTP and try again |
| INVALID_INPUT | 400 | No telephone bound to user | Update user profile with phone |
| INVALID_INPUT | 400 | User must be bound to yayasan and institution | Check user's organization binding |
| INSUFFICIENT_BALANCE | 400 | Insufficient available balance | Check account balance and reduce amount |
| RESOURCE_NOT_FOUND | 404 | Institution BILLING account not found | Ensure account setup in Chart of Accounts |
| RESOURCE_NOT_FOUND | 404 | Institution BILLING_CASH account not found | Create BILLING_CASH account |
| RESOURCE_NOT_FOUND | 404 | Account balance not found | Initialize account balance record |

---

## Use Cases

### Use Case 1: Daily Cash Collection
Transfer accumulated billing payments to cash account:
```bash
POST /api/withdrawals/transit
{
  "amount": 50000000,
  "description": "Daily billing to cash transfer",
  "referenceNumber": "DLY-20250115"
}
```

### Use Case 2: Cash Withdrawal to Bank
Request OTP, then withdraw to bank account:
```bash
# 1. Get OTP
POST /api/withdrawals/otp/request

# 2. Withdraw
POST /api/withdrawals/billing
{
  "otp_code": "123456",
  "amount": 100000000
}
```

### Use Case 3: End-of-Month Reconciliation
Report all transfers for the month:
```bash
GET /api/withdrawals/history?page=0&size=100&sortBy=timestampTxn&sortDirection=DESC
```

---

## Integration Notes

### With Chart of Accounts
- Both BILLING and BILLING_CASH accounts must be defined in AccountController
- Account Type enum must include both types
- Account Level must be INSTITUTION

### With Invoice System
- Transit and withdrawal generate invoices automatically
- Invoices link back to TransactionJournal for audit
- Invoice status immediately set to PAID

### With Audit Log
- All operations logged with:
  - User initiator ID
  - Timestamp
  - Account IDs
  - Amount and balances
  - Reference number (if provided)

### Frontend Integration

**React Example:**
```typescript
// Request OTP
const requestOtp = async () => {
  const response = await fetch('/api/withdrawals/otp/request', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}` }
  });
  return response.json();
};

// Perform transit
const transitBilling = async (amount: number, description: string) => {
  const response = await fetch('/api/withdrawals/transit', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ amount, description })
  });
  return response.json();
};

// Get history
const getHistory = async (page = 0, size = 10) => {
  const response = await fetch(
    `/api/withdrawals/history?page=${page}&size=${size}&format=standard`,
    { headers: { 'Authorization': `Bearer ${token}` } }
  );
  return response.json();
};
```

---

## Testing with HTTP Client

Use the provided `WithdrawalController.http` file in VS Code REST Client extension:

```http
### Request OTP
POST http://localhost:8081/api/withdrawals/otp/request
Authorization: Bearer YOUR_TOKEN

### Transit
POST http://localhost:8081/api/withdrawals/transit
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json

{
  "amount": 2500000,
  "description": "Test transfer"
}

### Get History
GET http://localhost:8081/api/withdrawals/history?page=0&size=10
Authorization: Bearer YOUR_TOKEN
```

---

## Related APIs

- **[Account API](14-account-api.md)** - Manage Chart of Accounts
- **[Billing API](10-billing-api.md)** - Billing management
- **[User Billing API](11-user-billing-api.md)** - Student billing records
- **[Transaction Journal API](13-transaction-journal-api.md)** - View all journal entries
- **[Transaction Biller API](12-transaction-biller-api.md)** - Payment processing

---

**Last Updated:** December 24, 2025  
**API Version:** v1  
**Status:** Production Ready
