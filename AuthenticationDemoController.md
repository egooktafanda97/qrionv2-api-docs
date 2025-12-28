# AuthenticationDemoController API Documentation

## Endpoint: `/api/demo`

### Register Demo Institution
- **POST** `/api/demo/register-institution`
- **Body:** `RegisterDemoRequest`
- **Response:**
  - Success message, demo institution registered.

### Verify Demo Registration OTP
- **POST** `/api/demo/register-otp-verify`
- **Body:** `RequestDemoVerify`
- **Response:**
  - Success or failure message for OTP verification.

### Login with OTP (Demo)
- **POST** `/api/demo/login-with-otp`
- **Body:** `LoginOtpRequest`
- **Response:**
  - Success message, OTP sent for login.

### Verify Login OTP (Demo)
- **POST** `/api/demo/login-otp-verify`
- **Body:** `RequestDemoVerify`
- **Response:**
  - LoginResponse (JWT token and user info) if OTP is valid.

---

## Business Logic
- Handles demo registration and login flows using OTP.
- Uses service layer for all business logic and data access.
- Throws errors for failed OTP verification.
- Integrates with logging for activity tracking.
