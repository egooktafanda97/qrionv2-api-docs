# BroadcastController API Documentation

## Endpoint: `/api/broadcasts`

### Create Broadcast Category
- **POST** `/api/broadcasts/categories`
- **Body:** `CategoryBroadcastRequest`
- **Response:**
  - Created category details.

### List Broadcast Categories
- **GET** `/api/broadcasts/categories`
- **Response:**
  - List of all broadcast categories.

### Create Broadcast Template
- **POST** `/api/broadcasts/templates`
- **Body:** `TemplateBroadcastRequest`
- **Response:**
  - Created template details.

### List Broadcast Templates
- **GET** `/api/broadcasts/templates`
- **Response:**
  - List of all broadcast templates.

### Get Broadcast Template by ID
- **GET** `/api/broadcasts/templates/{id}`
- **Response:**
  - Template details for the given ID.

### Create Broadcast
- **POST** `/api/broadcasts`
- **Body:** `BroadcastRequest`
- **Response:**
  - Created broadcast details.

### List Broadcasts
- **GET** `/api/broadcasts`
- **Response:**
  - List of all broadcasts.

### Get Broadcast Dashboard Summary
- **GET** `/api/broadcasts/dashboard/summary`
- **Response:**
  - Dashboard summary data for broadcasts.

### Get Broadcast History (Paginated)
- **GET** `/api/broadcasts/dashboard/history`
- **Query Params:**
  - `search` (string, optional)
  - `page` (int, default: 0)
  - `size` (int, default: 10)
- **Response:**
  - Paginated broadcast history data.

### Add Broadcast Recipient
- **POST** `/api/broadcasts/recipients`
- **Body:** `BroadcastRecipientRequest`
- **Response:**
  - Success message.

### Render Broadcast Template
- **POST** `/api/broadcasts/templates/render`
- **Body:** `TemplateRenderRequest`
- **Response:**
  - Rendered template and list of missing keys.

---

## Business Logic
- Handles CRUD operations for broadcast categories, templates, and broadcasts.
- Supports dashboard summary and paginated history.
- Allows adding recipients and rendering templates with dynamic values.
- Uses service layer for all business logic and data access.
