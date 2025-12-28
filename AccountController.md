# AccountController API Documentation

## Endpoint: `/api/account`

### Get Account Name
- **GET** `/api/account`
- **Roles:** INSTITUTION
- **Response:**
  - Account name (string)

### Get Admin Info
- **GET** `/api/account/account/admin`
- **Roles:** ADMIN
- **Query Params:**
  - `param` (string, optional)
- **Response:**
  - Admin info (string)

### Check Account
- **GET** `/api/account/account/check`
- **Query Params:**
  - `param` (string, required)
- **Response:**
  - "ok" if param is valid, 403 if not.

---

## Business Logic
- Uses custom `@Roles` annotation for role-based access control.
- Throws `AccessDeniedException` for invalid access.
- Returns simple string responses for demonstration.
