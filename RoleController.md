# RoleController API Documentation

## Endpoint: `/api/roles`

### List Roles (Paginated)
- **GET** `/api/roles`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: ASC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Paginated list of roles in various formats.

### Get Role by ID
- **GET** `/api/roles/{id}`
- **Response:**
  - Role details for the given ID.

### Create Role
- **POST** `/api/roles`
- **Body:** `RoleRequest`
- **Response:**
  - Created role details.

### Update Role
- **PUT** `/api/roles/{id}`
- **Body:** `RoleRequest`
- **Response:**
  - Updated role details.

### Delete Role
- **DELETE** `/api/roles/{id}`
- **Response:**
  - Success message (role deleted).

### Assign Privileges to Role
- **POST** `/api/roles/{roleId}/privileges`
- **Body:** `AssignPrivilegesToRoleRequest`
- **Response:**
  - Role details with assigned privileges.

### Remove Privileges from Role
- **DELETE** `/api/roles/{roleId}/privileges`
- **Body:** `AssignPrivilegesToRoleRequest`
- **Response:**
  - Role details with removed privileges.

---

## Business Logic
- Handles CRUD operations for roles.
- Supports assigning and removing privileges to/from roles.
- Uses service layer for all business logic and data access.
- Supports paginated and formatted listing.
