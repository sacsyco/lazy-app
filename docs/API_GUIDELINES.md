# API Design Guidelines

## Request/Response Format

### Success Response (200/201)
```json
{
  "success": true,
  "data": {},
  "message": "Resource created successfully"
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

## Status Codes

- `200` - OK
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `409` - Conflict
- `422` - Unprocessable Entity
- `500` - Internal Server Error

## Authentication

All authenticated endpoints require:
```
Authorization: Bearer <JWT_TOKEN>
```

## Rate Limiting

- 100 requests per minute per user
- 10 requests per second for payment endpoints

Response headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1234567890
```

## Pagination

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "pages": 5
  }
}
```

## Filtering

Use query parameters:
```
/api/jobs?zipCode=10001&status=open&category=repair
```

## Sorting

```
/api/jobs?sort=createdAt:desc,price:asc
```

## Versioning

- Current version: `/api/v1`
- Maintain backward compatibility
- Deprecate old versions with warning headers

## Documentation

All endpoints should be documented with:
- Description
- Request parameters/body
- Response examples
- Error codes
- Authentication requirements

## Database Transactions

For multi-step operations:
1. Begin transaction
2. Validate all conditions
3. Commit or rollback
4. Return appropriate status

## Logging

Log all:
- Authentication attempts
- Payment transactions
- Errors with stack traces
- Performance metrics
