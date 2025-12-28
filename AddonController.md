# AddonController API Documentation

## Endpoint: `/api/addons`

### Get All Addons
- **GET** `/api/addons`
- **Response:**
  - List of all addons.

### Get Addon by ID
- **GET** `/api/addons/{id}`
- **Response:**
  - Addon details for the given ID.

### Create Addon
- **POST** `/api/addons`
- **Body:** `AddonRequest`
- **Response:**
  - Created addon details.

### Update Addon
- **PUT** `/api/addons/{id}`
- **Body:** `AddonRequest`
- **Response:**
  - Updated addon details.

### Delete Addon
- **DELETE** `/api/addons/{id}`
- **Response:**
  - No content (success message).

---

## Business Logic
- CRUD operations for Addon entities.
- Uses service layer for all business logic and data access.
- Validates request bodies for create and update operations.
