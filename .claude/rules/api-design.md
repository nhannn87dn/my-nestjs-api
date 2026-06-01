# API Design Conventions

**Purpose**: RESTful API design standards and best practices

## API Versioning

```
/api/v1/[resources]    ← Version in URL path
```

## Resource Naming

```typescript
// Use plural nouns for collections
GET    /api/v1/users
GET    /api/v1/posts
GET    /api/v1/comments

// NOT verbs
GET    /api/v1/getUsers      ❌ Wrong
GET    /api/v1/fetchPosts    ❌ Wrong
```

## HTTP Methods

```typescript
// GET - Retrieve resource
GET /api/v1/users              // List all
GET /api/v1/users/:id          // Get one
GET /api/v1/users/:id/posts    // Get related

// POST - Create new resource
POST /api/v1/users             // Create
POST /api/v1/users/:id/posts   // Create related

// PUT - Replace entire resource
PUT /api/v1/users/:id          // Replace

// PATCH - Partial update
PATCH /api/v1/users/:id        // Update specific fields

// DELETE - Remove resource
DELETE /api/v1/users/:id       // Delete
DELETE /api/v1/users/:id/posts/:postId  // Delete related
```

## Query Parameters

```typescript
// Pagination
?page=1&limit=10

// Sorting
?sortBy=createdAt&sortOrder=DESC

// Filtering
?status=active&category=news

// Search
?search=keyword

// Selection
?fields=id,name,email  // Return specific fields only

// Include relations
?include=posts,comments
```

## Status Codes

```typescript
// 2xx - Success
200 OK                  // Successful GET, PUT, PATCH
201 CREATED             // Successful POST
202 ACCEPTED            // Async operation accepted
204 NO_CONTENT          // Successful DELETE or empty response

// 4xx - Client Error
400 BAD_REQUEST         // Invalid request format
401 UNAUTHORIZED        // Missing authentication
403 FORBIDDEN           // Insufficient permission
404 NOT_FOUND           // Resource not found
409 CONFLICT            // Resource conflict (duplicate)
422 UNPROCESSABLE_ENTITY // Validation error

// 5xx - Server Error
500 INTERNAL_SERVER_ERROR // Server error
503 SERVICE_UNAVAILABLE    // Service down
```

## Request Headers

```typescript
// Standard headers
Content-Type: application/json
Authorization: Bearer <token>
X-API-Key: <key>           // Optional API key
X-Request-ID: <uuid>       // Request tracking
Accept-Language: en-US
```

## Response Headers

```typescript
// Standard headers
Content-Type: application/json
Content-Length: <length>
X-Request-ID: <uuid>       // Request tracking
X-RateLimit-Limit: 1000    // Rate limit info
X-RateLimit-Remaining: 999
X-RateLimit-Reset: <timestamp>
Cache-Control: max-age=3600
```

## Nested Resources

```typescript
// For tightly coupled resources
GET    /api/v1/users/:userId/posts              // List user's posts
GET    /api/v1/users/:userId/posts/:postId      // Get specific post
POST   /api/v1/users/:userId/posts              // Create user's post
PUT    /api/v1/users/:userId/posts/:postId      // Update user's post
DELETE /api/v1/users/:userId/posts/:postId      // Delete user's post
```

## Custom Actions

```typescript
// For non-CRUD operations
POST   /api/v1/users/:id/activate              // Activate user
POST   /api/v1/users/:id/deactivate            // Deactivate user
POST   /api/v1/users/:id/change-password       // Change password
POST   /api/v1/users/:id/reset-password        // Reset password

// Prefer POST over GET for state changes
// Avoid using GET for mutations
```

## Bulk Operations

```typescript
// Create multiple
POST /api/v1/users/bulk
Body: { items: [{ ... }, { ... }] }

// Update multiple
PUT /api/v1/users/bulk
Body: { items: [{ id: 1, ... }, { id: 2, ... }] }

// Delete multiple
DELETE /api/v1/users/bulk
Body: { ids: [1, 2, 3] }
```

## Filtering and Searching

```typescript
// Exact match
?status=active
?category=news

// Range
?minPrice=10&maxPrice=100
?createdAfter=2025-01-01&createdBefore=2025-12-31

// Contains (fuzzy search)
?search=keyword

// Multiple values
?status=active,pending
?id=1,2,3
```

## Pagination Best Practices

```typescript
// Use offset/limit (pagination)
GET /api/v1/users?page=1&limit=10

// Response includes pagination metadata
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  }
}
```

## Rate Limiting

```typescript
// Headers indicate rate limit status
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000

// Exceeded rate limit
Status: 429 Too Many Requests
Retry-After: 60
```

## Best Practices Summary

✅ **DO**
- Use meaningful resource names (nouns, plural)
- Use correct HTTP methods
- Use proper status codes
- Implement pagination for large datasets
- Version your API
- Document with OpenAPI/Swagger
- Use consistent naming conventions
- Include request IDs for tracking
- Implement rate limiting
- Support filtering and sorting
- Use HTTPS only
- Implement CORS properly

❌ **DON'T**
- Use verbs in URLs
- Use GET for mutations
- Return 200 for all responses
- Mix API versions
- Expose internal details
- Skip authentication
- Forget pagination
- Use unclear parameter names
- Return too much data
- Break backward compatibility
