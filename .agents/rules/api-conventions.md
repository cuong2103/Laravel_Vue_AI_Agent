# API Conventions

## REST API Design Standards

### URL Structure
- Use **kebab-case** for URL paths: `/api/user-profiles`
- Use **plural nouns** for resource collections: `/api/users`, `/api/products`
- Nest related resources: `/api/users/{user}/orders`
- API version prefix: `/api/v1/...`

### HTTP Methods
| Method | Usage |
|--------|-------|
| GET | Read resources (idempotent) |
| POST | Create new resource |
| PUT | Replace entire resource |
| PATCH | Partial update |
| DELETE | Remove resource |

### HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | OK — Successful GET/PUT/PATCH |
| 201 | Created — Successful POST |
| 204 | No Content — Successful DELETE |
| 400 | Bad Request — Invalid input |
| 401 | Unauthorized — Not authenticated |
| 403 | Forbidden — No permission |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity — Validation failed (Laravel FormRequest default) |
| 500 | Internal Server Error |

### Request/Response Format
```json
// Success response
{
  "success": true,
  "data": { ... },
  "message": "Optional message"
}

// Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human readable message",
    "details": { "email": ["The email has already been taken."] }
  }
}

// Paginated list response (Laravel paginate())
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total": 100,
    "last_page": 5
  }
}
```

### Laravel Route Definition
```php
// routes/api.php
Route::prefix('v1')->middleware('auth:sanctum')->group(function () {
    Route::apiResource('users', UserController::class);
    Route::apiResource('users.orders', UserOrderController::class)->shallow();

    // Custom actions (non-CRUD)
    Route::post('payments/{payment}/refund', [PaymentController::class, 'refund']);
    Route::post('auth/logout', [AuthController::class, 'logout']);
});
```

### Naming Conventions
- Request/response body fields: **snake_case** (PHP/Laravel convention)
- Query parameters: **snake_case** (`sort_by`, `per_page`)
- Always return consistent field names

### Filtering & Pagination
```
GET /api/v1/users?page=1&per_page=20&sort_by=created_at&order=desc
GET /api/v1/products?category=electronics&min_price=100
```

### Documentation
- Every endpoint MUST have OpenAPI annotations (use Scribe package)
- Include request body schema, response schema, and error codes
