# AuthenticationController API Documentation

## Endpoint: `/api/auth`

### Register User and Send OTP
- **POST** `/api/auth/register`
- **Body:** `RegisterRequest`
- **Response:**
  - Success message, OTP sent to WhatsApp.

### Verify OTP and Login
- **POST** `/api/auth/verify-otp`
- **Query Params:**
  - `telephone` (string, required)
  - `otp` (string, required)
- **Response:**
  - LoginResponse (JWT token and user info).

### Set/Update Password (Authenticated)
- **POST** `/api/auth/set-password`
- **Headers:** `Authorization: Bearer <token>`
- **Body:** `SetPasswordRequest`
- **Response:**
  - Success message.

### Login with Identity and Password
- **POST** `/api/auth/login`
- **Body:** `LoginIdentityRequest`
- **Response:**
  - LoginResponse (JWT token and user info).

### Resend OTP
- **POST** `/api/auth/resend-otp`
- **Body:** `ResendOtpRequest`
- **Response:**
  - Success message, OTP resent to WhatsApp.

### Forgot Password (Send OTP)
- **POST** `/api/auth/forgot-password`
- **Body:** `ForgotPasswordRequest`
- **Response:**
  - Success message, OTP sent to WhatsApp.

### Reset Password with OTP
- **POST** `/api/auth/reset-password`
- **Body:** `ResetPasswordRequest`
- **Response:**
  - Success message.

### Get Current User Profile
- **GET** `/api/auth/me`
- **Headers:** `Authorization: Bearer <token>`
- **Response:**
  - UserMeResponse (full profile data).

---

## Business Logic
- Handles user registration, OTP verification, login, password management, and profile retrieval.
- Uses service layer for all business logic and data access.
- Integrates with WhatsApp for OTP delivery.
- Uses JWT authentication for protected endpoints.
- Logs activities for auditing.
