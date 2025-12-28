# AccountBalanceController API Documentation

## Endpoint: `/api/account-balance`

### Get Balance by Account Number (Institution)
- **GET** `/api/account-balance/institution/by-account-number`
- **Query Params:**
  - `accountNumber` (string, required)
- **Auth:** Required
- **Response:**
  - Account balance for institution by account number.

### Get My Balances
- **GET** `/api/account-balance/me`
- **Auth:** Required
- **Response:**
  - List of balances for current user.

### Get Institution Balances
- **GET** `/api/account-balance/institution`
- **Query Params:**
  - `accountType` (string, optional)
- **Auth:** Required
- **Response:**
  - List of balances for institution, optionally filtered by account type.

### Get Single Institution Balance
- **GET** `/api/account-balance/institution/one`
- **Query Params:**
  - `accountType` (string, required)
  - `accountLevel` (string, required)
- **Auth:** Required
- **Response:**
  - Single balance for institution by type and level.

### Get Balance by Account Number
- **GET** `/api/account-balance/by-account-number`
- **Query Params:**
  - `accountNumber` (string, required)
- **Response:**
  - Account balance by account number.

---

## Business Logic
- Authenticated endpoints using user details.
- Fetches balances for user or institution.
- Handles filtering by account type and level.
- Throws appropriate errors for unauthorized/forbidden access and invalid input.
- Uses service layer for all business logic and data access.
