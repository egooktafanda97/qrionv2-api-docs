# PaymentMethodController API Documentation

## Endpoint: `/api/payment-methods`

### List All Payment Methods
- **GET** `/api/payment-methods`
- **Response:**
  - List of all payment methods.

### Create Payment Method
- **POST** `/api/payment-methods`
- **Body:** `PaymentMethodRequest`
- **Response:**
  - Created payment method details.

### Update Payment Method
- **PUT** `/api/payment-methods/{id}`
- **Body:** `PaymentMethodRequest`
- **Response:**
  - Updated payment method details.

### Delete Payment Method
- **DELETE** `/api/payment-methods/{id}`
- **Response:**
  - No content (success message).

---

## Business Logic
- Handles CRUD operations for payment methods.
- Uses service layer for all business logic and data access.
