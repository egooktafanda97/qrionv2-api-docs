# BroadcastRecipientController API Documentation

## Endpoint: `/api/broadcast-recipients`

### Bulk Send Now (Student IDs)
- **POST** `/api/broadcast-recipients/bulk-id/send-now`
- **Body:** `BulkStudentIdRequest`
- **Response:**
  - Success message for immediate bulk send (student info).

### Bulk Schedule Send (Student IDs)
- **POST** `/api/broadcast-recipients/bulk-id/schedule`
- **Body:** `BulkStudentIdRequest`
- **Query Params:**
  - `scheduleDate` (string, required)
- **Response:**
  - Success message for scheduled bulk send (student info).

### Bulk Monthly Send (Student IDs)
- **POST** `/api/broadcast-recipients/bulk-id/monthly`
- **Body:** `BulkStudentIdRequest`
- **Query Params:**
  - `dayOfMonth` (int, required)
- **Response:**
  - Success message for monthly scheduled bulk send (student info).

### Bulk Send Now
- **POST** `/api/broadcast-recipients/bulk/send-now`
- **Body:** `BulkBroadcastRecipientRequest`
- **Response:**
  - Success message for immediate bulk send.

### Bulk Schedule Send
- **POST** `/api/broadcast-recipients/bulk/schedule`
- **Body:** `BulkBroadcastRecipientRequest`
- **Query Params:**
  - `scheduleDate` (string, required)
- **Response:**
  - Success message for scheduled bulk send.

### Bulk Monthly Send
- **POST** `/api/broadcast-recipients/bulk/monthly`
- **Body:** `BulkBroadcastRecipientRequest`
- **Query Params:**
  - `dayOfMonth` (int, required)
- **Response:**
  - Success message for monthly scheduled bulk send.

### Send Now
- **POST** `/api/broadcast-recipients/send-now`
- **Body:** `BroadcastRecipientRequest`
- **Response:**
  - Success message for immediate send.

### Schedule Send
- **POST** `/api/broadcast-recipients/schedule`
- **Body:** `BroadcastRecipientRequest`
- **Query Params:**
  - `scheduleDate` (string, required)
- **Response:**
  - Success message for scheduled send.

### Monthly Send
- **POST** `/api/broadcast-recipients/monthly`
- **Body:** `BroadcastRecipientRequest`
- **Query Params:**
  - `dayOfMonth` (int, required)
- **Response:**
  - Success message for monthly scheduled send.

### List Recipients
- **GET** `/api/broadcast-recipients`
- **Query Params:**
  - `group` (string, optional)
- **Response:**
  - List of broadcast recipients, optionally grouped.

---

## Business Logic
- Handles bulk and individual sending of broadcasts (immediate, scheduled, monthly).
- Supports both student ID and recipient object based requests.
- Uses service layer for all business logic and data access.
