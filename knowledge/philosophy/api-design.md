# API Design Philosophy

This document provides company-wide guidance for designing APIs. It establishes principles, conventions, and standards that ensure our APIs are consistent, maintainable, and developer-friendly.

## Audience

- **Primary:** Human developers building and consuming APIs
- **Secondary:** AI agents generating or working with API code

## Scope

This document covers:

- REST APIs
- Webhooks

The following require separate guidance and are **out of scope**:

- GraphQL APIs
- gRPC APIs
- Internal service-to-service authentication (mTLS, service mesh)
- Framework-specific implementation details

## How to Read This Document

This document uses RFC 2119 keywords to indicate requirement levels:

- **MUST** / **MUST NOT**: Absolute requirement or prohibition
- **SHOULD** / **SHOULD NOT**: Recommended, but exceptions may exist with justification
- **MAY**: Optional; implementers can choose

---

## Philosophy

These principles answer: *"What do we value in API design?"*

### 1. Consistency Over Cleverness

APIs SHOULD be predictable. Similar operations SHOULD work similarly across endpoints. A developer who learns one part of the API SHOULD be able to apply that knowledge elsewhere.

Clever solutions that surprise consumers create maintenance burden and integration friction.

### 2. Consumer-First Design

APIs MUST be designed for the developers who consume them, not the systems that implement them. Internal implementation details MUST NOT leak into the API contract.

Consider how the API will be discovered, understood, and used. Optimise for the consumer's experience.

### 3. Explicit Contracts

API behaviour MUST be documented and versioned. Consumers MUST NOT need to rely on implicit assumptions or reverse-engineering to understand how an API works.

If behaviour is not documented, it is not guaranteed.

### 4. Meaningful Errors

Error responses MUST help consumers diagnose and fix problems. Errors SHOULD include:

- A machine-readable error code
- A human-readable message
- Sufficient context to identify the cause

Opaque errors like "Something went wrong" violate this principle.

### 5. Appropriate Granularity

Endpoints SHOULD be neither too coarse nor too fine-grained for their use cases.

- Too coarse: consumers fetch large payloads when they need one field
- Too fine: consumers make dozens of requests to assemble basic data

Design for the common use cases. Consider supporting field selection or expansion for flexibility.

### 6. Backward Compatibility

Breaking changes MUST be managed through versioning and migration paths. Existing consumers MUST NOT break when new features are added.

Additive changes (new optional fields, new endpoints) are safe. Removing or renaming fields, changing types, or altering semantics are breaking changes.

### 7. Secure by Default

Authentication, authorisation, and data protection MUST be built in from the start.

- Endpoints MUST require authentication unless explicitly designed for public access
- Authorisation MUST be enforced at the API layer
- Sensitive data MUST NOT appear in URLs or logs

### 8. Observable

APIs MUST be instrumented for monitoring, debugging, and audit trails.

- Requests SHOULD include correlation IDs for tracing
- Responses SHOULD include request IDs for support reference
- Operations SHOULD be logged for audit purposes

### 9. Idempotent Where Possible

Safe operations SHOULD be repeatable without side effects.

- GET, HEAD, OPTIONS MUST be idempotent
- PUT, DELETE SHOULD be idempotent
- POST MAY be non-idempotent, but consider providing idempotency keys for critical operations

### 10. Documentation as Code

API documentation MUST be generated from and kept in sync with implementation.

- Internal APIs MUST have an OpenAPI specification
- Documentation MUST be versioned alongside the API
- Examples MUST be tested to ensure accuracy

---

## Conventions

These conventions answer: *"How do we do this consistently?"*

Conventions use testable rules that can be validated by linters or code review.

### Resource Naming

Resource names MUST use lowercase letters.

Resource names MUST use hyphens (`-`) to separate words.

Resource names MUST be nouns representing the resource, not verbs representing actions.

Resource names MUST be plural for collection endpoints.

```
✓ DO:   /api/bank-documents
✓ DO:   /api/taxpayers
✓ DO:   /api/system-settings

✗ DON'T: /api/BankDocuments      (not lowercase)
✗ DON'T: /api/bank_documents     (underscores)
✗ DON'T: /api/bankDocuments      (camelCase)
✗ DON'T: /api/getBankDocuments   (verb)
✗ DON'T: /api/taxpayer           (singular for collection)
```

### URL Structure

URLs SHOULD use top-level paths for resources. Nested paths are acceptable when the child resource has no independent existence outside its parent.

```
✓ DO:   /api/transactions?taxpayerId=1
✓ DO:   /api/bank-documents?uploadedBy=1
✓ DO:   /api/bank-documents/1/content   (content has no independent existence)

✗ DON'T: /api/taxpayers/1/transactions  (transactions exist independently)
✗ DON'T: /api/users/1/bank-documents    (creates multiple routes to same resource)
```

URLs MUST NOT include trailing slashes.

```
✓ DO:   /api/taxpayers
✓ DO:   /api/taxpayers?userId=1

✗ DON'T: /api/taxpayers/
✗ DON'T: /api/taxpayers/?userId=1
```

URLs MUST NOT include CRUD action names. Use HTTP methods to indicate the action.

```
✓ DO:   POST   /api/taxpayers      (create)
✓ DO:   GET    /api/taxpayers/1    (read)
✓ DO:   PATCH  /api/taxpayers/1    (update)
✓ DO:   DELETE /api/taxpayers/1    (delete)

✗ DON'T: POST /api/taxpayers/create
✗ DON'T: POST /api/taxpayers/1/update
✗ DON'T: POST /api/taxpayers/1/delete
```

URLs MUST NOT include verbs. Model actions as resources when needed.

```
✓ DO:   POST /api/scripts/1/executions   (create an execution)
✓ DO:   POST /api/scripts/1/status       (with action in body)

✗ DON'T: POST /api/scripts/1/execute
✗ DON'T: GET  /api/scripts/1/run
```

### HTTP Methods

HTTP methods MUST be used according to their semantics:

| Method | Purpose | Idempotent | Request Body | Success Code |
|--------|---------|------------|--------------|--------------|
| GET | Retrieve resource(s) | Yes | No | 200 |
| POST | Create resource | No | Yes | 201 |
| PATCH | Partial update | No | Yes | 200 |
| PUT | Full replacement | Yes | Yes | 200 |
| DELETE | Remove resource | Yes | No | 204 |

GET requests MUST NOT modify server state.

DELETE requests SHOULD be idempotent; deleting an already-deleted resource SHOULD return 204, not 404.

#### PATCH Semantics

PATCH requests MUST use JSON Merge Patch semantics (RFC 7396).

- Include only the fields to be updated
- Sending `null` for a field deletes (unsets) that field
- Omitting a field leaves it unchanged

```json
// Original resource
{ "name": "John", "email": "john@example.com", "phone": "+6421234567" }

// PATCH request body
{ "email": "newemail@example.com", "phone": null }

// Result
{ "name": "John", "email": "newemail@example.com" }
```

This behaviour MUST be documented in the API specification to avoid consumer confusion.

### Response Structure

Collection endpoints MUST return an array `[]`, even when filtered to a single expected result.

```
✓ DO:   GET /api/taxpayers           → []
✓ DO:   GET /api/taxpayers?userId=1  → []  (even if only one match)
✓ DO:   GET /api/addresses?taxpayerId=1 → []  (even if only one possible)
```

Single resource endpoints (addressed by ID) MUST return an object `{}`.

```
✓ DO:   GET /api/taxpayers/1  → {}
```

An empty collection `[]` indicates the user has access but no matching resources exist.

A `403 Forbidden` response indicates the user is not permitted to access the resource.

A `404 Not Found` response indicates a specific resource addressed by ID does not exist.

Related entities SHOULD be returned as IDs by default, not expanded objects.

```json
{
  "id": 1,
  "variableRate": "0.001",
  "taxpayer": 1,
  "setBy": 2
}
```

Expansion of related entities MAY be supported via query parameter. The `expand[]` parameter is the recommended pattern.

```
GET /api/custom-variable-rates?taxpayerId=1&expand[]=setBy
```

```json
{
  "id": 1,
  "variableRate": "0.001",
  "taxpayer": 1,
  "setBy": {
    "id": 2,
    "email": "user@example.com",
    "firstName": "Jane",
    "lastName": "Smith"
  }
}
```

### Query Parameters

Query parameter names MUST use camelCase.

```
✓ DO:   /api/transactions?fromDate=2024-01-01&toDate=2024-03-31
✓ DO:   /api/transactions?taxpayerId=1

✗ DON'T: /api/transactions?from_date=2024-01-01  (snake_case)
✗ DON'T: /api/transactions?FromDate=2024-01-01   (PascalCase)
```

Filtering SHOULD use query parameters on collection endpoints.

#### Filter Operators

Complex filters MUST use LHS (left-hand side) bracket syntax to specify operators.

Format: `field[operator]=value`

| Operator | Meaning | Example |
|----------|---------|---------|
| `eq` | Equals (default if no operator) | `status[eq]=active` or `status=active` |
| `ne` | Not equals | `status[ne]=cancelled` |
| `gt` | Greater than | `amount[gt]=100` |
| `gte` | Greater than or equal | `createdAt[gte]=2024-01-01` |
| `lt` | Less than | `amount[lt]=1000` |
| `lte` | Less than or equal | `createdAt[lte]=2024-12-31` |
| `in` | In list (comma-separated) | `status[in]=active,pending` |
| `nin` | Not in list | `status[nin]=cancelled,archived` |

```
✓ DO:   /api/transactions?amount[gte]=100&amount[lte]=500
✓ DO:   /api/transactions?status[in]=pending,processing
✓ DO:   /api/transactions?createdAt[gte]=2024-01-01

✗ DON'T: /api/transactions?minAmount=100&maxAmount=500  (ambiguous naming)
✗ DON'T: /api/transactions?amount_gte=100               (snake_case operator)
```

#### Sorting

Sorting SHOULD use a `sort` parameter with field name, prefixed with `-` for descending.

```
GET /api/taxpayers?sort=lastName        (ascending)
GET /api/taxpayers?sort=-createdAt      (descending)
```

### Pagination

APIs MUST implement pagination for collection endpoints. Two pagination strategies are supported:

#### Offset-Based Pagination

Use offset-based pagination for low-volume endpoints, admin interfaces, or when total counts are required.

Parameters:

- `page`: The page number (1-indexed)
- `limit`: The number of items per page

```
GET /api/taxpayers?page=2&limit=20
```

Response:

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "totalItems": 156,
    "totalPages": 8
  }
}
```

**Limitations:** Offset-based pagination performs poorly on large datasets (database offset scans become expensive). Items may be skipped or duplicated if the dataset changes between page requests.

#### Cursor-Based Pagination

Use cursor-based pagination for high-volume endpoints, real-time data, or when consistent iteration is critical.

Parameters:

- `limit`: The number of items per page
- `cursor`: Opaque token for the next page (omit for first page)

```
GET /api/transactions?limit=50
GET /api/transactions?limit=50&cursor=eyJpZCI6MTIzfQ
```

Response:

```json
{
  "data": [...],
  "pagination": {
    "limit": 50,
    "nextCursor": "eyJpZCI6MTU2fQ",
    "hasMore": true
  }
}
```

**Advantages:** Stable iteration (no skipped/duplicated items), consistent performance regardless of dataset size. **Limitations:** Cannot jump to arbitrary pages, total count not available without additional query.

#### Choosing a Strategy

| Use Case | Recommended Strategy |
|----------|---------------------|
| Admin dashboards, reports | Offset-based |
| User-facing lists with "Page X of Y" | Offset-based |
| High-velocity feeds (transactions, events) | Cursor-based |
| Infinite scroll interfaces | Cursor-based |
| Datasets > 10,000 items | Cursor-based |

Default `limit` SHOULD be reasonable (e.g., 20-50). Maximum `limit` MUST be enforced to prevent abuse.

### Versioning

API versions MUST be included in the URL path.

```
✓ DO:   /api/v1/taxpayers
✓ DO:   /api/v2/taxpayers

✗ DON'T: /api/taxpayers?version=1  (query parameter)
✗ DON'T: Accept: application/vnd.api.v1+json  (header versioning)
```

Major version increments indicate breaking changes.

When introducing a new version:

- The previous version MUST remain available during a migration period
- Deprecation SHOULD be communicated in advance
- Sunset dates SHOULD be documented

### Authentication and Authorisation

#### Principles

APIs MUST require authentication by default. Public endpoints MUST be explicitly documented as such.

Authorisation MUST enforce the principle of least privilege. Users SHOULD only access resources they own or have explicit permission to view.

Authentication credentials MUST NOT appear in URLs or query parameters.

Failed authentication MUST return `401 Unauthorized`.

Failed authorisation (authenticated but not permitted) MUST return `403 Forbidden`.

#### Patterns

Authentication MUST use the `Authorization` header with Bearer tokens.

```
Authorization: Bearer <token>
```

Tokens SHOULD be JWTs issued by the identity provider.

Token validation MUST occur on every request. Cached validation MAY be used with appropriate TTLs.

### Idempotency Keys

For non-idempotent operations (POST requests), APIs SHOULD support idempotency keys to enable safe retries.

Clients MAY send an `Idempotency-Key` header containing a unique identifier (UUID recommended).

```
POST /api/payments
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{ "amount": "150.00", "currency": "NZD" }
```

When an `Idempotency-Key` is provided:

- The server MUST cache the response for that key
- Subsequent requests with the same key MUST return the cached response
- The cache SHOULD be retained for at least 24 hours
- The key MUST be scoped to the authenticated client

If a request with the same key is received while the original is still processing, the server SHOULD return `409 Conflict`.

```
✓ DO:   Use UUIDs for idempotency keys
✓ DO:   Generate a new key for each distinct operation
✓ DO:   Retry with the same key on network failures

✗ DON'T: Reuse keys across different operations
✗ DON'T: Use predictable or sequential keys
```

### Bulk Operations

For creating or modifying multiple resources in a single request, APIs MAY provide bulk endpoints.

Bulk endpoints MUST use the path suffix `/batch`.

```
POST /api/taxpayers/batch
```

Request body MUST wrap items in an `items` array (not a top-level array) to allow for metadata.

```json
{
  "items": [
    { "firstName": "John", "lastName": "Smith" },
    { "firstName": "Jane", "lastName": "Doe" }
  ]
}
```

Responses MUST indicate the status of each item, preserving order.

```json
{
  "results": [
    { "status": "created", "id": 1 },
    { "status": "error", "errors": [{ "code": "VALIDATION_ERROR", "message": "..." }] }
  ],
  "summary": {
    "total": 2,
    "succeeded": 1,
    "failed": 1
  }
}
```

Bulk operations SHOULD be atomic by default (all succeed or all fail). Non-atomic behaviour MAY be supported via an `atomic` parameter.

```
POST /api/taxpayers/batch?atomic=false
```

### Webhooks

#### Event Naming

Webhook event names MUST use `resource.action` format.

```
✓ DO:   payment.completed
✓ DO:   user.created
✓ DO:   invoice.overdue

✗ DON'T: PaymentCompleted   (PascalCase)
✗ DON'T: payment_completed  (snake_case)
✗ DON'T: onPaymentComplete  (verb prefix)
```

#### Payload Structure

Webhook payloads MUST include:

- `event_id`: Unique identifier for this event instance
- `event`: The event name
- `timestamp`: ISO 8601 timestamp of when the event occurred
- `data`: The event payload

```json
{
  "event_id": "evt_abc123",
  "event": "payment.completed",
  "timestamp": "2024-01-15T14:30:00Z",
  "data": {
    "payment_id": 456,
    "amount": "150.00",
    "currency": "NZD"
  }
}
```

#### Idempotency

Each event MUST include a unique `event_id`.

Webhook consumers SHOULD handle duplicate deliveries idempotently. The same `event_id` MAY be delivered multiple times due to retries.

#### Signatures

Webhook payloads MUST be signed.

Signatures SHOULD use HMAC-SHA256.

The signature SHOULD be included in the `X-Webhook-Signature` header.

```
X-Webhook-Signature: sha256=<hex-encoded-signature>
```

Consumers MUST verify the signature before processing the payload.

#### Retries

Failed webhook deliveries SHOULD be retried with exponential backoff.

Retries MUST stop after a reasonable number of attempts (e.g., 5-10 attempts over 24-48 hours).

Permanent failures (4xx responses except 429) SHOULD NOT be retried.

---

## Standards

These standards answer: *"What must this meet?"*

Standards define specific formats and requirements for compliance.

### Error Response Format

Error responses MUST use a consistent structure across all endpoints.

Error responses MUST contain an `errors` array, even for single errors.

Each error object MUST include:

- `code`: Machine-readable string constant (SCREAMING_SNAKE_CASE)
- `message`: Human-readable description

Each error object SHOULD include when applicable:

- `field`: The field that caused the error (for validation errors)
- `detail`: Additional context or guidance

```json
{
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "Email address is invalid",
      "field": "email",
      "detail": "Must be a valid email address format"
    },
    {
      "code": "VALIDATION_ERROR",
      "message": "Phone number is required",
      "field": "phoneNumber"
    }
  ]
}
```

HTTP status codes MUST be used appropriately:

| Status | Meaning | When to Use |
|--------|---------|-------------|
| 400 | Bad Request | Malformed request syntax, invalid parameters |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not permitted |
| 404 | Not Found | Resource addressed by ID does not exist |
| 409 | Conflict | Request conflicts with current state |
| 422 | Unprocessable Entity | Validation errors on well-formed request |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server error |

### Request and Response Content Types

APIs MUST use JSON as the default content type.

Requests with a body MUST include `Content-Type: application/json`.

Clients SHOULD include `Accept: application/json`.

Responses MUST include `Content-Type: application/json`.

Date and time values MUST use ISO 8601 format in UTC (indicated by the `Z` suffix).

```json
{
  "createdAt": "2024-01-15T01:30:00Z"
}
```

Using UTC for all wire transmission eliminates timezone ambiguity. Clients are responsible for converting to local time for display.

### Rate Limiting

When rate limiting is implemented, responses MUST include rate limit headers:

- `X-RateLimit-Limit`: Maximum requests allowed in the window
- `X-RateLimit-Remaining`: Requests remaining in the current window
- `X-RateLimit-Reset`: Unix timestamp when the window resets

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 47
X-RateLimit-Reset: 1705320000
```

When the rate limit is exceeded, the API MUST return `429 Too Many Requests` with a `Retry-After` header.

```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
```

### API Documentation

Internal APIs MUST have an OpenAPI (Swagger) specification.

The OpenAPI specification MUST be:

- Versioned alongside the API code
- Validated in CI/CD pipelines
- Published to an accessible location for consumers

The specification MUST include:

- All endpoints with request/response schemas
- Authentication requirements
- Error response formats
- Example requests and responses

Documentation SHOULD be generated from the OpenAPI specification to ensure accuracy.

---

## Related Documents

- Go Development Philosophy (`knowledge/philosophy/go-development.md`)
- JavaScript Development Philosophy (`knowledge/philosophy/javascript-development.md`)
- PHP Development Philosophy (`knowledge/philosophy/php-development.md`)
