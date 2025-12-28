# BroadcastRecipientQueryController API Documentation

## Endpoint: `/api/broadcast-recipients/query`

### List Broadcast Recipients with Dynamic Filters
- **GET** `/api/broadcast-recipients/query`
- **Query Params:**
  - `group` (string, optional)
  - `name` (string, optional)
  - `scheduleDate` (datetime, optional, ISO 8601)
  - `tanggal` (date, optional, ISO 8601)
  - `sendStatus` (string, optional)
  - `category` (string, optional)
- **Response:**
  - List of broadcast recipients matching the filters.

---

## Business Logic
- Supports dynamic filtering by group, name, schedule date, creation date, send status, and category.
- Uses repository to fetch all recipients, then applies filters in memory.
- Maps entities to response DTOs.
