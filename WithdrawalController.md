# WithdrawalController API Documentation

## Base Path
`/api/withdrawals`

---

## Endpoints

### 1. Request OTP for Withdrawal
- **URL:** `POST /api/withdrawals/otp/request` and `GET /api/withdrawals/otp/request`
- **Responses:**
  - Success message (OTP sent to user's telephone via WhatsApp)
- **Business Logic:**
  - Authenticates user, checks telephone binding
  - Sends OTP via WhatsApp using `OtpCodeService`

---

### 2. Withdraw Billing Cash
- **URL:** `POST /api/withdrawals/cash`
- **Request Body:**
  - `WithdrawBillingRequest`: amount, otp_code
- **Responses:**
  - Transaction journal response (withdrawal details)
- **Business Logic:**
  - Authenticates user, validates OTP
  - Checks yayasan and institution binding
  - Finds institution's BILLING_CASH account
  - Processes withdrawal via `WithdrawalService`

---

### 3. Transit Billing to Billing Cash
- **URL:** `POST /api/withdrawals/transit`
- **Request Body:**
  - `TransitBillingRequest`: amount, description/reference (optional)
- **Responses:**
  - Transaction journal response (transfer details)
- **Business Logic:**
  - Authenticates user
  - Transfers funds from BILLING to BILLING_CASH at institution level
  - Records double-entry journal, generates invoice, updates balances

---

### 4. Get Withdrawal and Transit History
- **URL:** `GET /api/withdrawals/history`
- **Query Parameters:**
  - `page` (int, default: 0): Page number
  - `size` (int, default: 10): Page size
  - `sortBy` (string, default: "id"): Field to sort by
  - `sortDirection` (string, default: "DESC"): Sort direction (ASC/DESC)
  - `draw` (int, optional): For DataTables integration
  - `format` (string, default: "standard"): Response format
- **Responses:**
  - Paginated transaction journal responses (withdrawal and transit history)
- **Business Logic:**
  - Retrieves paginated, sorted withdrawal and transit history via `WithdrawalService`

---

## Data Models
- `WithdrawBillingRequest`: Input for withdrawal (amount, otp_code)
- `TransitBillingRequest`: Input for transit (amount, description/reference)
- `TransactionJournalResponse`: Output for transaction details

## Notes
- All endpoints return standardized API responses.
- Business logic is delegated to `WithdrawalService` and `OtpCodeService`.
- Validation and authentication are enforced.
