# ParentController API Documentation

## Endpoint: `/api/parents`

### Register Parent
- **POST** `/api/parents/register`
- **Body:** `ParentRegisterRequest`
- **Response:**
  - Created parent details, OTP sent to WhatsApp.

### Verify Parent OTP
- **POST** `/api/parents/verify-otp`
- **Body:** `ParentVerifyOtpRequest`
- **Response:**
  - Parent details after successful OTP verification and activation.

### List All Parents (Paginated)
- **GET** `/api/parents`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Paginated list of parents in various formats.

### Get Parent by ID
- **GET** `/api/parents/{id}`
- **Response:**
  - Parent details for the given ID.

### Get Parent by Phone
- **GET** `/api/parents/phone/{phone}`
- **Response:**
  - Parent details for the given phone number.

### Delete Parent (Soft Delete)
- **DELETE** `/api/parents/{id}`
- **Response:**
  - No content (success message).

---

## Business Logic
- Handles parent registration, OTP verification, and activation.
- Supports paginated listing, lookup by ID and phone, and soft deletion.
- Uses service layer for all business logic and data access.
- Sends OTP via WhatsApp and links parent to students upon activation.
