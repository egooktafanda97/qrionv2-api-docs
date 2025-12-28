# SchoolClassController API Documentation

## Endpoint: `/api/school-classes`

### List School Classes (Paginated)
- **GET** `/api/school-classes`
- **Query Params:**
  - `page` (int, default: 0)
  - `size` (int, default: 10)
  - `sortBy` (string, default: id)
  - `sortDirection` (string, default: DESC)
  - `draw` (int, optional)
  - `format` (string, default: standard, options: standard, jquery-datatable, ant-table)
- **Response:**
  - Paginated list of school classes in various formats.

### Get School Class by ID
- **GET** `/api/school-classes/{id}`
- **Response:**
  - School class details for the given ID.

### Create School Class
- **POST** `/api/school-classes`
- **Body:** `SchoolClassRequest`
- **Response:**
  - Created school class details.

### Update School Class
- **PUT** `/api/school-classes/{id}`
- **Body:** `SchoolClassRequest`
- **Response:**
  - Updated school class details.

### Delete School Class
- **DELETE** `/api/school-classes/{id}`
- **Response:**
  - No content (success message).

### List School Classes with SubClasses (Paginated)
- **GET** `/api/school-classes/with-subclasses`
- **Query Params:**
  - `page`, `size`, `sortBy`, `sortDirection`, `draw`, `format` (same as above)
- **Response:**
  - Paginated list of school classes with their subclasses.

### Get School Class with SubClasses by ID
- **GET** `/api/school-classes/{id}/with-subclasses`
- **Response:**
  - School class with its subclasses for the given ID.

### Compact List of School Classes (with SubClasses)
- **GET** `/api/school-classes/compact`
- **Response:**
  - List of school classes and their subclasses for the current user's yayasan/institution.

---

## Business Logic
- Handles CRUD operations for school classes.
- Supports paginated and formatted listing, and relations with subclasses.
- Provides compact listing for current user's yayasan/institution.
- Uses service and repository layers for all business logic and data access.
