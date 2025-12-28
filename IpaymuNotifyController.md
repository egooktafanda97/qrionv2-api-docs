# IpaymuNotifyController API Documentation

## Endpoint: `/api/ipaymu/notify`

### Notify Payment Success
- **POST** `/api/ipaymu/notify/success`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes payment success, updates transaction, activates institution, installs addons, and logs payment.

### Notify Payment Return
- **POST** `/api/ipaymu/notify/return`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes payment return callback.

### Notify Addon Payment Success
- **POST** `/api/ipaymu/notify/success/addons`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes addon payment success, installs addons, updates transaction, and logs payment.

### Notify Addon Payment Cancel
- **POST** `/api/ipaymu/notify/cancel/addons`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes addon payment cancellation.

### Notify Addon Payment Return
- **POST** `/api/ipaymu/notify/return/addons`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes addon payment return callback.

### Notify Payment Cancel
- **POST** `/api/ipaymu/notify/cancel`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes payment cancellation.

### Notify Billing Payment Success
- **POST** `/api/ipaymu/notify/successBilling`
- **Body:** `IpaymuNotifyRequest`
- **Response:**
  - Processes billing payment success, updates billing, logs payment, and returns result.

---

## Business Logic
- Handles payment gateway callbacks for success, cancel, and return events for both main and addon transactions.
- Updates transaction status, activates institutions, installs addons, and logs all payment events.
- Uses service and repository layers for all business logic and data access.
- Ensures idempotency and error handling for repeated or failed callbacks.
