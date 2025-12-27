Standard API response

This project uses a consistent response envelope `ApiResponse<T>` and the helper class `ApiResponses` to produce standardized HTTP responses.

Success response structure

- `success` : boolean (true)
- `message` : string (user-facing message)
- `data` : object | null (payload)
- `status` : integer (HTTP status code)
- `timestamp` : string (ISO-8601 instant)
- `path` : string (request path)

Example (200 OK)

{
  "success": true,
  "message": "OK",
  "data": { "id": 123, "name": "Acme" },
  "status": 200,
  "timestamp": "2025-11-21T12:34:56.789Z",
  "path": "/api/institutions/123"
}

Example (201 Created)

{
  "success": true,
  "message": "Institution created",
  "data": { "id": 456, "name": "New Inst" },
  "status": 201,
  "timestamp": "2025-11-21T12:34:56.789Z",
  "path": "/auth/register"
}

Error response structure

- `success` : boolean (false)
- `message` : string (error message)
- `errorCode` : string | null (internal error code)
- `status` : integer (HTTP status code)
- `timestamp` : string (ISO-8601 instant)
- `path` : string (request path)
- `errors` : map<string,string> | null (field validation errors)

Example (validation error 400)

{
  "success": false,
  "message": "Validation failed",
  "errorCode": "INVALID_INPUT",
  "status": 400,
  "timestamp": "2025-11-21T12:34:56.789Z",
  "path": "/api/demo/register-institution",
  "errors": {
    "email": "must be a well-formed email address",
    "name": "must not be blank"
  }
}

How to use in controllers

- Prefer `ApiResponses.ok(data, message, request)` for 200 responses.
- Prefer `ApiResponses.created(data, message, request)` for 201 responses.
- Use `ApiResponses.noContent(message, request)` for 204 responses.
- Exceptions and errors are handled centrally by `GlobalExceptionHandler` which returns the standard error envelope.

Examples

```java
@PostMapping
public ResponseEntity<ApiResponse<MyDto>> create(@RequestBody MyCreateRequest req, HttpServletRequest request) {
  MyDto dto = service.create(req);
  return ApiResponses.created(dto, "Created", request);
}
```

Notes

- The `GlobalExceptionHandler` already standardizes error responses (validation, constraints, access denied, etc.).
- `ApiResponses` sets `timestamp` and `path` automatically when a `HttpServletRequest` is passed.

