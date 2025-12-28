# DebugIpaymuController API Documentation

## Endpoint: `/api/debug`

### Open iPaymu Transaction (Debug)
- **POST** `/api/debug/ipaymu/open`
- **Body:** `IpaymuRequest` (JSON)
- **Response:**
  - Raw response from iPaymuService.openTrx, logs request and response, persists payment gateway log.

### Debug Create Payment Transaction
- **POST** `/api/debug/transaction/create-payment`
- **Body:** `Map<String, Object>` (JSON)
- **Response:**
  - Raw response from TransactionBusinessService.createPayment, or error if service unavailable.

---

## Business Logic
- Forwards requests to payment gateway service for debugging.
- Logs and persists request/response for audit.
- Used for sandbox/prod debugging and troubleshooting payment integration.
