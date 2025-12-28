# InstitutionController API Documentation

## Endpoint: `/api/institutions`

### Update Institution Logo (by URL)
- **PUT** `/api/institutions/logo`
- **Body:** `UpdateInstitutionLogoRequest`
- **Auth:** Required
- **Response:**
  - Updated institution details with new logo URL.

### Upload Institution Logo (Multipart)
- **POST/PUT** `/api/institutions/logo/upload`
- **Body:** Multipart file (`file`)
- **Auth:** Required
- **Response:**
  - Updated institution details with uploaded logo path.

---

## Business Logic
- Allows updating institution logo by URL or by uploading a file.
- Handles file storage and generates unique filenames for uploads.
- Requires authenticated user with yayasan and institution context.
- Uses service layer for all business logic and data access.
