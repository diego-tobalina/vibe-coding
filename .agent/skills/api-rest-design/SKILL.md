---
name: api-rest-design
description: REST API design standards including URL conventions, HTTP methods, status codes, pagination, versioning, and error responses. Use when designing or implementing REST APIs.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# REST API Design Guidelines

Standards for designing consistent, scalable REST APIs.

## URL Design

### Resource Naming

```
✅ CORRECT                        ❌ WRONG
/api/users                         /api/getUsers
/api/users/{id}                    /api/user/{id}
/api/users/{id}/orders             /api/orders?userId={id}
/api/orders/{orderId}/items        /api/orderItems?orderId={id}
```

### Rules
- Use **plural nouns** for collections (`/users`, not `/user`)
- Use **lowercase** with hyphens for multi-word (`/order-items`)
- Use **UUIDs or opaque IDs** - never expose sequential DB IDs publicly
- **No verbs** in URLs - HTTP methods indicate the action
- **Hierarchical nesting** for sub-resources (max 2-3 levels deep)

## HTTP Methods

| Method | Use For | Success | Error |
|--------|---------|---------|-------|
| **GET** | Retrieve resource(s) | 200 | 404, 400 |
| **POST** | Create new resource | 201 | 400, 409 |
| **PUT** | Full update/replace | 200 | 400, 404 |
| **PATCH** | Partial update | 200 | 400, 404 |
| **DELETE** | Remove resource | 204 | 404 |

### Examples

```java
// Spring Boot
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping
    public Page<UserDto> list(Pageable pageable) { ... }
    
    @GetMapping("/{id}")
    public UserDto get(@PathVariable UUID id) { ... }
    
    @PostMapping
    public ResponseEntity<UserDto> create(@RequestBody @Valid CreateUserDto dto) {
        UserDto created = userService.create(dto);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
    
    @PutMapping("/{id}")
    public UserDto update(@PathVariable UUID id, @RequestBody @Valid UpdateUserDto dto) { ... }
    
    @PatchMapping("/{id}")
    public UserDto patch(@PathVariable UUID id, @RequestBody @Valid PatchUserDto dto) { ... }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable UUID id) { ... }
}
```

```typescript
// Node.js/Express
import { Router } from 'express';

const router = Router();

router.get('/', userController.list);
router.get('/:id', userController.get);
router.post('/', validate(createUserSchema), userController.create);
router.put('/:id', validate(updateUserSchema), userController.update);
router.patch('/:id', validate(patchUserSchema), userController.patch);
router.delete('/:id', userController.delete);

export { router as userRoutes };
```

## HTTP Status Codes

### Success (2xx)
- **200 OK** - Standard success response
- **201 Created** - Resource created successfully (include Location header)
- **202 Accepted** - Request accepted for async processing
- **204 No Content** - Success with no body (DELETE, empty list)

### Client Errors (4xx)
- **400 Bad Request** - Invalid request body/parameters
- **401 Unauthorized** - Authentication required
- **403 Forbidden** - No permission (authenticated but not authorized)
- **404 Not Found** - Resource doesn't exist
- **409 Conflict** - Resource conflict (e.g., duplicate email)
- **422 Unprocessable Entity** - Validation failed (business rules)
- **429 Too Many Requests** - Rate limit exceeded

### Server Errors (5xx)
- **500 Internal Server Error** - Unexpected server error
- **502 Bad Gateway** - Upstream service error
- **503 Service Unavailable** - Temporary outage/overload

## Error Response Format

### Standard Error Structure

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id 550e8400-e29b-41d4-a716-446655440000 not found",
    "details": [
      {
        "field": "email",
        "message": "Email is already registered"
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/users/550e8400-e29b-41d4-a716-446655440000",
    "requestId": "req-123456"
  }
}
```

### Error Codes Convention

```
{RESOURCE}_{ERROR_TYPE}

Examples:
USER_NOT_FOUND
USER_DUPLICATE_EMAIL
ORDER_INVALID_STATUS
PAYMENT_INSUFFICIENT_FUNDS
VALIDATION_ERROR
AUTHENTICATION_REQUIRED
AUTHORIZATION_INSUFFICIENT
```

### Implementation

```java
// Java/Spring
public record ErrorResponse(
    String code,
    String message,
    List<ErrorDetail> details,
    Instant timestamp,
    String path,
    String requestId
) {
    public ErrorResponse(String code, String message) {
        this(code, message, null, Instant.now(), null, null);
    }
    
    public record ErrorDetail(String field, String message, Object rejectedValue) {}
}

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusiness(BusinessException ex, WebRequest request) {
        ErrorResponse response = new ErrorResponse(
            ex.getErrorCode(),
            ex.getMessage(),
            null,
            Instant.now(),
            request.getDescription(false).replace("uri=", ""),
            MDC.get("requestId")
        );
        return ResponseEntity.status(ex.getHttpStatus()).body(response);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex, WebRequest request) {
        List<ErrorResponse.ErrorDetail> details = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> new ErrorResponse.ErrorDetail(
                error.getField(),
                error.getDefaultMessage(),
                error.getRejectedValue()
            ))
            .collect(Collectors.toList());
        
        ErrorResponse response = new ErrorResponse(
            "VALIDATION_ERROR",
            "Request validation failed",
            details,
            Instant.now(),
            request.getDescription(false).replace("uri=", ""),
            MDC.get("requestId")
        );
        return ResponseEntity.badRequest().body(response);
    }
}
```

```typescript
// Node.js
export interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Array<{
      field: string;
      message: string;
      rejectedValue?: unknown;
    }>;
    timestamp: string;
    path: string;
    requestId: string;
  };
}

// Error handler middleware
export function errorHandler(
  err: AppError,
  req: Request,
  res: Response,
  next: NextFunction
) {
  const statusCode = err.statusCode || 500;
  const response: ErrorResponse = {
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.message || 'Internal server error',
      timestamp: new Date().toISOString(),
      path: req.path,
      requestId: req.correlationId,
    },
  };
  
  if (err.details) {
    response.error.details = err.details;
  }
  
  res.status(statusCode).json(response);
}
```

## Pagination

### Request Parameters

```
GET /api/users?page=0&size=20&sort=name,asc
GET /api/users?page=0&size=20&sort=createdAt,desc&sort=name,asc
```

Parameters:
- `page` - Page number (0-indexed)
- `size` - Items per page (default: 20, max: 100)
- `sort` - Sort field and direction (`field,direction`)

### Response Format

```json
{
  "data": [
    { "id": "...", "name": "John", "email": "john@example.com" },
    { "id": "...", "name": "Jane", "email": "jane@example.com" }
  ],
  "pagination": {
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8,
    "first": true,
    "last": false
  },
  "links": {
    "self": "/api/users?page=0&size=20",
    "first": "/api/users?page=0&size=20",
    "next": "/api/users?page=1&size=20",
    "last": "/api/users?page=7&size=20"
  }
}
```

### Implementation

```java
// Spring Data automatically provides Page<T>
@GetMapping
public PageResponse<UserDto> findAll(Pageable pageable) {
    Page<UserDto> page = userService.findAll(pageable);
    return PageResponse.of(page);
}

public record PageResponse<T>(
    List<T> data,
    Pagination pagination,
    Links links
) {
    public static <T> PageResponse<T> of(Page<T> page) {
        return new PageResponse<>(
            page.getContent(),
            new Pagination(
                page.getNumber(),
                page.getSize(),
                page.getTotalElements(),
                page.getTotalPages(),
                page.isFirst(),
                page.isLast()
            ),
            null // Build links based on current request
        );
    }
    
    public record Pagination(int page, int size, long totalElements, int totalPages, boolean first, boolean last) {}
    public record Links(String self, String first, String next, String last) {}
}
```

## API Versioning

### URL Path Versioning (Recommended)

```
/api/v1/users
/api/v2/users
```

Benefits:
- Clear and explicit
- Easy to route
- Cache-friendly
- Works with all clients

### Implementation

```java
// Spring Boot
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { ... }
```

```typescript
// Express
const v1Router = Router();
const v2Router = Router();

v1Router.get('/users', userControllerV1.list);
v2Router.get('/users', userControllerV2.list);

app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
```

### Version Strategy

1. **Major versions (v1, v2)** - Breaking changes
2. **Support N-1 versions** - Keep previous version for 6-12 months
3. **Deprecation headers**:
   ```
   Deprecation: true
   Sunset: Sat, 01 Jun 2024 00:00:00 GMT
   ```

## Request/Response Examples

### Complete CRUD Example

```
POST /api/v1/users
Content-Type: application/json

{
  "email": "john@example.com",
  "name": "John Doe"
}

---
201 Created
Location: /api/v1/users/550e8400-e29b-41d4-a716-446655440000

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john@example.com",
  "name": "John Doe",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

```
GET /api/v1/users/550e8400-e29b-41d4-a716-446655440000

---
200 OK

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john@example.com",
  "name": "John Doe",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

```
PATCH /api/v1/users/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "name": "John Smith"
}

---
200 OK

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john@example.com",
  "name": "John Smith",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

```
DELETE /api/v1/users/550e8400-e29b-41d4-a716-446655440000

---
204 No Content
```

## Common API Design Mistakes

### Mistake 1: Using Verbs in URLs

**The Problem:**
```
❌ WRONG
GET /api/getUsers
POST /api/createUser
PUT /api/updateUser
DELETE /api/deleteUser
```

**Why it's bad:**
- HTTP methods already indicate the action
- URLs should identify resources, not actions
- Makes API inconsistent and hard to learn

**Solution:**
```
✅ CORRECT
GET    /api/users          # Get all users
GET    /api/users/{id}     # Get specific user
POST   /api/users          # Create user
PUT    /api/users/{id}     # Update user (full)
PATCH  /api/users/{id}     # Update user (partial)
DELETE /api/users/{id}     # Delete user
```

### Mistake 2: Exposing Database IDs

**The Problem:**
```json
{
  "id": 12345,  // ❌ Sequential DB ID
  "name": "John"
}
```

**Why it's dangerous:**
- Allows attackers to guess other IDs (`/users/12346`)
- Reveals business volume (if ID is 1000000, you have ~1M users)
- Hard to migrate databases

**Solution:**
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)  // ✅ Use UUID
    private UUID id;
    
    // Or use a separate public ID
    @Column(unique = true)
    private String publicId;  // e.g., "usr_550e8400-e29b-41d4-a716-446655440000"
}
```

### Mistake 3: Inconsistent Error Responses

**The Problem:**
```json
// ❌ WRONG - Different formats for different errors
// Validation error:
{
  "error": "Email is invalid"
}

// Not found:
{
  "message": "User not found",
  "status": 404
}

// Server error:
{
  "error_code": "INTERNAL_ERROR",
  "description": "Something went wrong",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Why it's bad:**
- Clients can't parse errors consistently
- Hard to handle errors programmatically
- Poor developer experience

**Solution:**
```json
// ✅ CORRECT - Consistent format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req-123456"
  }
}
```

### Mistake 4: Wrong HTTP Status Codes

**The Problem:**
```java
// ❌ WRONG - Always return 200
@PostMapping
public ResponseEntity<?> createUser(@RequestBody UserDto dto) {
    try {
        User user = userService.create(dto);
        return ResponseEntity.ok(user);  // Should be 201
    } catch (DuplicateEmailException e) {
        return ResponseEntity.ok()  // Should be 409!
            .body(Map.of("error", "Email exists"));
    }
}
```

**Common mistakes:**
- Returning 200 for created resources (should be 201)
- Returning 200 for errors (confuses clients)
- Returning 500 for validation errors (should be 400)
- Returning 404 when user lacks permission (should be 403)

**Solution:**
```java
// ✅ CORRECT - Proper status codes
@PostMapping
public ResponseEntity<UserDto> createUser(@RequestBody UserDto dto) {
    UserDto created = userService.create(dto);
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(created.getId())
        .toUri();
    
    return ResponseEntity.created(location).body(created);  // 201
}

@ExceptionHandler(DuplicateEmailException.class)
public ResponseEntity<ErrorResponse> handleDuplicate(DuplicateEmailException e) {
    return ResponseEntity.status(HttpStatus.CONFLICT)  // 409
        .body(new ErrorResponse("DUPLICATE_EMAIL", e.getMessage()));
}
```

### Mistake 5: Deep Nesting

**The Problem:**
```
❌ WRONG - Too deep
/api/users/{userId}/orders/{orderId}/items/{itemId}/reviews
```

**Why it's bad:**
- Hard to read and understand
- Complicates authorization (check user owns order owns item)
- URL length limits
- Difficult to change hierarchy

**Solution:**
```
✅ CORRECT - Flatten after 2-3 levels
/api/users/{userId}/orders
/api/orders/{orderId}/items
/api/order-items/{itemId}/reviews

// Or use query parameters
/api/reviews?orderItemId={itemId}
```

### Mistake 6: Not Handling Pagination

**The Problem:**
```java
// ❌ WRONG - Returns all records
@GetMapping
public List<User> getAllUsers() {
    return userRepository.findAll();  // Returns 100,000 users!
}
```

**Why it's dangerous:**
- Crashes with large datasets (OOM)
- Slow response times
- DOS attack vector
- Poor user experience

**Solution:**
```java
// ✅ CORRECT - Always paginate
@GetMapping
public PageResponse<UserDto> getUsers(Pageable pageable) {
    // Returns 20 items by default
    Page<UserDto> page = userService.findAll(pageable);
    return PageResponse.of(page);
}

// Or require explicit limits
@GetMapping
public List<User> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") @Max(100) int size) {
    return userService.findAll(PageRequest.of(page, size));
}
```

### Mistake 7: Missing Content-Type

**The Problem:**
```
❌ WRONG - No Content-Type header
POST /api/users

john@example.com  // Plain text body!
```

**Solution:**
```
✅ CORRECT - Always include Content-Type
POST /api/users
Content-Type: application/json

{
  "email": "john@example.com",
  "name": "John Doe"
}
```

### Mistake 8: Not Versioning from Day One

**The Problem:**
```
❌ WRONG - No versioning
/api/users

// Later need breaking change... problem!
```

**Why you need versioning:**
- Can't change existing endpoints without breaking clients
- Forces maintenance of backward compatibility forever
- Hard to evolve API

**Solution:**
```
✅ CORRECT - Version from the start
/api/v1/users

// Later:
/api/v2/users  # New version with breaking changes
/api/v1/users  # Keep old version for 6-12 months
```

### Mistake 9: Exposing Internal Details

**The Problem:**
```json
// ❌ WRONG - Exposes internal error details
{
  "error": "SQLException: ORA-00942: table or view does not exist",
  "stackTrace": "at com.company.UserRepository.findAll(UserRepository.java:25)..."
}
```

**Why it's dangerous:**
- Reveals database structure
- Shows library versions and vulnerabilities
- Helps attackers craft attacks

**Solution:**
```json
// ✅ CORRECT - Generic error in production
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred",
    "requestId": "req-123456"
  }
}

// Log details internally:
// ERROR [req-123456] SQLException: ORA-00942: table or view does not exist
// at com.company.UserRepository.findAll(UserRepository.java:25)
```

---

## Best Practices

### Do
- ✅ Use nouns, not verbs in URLs
- ✅ Use plural for collections
- ✅ Return 201 with Location header on creation
- ✅ Return 204 on successful deletion
- ✅ Use query parameters for filtering, sorting, pagination
- ✅ Version your API from day one
- ✅ Use consistent error formats
- ✅ Include request IDs for tracing
- ✅ Document with OpenAPI/Swagger

### Don't
- ❌ Use verbs in URLs (`/getUsers`, `/createOrder`)
- ❌ Return 200 with error message in body
- ❌ Expose internal error details in production
- ❌ Use camelCase in URLs (use kebab-case)
- ❌ Expose database IDs (use UUIDs)
- ❌ Deep nesting beyond 2-3 levels
- ❌ Ignore HTTP caching headers

### Security
- ❗ Always use HTTPS in production
- ❗ Validate all input data
- ❗ Implement rate limiting
- ❗ Use authentication (OAuth2, JWT)
- ❗ Never return sensitive data (passwords, tokens)
- ❗ Include security headers (CSP, HSTS)
