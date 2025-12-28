# ActivationInstitutAuthController API Documentation

## Endpoint: `/api/activation-institut`

### Get Activation Institut by Authenticated Institution
- **GET** `/api/activation-institut/me`
- **Auth:** Required (uses authenticated user's institution)
- **Response:**
  - ActivationInstitutResponse for the authenticated user's institution, or null if not authenticated.

---

## Business Logic
- Uses Spring Security to get the authenticated user.
- Returns activation information for the user's institution.
- Returns null if not authenticated or user details are not present.
- Uses service layer for business logic and data access.
