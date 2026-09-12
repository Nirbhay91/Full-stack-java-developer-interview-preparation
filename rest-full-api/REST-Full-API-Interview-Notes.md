# RESTful API — Interview Notes

## 1. What is REST?
REST (Representational State Transfer) is an architectural style for designing networked applications using HTTP and resource-oriented URLs.

**Interview answer:**
> REST is an architectural style where resources are identified by URIs and manipulated using standard HTTP methods such as GET, POST, PUT and DELETE. REST services are typically stateless and use standard HTTP semantics.

**Memory:** Resource + HTTP + Stateless.

---

## 2. REST vs RESTful
- **REST** = architectural style/principles.
- **RESTful API** = an API designed following REST principles.

---

## 3. Main REST Principles
### Client-Server
Client and server responsibilities are separated.

### Stateless
Every request contains the information needed to process it. The server does not rely on client session state stored between requests.

### Cacheable
Responses should indicate whether they can be cached.

### Uniform Interface
Resources and interactions follow consistent HTTP semantics.

### Layered System
Client may communicate with an intermediary such as API Gateway, proxy or load balancer without needing to know the internal layers.

### Code-on-Demand (optional)
A server may send executable code to the client; this is optional in REST.

**Memory:** C-S-S-C-U-L-C

---

## 4. Resource
A resource is a business entity exposed through the API.

Examples:
```text
/users
/users/101
/orders
/orders/5001/items
```

**Best practice:** Use nouns, not verbs.

Good:
```text
GET /users/101
```

Avoid:
```text
GET /getUser/101
```

---

## 5. HTTP Methods ⭐⭐⭐⭐⭐

### GET
Used to retrieve a resource.
```http
GET /users/101
```

### POST
Usually used to create a new resource or submit data for processing.
```http
POST /users
```

### PUT
Used to replace/update a resource at a known URI; intended to be idempotent.
```http
PUT /users/101
```

### PATCH
Used for partial modification.
```http
PATCH /users/101
```

### DELETE
Used to delete a resource; intended to be idempotent.
```http
DELETE /users/101
```

### HEAD
Like GET for response headers, without a response body.

### OPTIONS
Used to discover supported communication options/methods. Commonly involved in CORS preflight requests.

**Memory:**
- GET → Read
- POST → Create/process
- PUT → Replace
- PATCH → Partial update
- DELETE → Remove

---

## 6. Safe vs Idempotent ⭐⭐⭐⭐

### Safe
A safe method is intended not to change server state.
- GET
- HEAD
- OPTIONS

### Idempotent
Multiple identical requests have the same intended effect as making one request.
- GET
- HEAD
- OPTIONS
- PUT
- DELETE

**POST is generally not idempotent.**

**Important:** Idempotent does not mean the response must be identical every time; it refers to the intended effect on server state.

---

## 7. Path Variable vs Query Parameter

### Path Variable
Identifies a specific resource.
```text
GET /users/101
```

### Query Parameter
Used for filtering, searching, sorting, pagination, etc.
```text
GET /users?department=IT&page=1&size=20
```

**Memory:**
Path = Which resource?
Query = How to filter/control result?

---

## 8. Request Headers
Headers carry metadata about the request.

Common headers:
```text
Authorization
Content-Type
Accept
Idempotency-Key
If-None-Match
```

### Content-Type
Specifies the media type of the request body.

```http
Content-Type: application/json
```

### Accept
Indicates the response media types the client can process.

```http
Accept: application/json
```

**Memory:**
Content-Type = What am I sending?
Accept = What can I receive?

---

## 9. Request Body
Used mainly with POST, PUT and PATCH to send payload data.

Example:
```json
{
  "name": "Nirbhay",
  "email": "test@example.com"
}
```

GET requests can technically have a body in some implementations, but it should not be relied on for standard interoperable API design.

---

## 10. HTTP Status Codes ⭐⭐⭐⭐⭐

### 2xx — Success
- **200 OK** → successful request
- **201 Created** → resource created
- **202 Accepted** → accepted for processing, not necessarily completed
- **204 No Content** → success with no response body

### 4xx — Client Error
- **400 Bad Request** → invalid request
- **401 Unauthorized** → authentication is missing/invalid
- **403 Forbidden** → authenticated but not allowed
- **404 Not Found** → resource not found
- **405 Method Not Allowed** → method not supported for resource
- **409 Conflict** → request conflicts with current resource state
- **415 Unsupported Media Type** → unsupported request content type
- **429 Too Many Requests** → rate limit exceeded

### 5xx — Server Error
- **500 Internal Server Error**
- **502 Bad Gateway**
- **503 Service Unavailable**
- **504 Gateway Timeout**

**Memory:**
2xx = Success
4xx = Client problem
5xx = Server/infrastructure problem

---

## 11. 401 vs 403 ⭐⭐⭐
**401:** Client is not successfully authenticated.

**403:** Client identity may be known, but it does not have permission.

**Memory:**
401 = Who are you?
403 = I know you, but you can't do this.

---

## 12. PUT vs PATCH ⭐⭐⭐⭐
**PUT:** Full replacement semantics for the target resource.

**PATCH:** Partial modification.

Example:
```http
PUT /users/101
```
May send the complete resource representation.

```http
PATCH /users/101
```
May send only:
```json
{
  "email": "new@example.com"
}
```

---

## 13. POST vs PUT ⭐⭐⭐⭐
**POST:** The server typically chooses the new resource URI and POST is generally non-idempotent.

**PUT:** Client targets the resource URI and PUT is intended to be idempotent.

Example:
```http
POST /users
```
```http
PUT /users/101
```

---

## 14. Statelessness
REST statelessness means the server does not need to remember conversational state between requests to understand the current request.

Example with token-based authentication:
```http
Authorization: Bearer <token>
```
The request carries the credentials needed for authentication/authorization.

**Interview answer:**
> Each request should be self-contained with respect to the information required to process it; the server should not depend on prior request state stored in session to understand the current request.

---

## 15. REST vs SOAP ⭐⭐⭐⭐

| REST | SOAP |
|---|---|
| Architectural style | Protocol |
| Commonly uses HTTP | Can use multiple transports |
| Usually JSON, but can use other representations | XML-based messaging |
| Lightweight/simple in many web APIs | More formal messaging standards |
| Statelessness is a core REST constraint | Stateful behavior can be designed |

**Interview line:**
> REST is an architectural style, while SOAP is a protocol with a formal XML messaging model and standards ecosystem.

---

## 16. JSON vs XML
JSON is commonly used in REST APIs because it is compact and convenient for web applications.

XML is more verbose and can be useful where XML-based standards or schema-driven messaging are required.

---

## 17. API Versioning ⭐⭐⭐⭐
Common strategies:

### URI versioning
```text
/api/v1/users
/api/v2/users
```

### Header versioning
```http
Accept: application/vnd.company.user-v2+json
```

### Query parameter
```text
/api/users?version=2
```

Choose a consistent strategy based on API compatibility and organizational standards.

---

## 18. Pagination ⭐⭐⭐⭐
Avoid returning thousands/millions of records in one response.

Example:
```text
GET /users?page=0&size=20
```

Possible response:
```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "totalElements": 150,
  "totalPages": 8
}
```

For very large/high-throughput datasets, cursor/keyset pagination can be more efficient than deep offset pagination.

---

## 19. Filtering, Sorting, Searching
Example:
```text
GET /users?department=IT&status=ACTIVE
```

Sorting:
```text
GET /users?sort=name,asc
```

Searching:
```text
GET /users?search=nirbhay
```

Keep query parameters focused on retrieving/filtering resources.

---

## 20. HATEOAS
Hypermedia As The Engine Of Application State.

A response can include links describing related next actions/resources.

Example:
```json
{
  "id": 101,
  "name": "Nirbhay",
  "links": [
    { "rel": "self", "href": "/users/101" },
    { "rel": "orders", "href": "/users/101/orders" }
  ]
}
```

It is one of the REST constraints, though many practical APIs do not implement full HATEOAS.

---

## 21. Content Negotiation
The client and server negotiate the representation format.

Example:
```http
Accept: application/json
```

The server can select an appropriate representation based on supported media types.

---

## 22. Caching ⭐⭐⭐⭐
REST commonly relies on HTTP caching semantics.

Important headers:
```text
Cache-Control
ETag
Last-Modified
If-None-Match
If-Modified-Since
```

### ETag
A response identifier representing a version/state of the resource.

Client can send:
```http
If-None-Match: "abc123"
```

Server may respond:
```http
304 Not Modified
```

**Benefit:** Reduces bandwidth and server work when cached representation is still valid.

---

## 23. CORS ⭐⭐⭐⭐
CORS = Cross-Origin Resource Sharing.

It controls whether a browser allows web pages from one origin to call resources on another origin.

Typical preflight:
```http
OPTIONS /api/users
Origin: https://frontend.example.com
Access-Control-Request-Method: POST
```

Server can respond with headers such as:
```http
Access-Control-Allow-Origin: https://frontend.example.com
Access-Control-Allow-Methods: GET,POST,PUT,DELETE
```

**Important:** CORS is primarily a browser security mechanism; it is not an authentication mechanism.

---

## 24. Authentication vs Authorization
### Authentication
Who are you?

Example:
```text
JWT / OAuth 2.0 / session
```

### Authorization
What are you allowed to do?

Example:
```text
USER → GET /users
ADMIN → DELETE /users/101
```

**Memory:**
Authentication = Identity
Authorization = Permission

---

## 25. JWT Flow ⭐⭐⭐⭐⭐
Typical flow:
```text
Client
  ↓ login credentials
Auth Server
  ↓ access token
Client
  ↓ Authorization: Bearer <JWT>
API Gateway / Service
  ↓ validate token + authorize
Response
```

JWT commonly contains claims such as subject and roles/authorities, but sensitive secrets should not be placed in a token merely because it is encoded.

---

## 26. API Gateway ⭐⭐⭐⭐⭐
An API Gateway can provide a common entry point to backend services.

Common responsibilities:
- Routing
- Authentication/token checks
- Rate limiting
- TLS termination
- Request/response transformations
- Observability/correlation
- Sometimes aggregation

**Interview answer:**
> Instead of exposing every internal microservice directly to clients, we can use an API Gateway as a controlled entry point that handles cross-cutting concerns and routes requests to the appropriate services.

---

## 27. REST in Microservices
Typical flow:
```text
Frontend
   ↓
API Gateway
   ↓
Order Service
   ↓
Payment Service
```

Services commonly communicate over HTTP/REST when synchronous request-response communication is suitable.

For asynchronous workflows, messaging systems may be preferable.

---

## 28. Response Aggregation
When frontend needs data from multiple services, options include:

```text
Frontend → Gateway/BFF → Service A
                    → Service B
                    → Service C
```

The gateway/BFF can aggregate responses and return a client-specific response.

Benefits:
- Fewer client round trips
- Encapsulates backend topology

Trade-off:
- More responsibility in gateway/BFF
- Increased latency if dependent calls are sequential

---

## 29. Error Response Design ⭐⭐⭐⭐
Avoid returning inconsistent error payloads.

Example:
```json
{
  "timestamp": "2026-09-12T10:15:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "User not found",
  "path": "/users/101",
  "traceId": "abc-123"
}
```

A standardized error format improves client handling and troubleshooting.

---

## 30. Idempotency for POST ⭐⭐⭐⭐
POST is generally non-idempotent. For payment/order APIs, retries can accidentally create duplicates.

A common solution is an idempotency key:
```http
Idempotency-Key: 9f1a2c...
```

The server stores/coordinates the result for the key so a retry does not create another logical operation.

---

## 31. Rate Limiting / Throttling
Controls the number of requests a client can make during a period.

Example:
```text
100 requests/minute/client
```

Benefits:
- Protect backend
- Prevent abuse
- Fair resource usage

Common response:
```text
429 Too Many Requests
```

---

## 32. Timeout / Retry / Circuit Breaker ⭐⭐⭐⭐⭐
In distributed REST calls, never rely on infinite waits.

Use:
- Connection/read timeouts
- Limited retries
- Exponential backoff where appropriate
- Circuit breaker for repeated downstream failures
- Bulkheads/resource isolation

**Important:** Do not blindly retry non-idempotent operations because retries can create duplicate side effects.

---

## 33. REST Exception Handling in Spring Boot ⭐⭐⭐⭐⭐
Typical approach:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<?> handleUserNotFound(
            UserNotFoundException ex) {

        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(ex.getMessage());
    }
}
```

Benefits:
- Centralized exception mapping
- Consistent response structure
- Cleaner controllers

---

## 34. Validation in REST API ⭐⭐⭐⭐⭐
Use Bean Validation for request validation.

```java
public class UserRequest {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

Controller:
```java
@PostMapping("/users")
public ResponseEntity<?> create(
        @Valid @RequestBody UserRequest request) {
    //...
}
```

Common validation annotations:
```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Email
@Min
@Max
@Pattern
```

Invalid request commonly maps to **400 Bad Request**.

---

## 35. Spring Boot REST Controller ⭐⭐⭐⭐⭐
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return service.getUser(id);
    }

    @PostMapping
    public User createUser(@RequestBody UserRequest request) {
        return service.createUser(request);
    }
}
```

Important annotations:
```text
@RestController
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@PathVariable
@RequestParam
@RequestBody
@RequestHeader
```

---

## 36. DTO vs Entity ⭐⭐⭐⭐
Avoid exposing persistence entities directly from public APIs when it creates coupling or leaks internal fields.

Use DTOs:
```text
Request DTO
   ↓
Service
   ↓
Entity
   ↓
Repository
```

And map back:
```text
Entity → Response DTO
```

Benefits:
- API contract isolation
- Security/control over exposed fields
- Easier versioning
- Reduced coupling

---

## 37. API Documentation
Commonly use OpenAPI/Swagger for describing endpoints, request/response models, parameters, authentication requirements and status codes.

Useful for:
- Developer understanding
- Testing/exploration
- Contract visibility

---

## 38. REST Security Best Practices ⭐⭐⭐⭐⭐
- Use HTTPS/TLS.
- Validate and authorize every protected operation.
- Do not put secrets in URLs.
- Validate request payloads.
- Avoid exposing stack traces/internal details.
- Apply rate limits where needed.
- Use short-lived access tokens and secure token handling.
- Log security-relevant events without logging passwords/tokens.

---

# Interview Rapid-Fire ⭐⭐⭐⭐⭐

### Q: What is REST?
> REST is an architectural style for resource-oriented network applications using standard HTTP semantics.

### Q: What is RESTful API?
> An API designed according to REST principles and HTTP resource semantics.

### Q: GET vs POST?
> GET retrieves data and is safe; POST is generally used to create or process data and is not generally idempotent.

### Q: PUT vs PATCH?
> PUT is for replacement semantics; PATCH is for partial modification.

### Q: 401 vs 403?
> 401 means authentication failed or is missing; 403 means the caller is not permitted to perform the operation.

### Q: 400 vs 404?
> 400 means the request is invalid; 404 means the requested resource was not found.

### Q: What is statelessness?
> Each request contains the information required to process it and does not depend on prior conversational state stored on the server.

### Q: What is Content-Type?
> It specifies the media type of the request body.

### Q: Content-Type vs Accept?
> Content-Type describes what the client sends; Accept describes what response media types the client can handle.

### Q: Why API Gateway?
> It provides a common entry point for routing and cross-cutting concerns such as authentication, rate limiting and observability.

### Q: How do you handle duplicate POST requests?
> Use an idempotency key and server-side idempotency handling for operations where retries must not create duplicate side effects.

### Q: How do you handle REST failures in microservices?
> Use timeouts, controlled retries with backoff, circuit breakers, fallback where appropriate, and good observability. Avoid unsafe retries for non-idempotent operations.

### Q: Why DTO?
> To decouple the external API contract from internal persistence models and control exactly what is exposed.

---

# 🧠 Ultimate Memory Map

```text
REST API
│
├── Resource
│   └── /users/101
│
├── HTTP Methods
│   ├── GET     → Read
│   ├── POST    → Create/Process
│   ├── PUT     → Replace
│   ├── PATCH   → Partial Update
│   └── DELETE  → Delete
│
├── Request
│   ├── Path Variable
│   ├── Query Param
│   ├── Header
│   └── Body
│
├── Response
│   ├── Status Code
│   ├── Headers
│   └── Body
│
├── Security
│   ├── HTTPS
│   ├── Authentication
│   └── Authorization
│
├── Production
│   ├── Validation
│   ├── Exception Handling
│   ├── Pagination
│   ├── Caching
│   ├── Rate Limiting
│   ├── Timeout
│   ├── Retry
│   └── Circuit Breaker
│
└── Microservices
    ├── API Gateway
    ├── Service-to-Service REST
    └── Response Aggregation
```

# ⭐ 2-Minute Revision

> **REST is an architectural style based on resources and HTTP semantics. APIs should use nouns for resources and standard HTTP methods such as GET, POST, PUT, PATCH and DELETE. REST is stateless, so each request carries the information needed to process it. I use proper status codes such as 200, 201, 204, 400, 401, 403, 404, 409 and 5xx codes. Content-Type tells the server what is being sent, while Accept tells the server what response representation the client can handle. In production REST APIs, I focus on validation, centralized exception handling, authentication and authorization, pagination, caching, rate limiting, timeouts, safe retries, idempotency for retryable business operations, and observability. In microservices, an API Gateway can provide routing and cross-cutting concerns, while DTOs help keep API contracts decoupled from persistence entities.**