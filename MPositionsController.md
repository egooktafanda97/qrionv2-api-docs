# MPositionsController API Documentation

## Endpoint: `/api/m-positions`

### List All Positions
- **GET** `/api/m-positions`
- **Response:**
  - List of all positions.

### Get Position by ID
- **GET** `/api/m-positions/{id}`
- **Response:**
  - Position details for the given ID.

### Create Position
- **POST** `/api/m-positions`
- **Body:** `MPositionsRequest`
- **Response:**
  - Created position details.

### Update Position
- **PUT** `/api/m-positions/{id}`
- **Body:** `MPositionsRequest`
- **Response:**
  - Updated position details.

### Delete Position
- **DELETE** `/api/m-positions/{id}`
- **Response:**
  - No content (success message).

---

## Business Logic
- Handles CRUD operations for positions (jabatan/master data jabatan).
- Uses service layer for all business logic and data access.
