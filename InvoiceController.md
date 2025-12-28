# InvoiceController API Documentation

## Endpoint: `/api/invoices`

### Create Invoice
- **POST** `/api/invoices`
- **Body:** `InvoiceRequest`
- **Auth:** Required
- **Response:**
  - Created invoice details.

### List All Invoices
- **GET** `/api/invoices`
- **Auth:** Required
- **Response:**
  - List of all invoices for yayasan/institution.

### Get Invoice by ID
- **GET** `/api/invoices/{id}`
- **Auth:** Required
- **Response:**
  - Invoice details for the given ID.

### Get Invoice by Number
- **GET** `/api/invoices/number/{invoiceNumber}`
- **Auth:** Required
- **Response:**
  - Invoice details for the given invoice number.

### Update Invoice
- **PUT** `/api/invoices/{id}`
- **Body:** `InvoiceRequest`
- **Auth:** Required
- **Response:**
  - Updated invoice details.

### Delete Invoice
- **DELETE** `/api/invoices/{id}`
- **Auth:** Required
- **Response:**
  - No content (success message).

### Generate Invoice Number
- **GET** `/api/invoices/generate-number`
- **Response:**
  - Generated invoice number string.

---

## Business Logic
- Handles CRUD operations for invoices.
- Supports lookup by ID and invoice number.
- Generates unique invoice numbers.
- Requires yayasan and institution context for all operations.
- Uses service layer for all business logic and data access.
