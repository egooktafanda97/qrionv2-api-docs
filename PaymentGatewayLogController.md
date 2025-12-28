# PaymentGatewayLogController API Documentation

## Endpoint: `/api/payment-gateway-logs`

### List All Payment Gateway Logs (Institution)
- **GET** `/api/payment-gateway-logs`
- **Auth:** Required
- **Response:**
  - List of all payment gateway logs for the authenticated user's institution.

### Get Payment Gateway Log by ID
- **GET** `/api/payment-gateway-logs/{id}`
- **Response:**
  - Payment gateway log details for the given ID.

### Get Payment Gateway Log by Reference Number
- **GET** `/api/payment-gateway-logs/reference/{referenceNumber}`
- **Response:**
  - Payment gateway log details for the given reference number.

---

## Business Logic
- Handles retrieval of payment gateway logs by institution, ID, or reference number.
- Requires authentication and institution context for listing.
- Uses service layer for all business logic and data access.
