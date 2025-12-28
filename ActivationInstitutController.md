# ActivationInstitutController API Documentation

## Endpoint: `/api/activation`

### Get Current Institution Activation
- **GET** `/api/activation`
- **Auth:** Required
- **Response:**
  - Institution activation info for the current user.

### Get Addons Installed by Institution
- **GET** `/api/activation/addons-by-institution`
- **Auth:** Required
- **Response:**
  - List of installed addons for the user's institution.

### Get Active Addons by Institution
- **GET** `/api/activation/addons-active`
- **Auth:** Required
- **Response:**
  - List of active (ENABLED) addons for the user's institution.

### Get All Addons Installed (Admin)
- **GET** `/api/activation/addons-installed`
- **Auth:** Admin/Superadmin
- **Response:**
  - List of all installed addons with full details.

### Get ActivationInstitut by Institution
- **GET** `/api/activation/activation-institut-by-institution`
- **Auth:** Required
- **Response:**
  - List of activation records for the user's institution.

### Get Active ActivationInstitut by Institution
- **GET** `/api/activation/activation-institut-active`
- **Auth:** Required
- **Response:**
  - List of active activation records for the user's institution.

### Get All ActivationInstitut (Admin)
- **GET** `/api/activation/activation-institut-all`
- **Auth:** Admin/Superadmin
- **Response:**
  - List of all activation records with full details.

---

## Business Logic
- Uses authenticated user to determine institution context.
- Handles permission checks and throws errors if user is not linked to an institution.
- Provides detailed mapping of related entities (addon, institution, plan, transaction, etc.) in responses.
- Uses service and repository layers for all business logic and data access.
