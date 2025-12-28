# JurnalController API Documentation

## Endpoint: `/api/jurnals`

### List Transaction Journals (Paginated)
- **GET** `/api/jurnals`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: timestampTxn)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `search` (string, optional)
  - `transactionType` (string, optional)
  - `systemSource` (string, optional)
  - `status` (string, optional)
  - `from` (datetime, optional, ISO 8601)
  - `to` (datetime, optional, ISO 8601)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Paginated list of transaction journals in various formats.

---

## Business Logic
- Supports paginated, filtered, and formatted listing of transaction journals.
- Filters by search, transaction type, system source, status, and date range.
- Uses service layer for all business logic and data access.
- Returns error for invalid date formats.
