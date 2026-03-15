# API Design

**Last Updated:** 15/03/2026
**Version:** 1.8.0
**Maintained By:** Development Team
**Language:** British English (en_GB)
**Timezone:** Europe/London
**Plugin Scope:** syntek-dev-suite (Python/Django, PHP/Laravel, TypeScript/React, React Native)

---

## Table of Contents

- [Overview](#overview)
- [General Principles](#general-principles)
- [REST API Conventions](#rest-api-conventions)
  - [URL Structure](#url-structure)
  - [HTTP Methods](#http-methods)
  - [Status Codes](#status-codes)
  - [Request and Response Shapes](#request-and-response-shapes)
  - [Pagination](#pagination)
  - [Filtering, Sorting, and Search](#filtering-sorting-and-search)
- [GraphQL Conventions](#graphql-conventions)
  - [Schema Design](#schema-design)
  - [Query Design](#query-design)
  - [Mutation Design](#mutation-design)
  - [Error Handling in GraphQL](#error-handling-in-graphql)
- [Error Response Format](#error-response-format)
- [Authentication and Authorisation in APIs](#authentication-and-authorisation-in-apis)
- [Rate Limiting](#rate-limiting)
- [Versioning](#versioning)
- [Webhooks](#webhooks)
- [API Documentation](#api-documentation)
- [Client-Side Consumption Patterns](#client-side-consumption-patterns)
- [API Design Checklist](#api-design-checklist)

---

## Overview

This guide defines how APIs are designed and consumed across the syntek-dev-suite stack. It covers both REST (Django REST Framework, Laravel) and GraphQL (Strawberry) conventions, plus the patterns used on the client side (React, React Native).

An API is a contract. Changes to it affect every consumer. Design it carefully, document it explicitly, and change it deliberately.

---

## General Principles

1. **APIs are domain contracts, not database mirrors.** Endpoints and types expose what the consumer needs, not the internal schema. See `DATA-STRUCTURES.md` for domain modelling.
2. **Consistency over cleverness.** Every endpoint in a project follows the same naming, pagination, error, and authentication patterns. A consumer who understands one endpoint understands them all.
3. **Fail explicitly.** Every error returns a structured response with enough information for the consumer to understand what went wrong and how to fix it.
4. **Least privilege.** Every endpoint requires authentication unless it is explicitly designed to be public. Every response includes only the data the consumer is authorised to see.
5. **Idempotency.** `GET`, `PUT`, and `DELETE` are idempotent. `POST` is not — but where possible, design `POST` endpoints to handle duplicate submissions gracefully (e.g., idempotency keys for payment creation).

---

## REST API Conventions

### URL Structure

URLs are nouns, not verbs. The HTTP method conveys the action.

```
# Good
GET    /api/v1/orders              # List orders
POST   /api/v1/orders              # Create an order
GET    /api/v1/orders/{id}         # Retrieve an order
PUT    /api/v1/orders/{id}         # Replace an order
PATCH  /api/v1/orders/{id}         # Partially update an order
DELETE /api/v1/orders/{id}         # Delete an order

# Good — nested resources for clear ownership
GET    /api/v1/orders/{id}/lines   # List lines for an order
POST   /api/v1/orders/{id}/lines   # Add a line to an order

# Bad — verbs in URLs
POST   /api/v1/createOrder
GET    /api/v1/getOrderById
POST   /api/v1/orders/{id}/cancel  # Use PATCH with status change instead
```

**Rules:**

- Use plural nouns for resource collections (`/orders`, `/users`, `/bookings`).
- Use kebab-case for multi-word URLs (`/order-lines`, `/payment-methods`).
- Nest resources to a maximum of two levels (`/orders/{id}/lines`). Deeper nesting should use a flat URL with query parameters.
- All API URLs are prefixed with `/api/` and a version identifier (`/api/v1/`).

### HTTP Methods

| Method | Purpose | Idempotent | Request body |
|--------|---------|-----------|--------------|
| `GET` | Retrieve a resource or collection | Yes | No |
| `POST` | Create a new resource | No | Yes |
| `PUT` | Replace a resource entirely | Yes | Yes (complete) |
| `PATCH` | Partially update a resource | No (by convention, treat as Yes) | Yes (partial) |
| `DELETE` | Remove a resource | Yes | No |

- `PUT` requires the full resource in the body. Missing fields are set to their defaults or null.
- `PATCH` requires only the fields being changed.
- `DELETE` returns `204 No Content` on success.
- Do not use `GET` for operations that modify state.

### Status Codes

Use the correct HTTP status code for every response. Do not return `200` for everything.

| Code | Meaning | Use when |
|------|---------|----------|
| `200 OK` | Success | GET, PUT, PATCH success with a response body |
| `201 Created` | Resource created | POST success. Include `Location` header with the new resource URL |
| `204 No Content` | Success, no body | DELETE success, or updates that return no body |
| `400 Bad Request` | Client error — invalid input | Validation failures, malformed JSON |
| `401 Unauthorized` | Not authenticated | Missing or invalid authentication token |
| `403 Forbidden` | Authenticated but not authorised | The user does not have permission for this action |
| `404 Not Found` | Resource does not exist | The requested resource ID is not found |
| `409 Conflict` | State conflict | Duplicate creation, version mismatch |
| `422 Unprocessable Entity` | Validation error | Semantically invalid input (Laravel convention) |
| `429 Too Many Requests` | Rate limit exceeded | Include `Retry-After` header |
| `500 Internal Server Error` | Server error | Unhandled exception (never return internal details) |

**Rules:**

- `400` vs `422`: use `422` for validation errors (the request was syntactically valid but semantically wrong). Use `400` for malformed requests (invalid JSON, wrong content type). Laravel uses `422` by default for validation; DRF uses `400`. Pick one per project and be consistent.
- Never return `200` with an error body. If the operation failed, use a 4xx or 5xx status code.
- Include `Retry-After` (in seconds) with `429` responses.

### Request and Response Shapes

**Responses** use a consistent envelope:

```json
// Single resource
{
  "data": {
    "id": "ord_abc123",
    "status": "confirmed",
    "total": "49.99",
    "currency": "GBP",
    "lines": [
      {
        "product_name": "Widget",
        "quantity": 2,
        "line_total": "29.98"
      }
    ],
    "created_at": "2026-03-15T10:30:00Z"
  }
}

// Collection
{
  "data": [ ... ],
  "meta": {
    "current_page": 1,
    "per_page": 25,
    "total": 148,
    "total_pages": 6
  },
  "links": {
    "first": "/api/v1/orders?page=1",
    "last": "/api/v1/orders?page=6",
    "prev": null,
    "next": "/api/v1/orders?page=2"
  }
}
```

**Rules:**

- Wrap single resources in `{ "data": { ... } }`.
- Wrap collections in `{ "data": [ ... ], "meta": { ... } }`.
- Use ISO 8601 format for all dates and times, always in UTC (`2026-03-15T10:30:00Z`).
- Use strings for monetary values to avoid floating-point precision issues.
- Use snake_case for JSON keys. Transform to camelCase at the client boundary (see `DATA-STRUCTURES.md`).
- Include only the fields the consumer needs. Do not expose internal IDs, database column names, or sensitive data.

### Pagination

All collection endpoints must be paginated. Never return unbounded lists.

**Cursor-based pagination** (preferred for large or frequently changing datasets):

```
GET /api/v1/orders?cursor=eyJpZCI6MTAwfQ&per_page=25
```

**Offset-based pagination** (simpler, acceptable for small datasets):

```
GET /api/v1/orders?page=2&per_page=25
```

**Rules:**

- Default `per_page` is 25. Maximum is 100.
- Include pagination metadata in the `meta` object.
- Include navigation links in the `links` object.
- For cursor-based pagination, the cursor is opaque to the client — do not expose database IDs or offsets in the cursor value.

### Filtering, Sorting, and Search

Use query parameters for filtering and sorting:

```
# Filter by status
GET /api/v1/orders?status=confirmed

# Filter by date range
GET /api/v1/orders?created_after=2026-01-01&created_before=2026-03-31

# Sort (prefix with - for descending)
GET /api/v1/orders?sort=-created_at

# Multiple filters
GET /api/v1/orders?status=confirmed&sort=-created_at&per_page=10

# Search
GET /api/v1/orders?search=widget
```

**Rules:**

- Use the field name as the query parameter name for simple equality filters.
- Use suffixed parameters for range filters (`created_after`, `created_before`, `total_gte`, `total_lte`).
- Use `sort` with field names, prefixed with `-` for descending.
- Use `search` for full-text search across multiple fields.
- Validate all filter parameters. Unknown parameters should be ignored (not cause errors).

---

## GraphQL Conventions

### Schema Design

- Name types after the domain, not the database table. `Order`, not `OrdersTable`.
- Use clear, specific type names. `CreateOrderInput`, not `Input`.
- Every query and mutation must have a description.
- Use `ID` scalar for identifiers, `String` for text, `Decimal` for money, `DateTime` for timestamps.
- Mark fields as non-nullable (`!`) by default. Use nullable only when the absence of a value is meaningful.

```python
@strawberry.type(description="A customer order with its line items.")
class Order:
    id: strawberry.ID
    status: OrderStatus
    total: Decimal
    currency: str
    lines: list["OrderLine"]
    created_at: datetime
```

### Query Design

- Name queries as nouns: `order`, `orders`, `currentUser`.
- Accept filter arguments directly, not wrapped in an input type.
- Return connection types for paginated lists.

```python
@strawberry.type
class Query:
    @strawberry.field(description="Retrieve a single order by ID.")
    def order(self, id: strawberry.ID, info: strawberry.types.Info) -> Order | None:
        ...

    @strawberry.field(description="List orders for the authenticated user.")
    def orders(
        self,
        info: strawberry.types.Info,
        status: OrderStatus | None = None,
        first: int = 25,
        after: str | None = None,
    ) -> OrderConnection:
        ...
```

### Mutation Design

- Name mutations as verbs: `createOrder`, `cancelOrder`, `updateBookingStatus`.
- Accept a single input type for complex mutations.
- Return the affected resource (or a union of success/error types) — never return `Boolean`.

```python
@strawberry.input(description="Input for creating a new order.")
class CreateOrderInput:
    lines: list[OrderLineInput]
    currency: str = "GBP"

@strawberry.type
class Mutation:
    @strawberry.mutation(description="Create a new order.")
    def create_order(self, input: CreateOrderInput, info: strawberry.types.Info) -> Order:
        ...
```

### Error Handling in GraphQL

Use the `errors` array for operational errors. Use union types for expected domain errors that the client needs to handle:

```python
@strawberry.type
class OrderNotFoundError:
    message: str

@strawberry.type
class InsufficientStockError:
    message: str
    product_id: strawberry.ID

OrderResult = strawberry.union("OrderResult", [Order, OrderNotFoundError, InsufficientStockError])

@strawberry.type
class Mutation:
    @strawberry.mutation
    def cancel_order(self, id: strawberry.ID, info: strawberry.types.Info) -> OrderResult:
        ...
```

Clients can then handle each case with `__typename` discrimination.

---

## Error Response Format

All error responses — REST and GraphQL — must use a consistent structure.

### REST errors

```json
{
  "error": {
    "code": "validation_failed",
    "message": "The request contained invalid fields.",
    "details": [
      {
        "field": "email",
        "message": "This field is required."
      },
      {
        "field": "quantity",
        "message": "Must be a positive integer."
      }
    ]
  }
}
```

**Rules:**

- `error.code` is a machine-readable string (snake_case). Clients use this for programmatic error handling.
- `error.message` is a human-readable summary.
- `error.details` is an optional array of field-level errors for validation failures.
- Never include stack traces, file paths, SQL queries, or internal exception messages in error responses. See `SECURITY.md`.

### Django REST Framework

```python
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    if response is not None:
        response.data = {
            "error": {
                "code": _get_error_code(exc),
                "message": _get_error_message(exc),
                "details": _get_error_details(response.data),
            }
        }
    return response
```

### Laravel

```php
// app/Exceptions/Handler.php — or via a custom exception renderer
public function render($request, Throwable $e): JsonResponse|Response
{
    if ($request->expectsJson() && $e instanceof ValidationException) {
        return response()->json([
            'error' => [
                'code'    => 'validation_failed',
                'message' => 'The request contained invalid fields.',
                'details' => collect($e->errors())->map(fn ($messages, $field) => [
                    'field'   => $field,
                    'message' => $messages[0],
                ])->values(),
            ],
        ], 422);
    }

    // ... handle other exception types
}
```

---

## Authentication and Authorisation in APIs

### Token-based authentication (REST)

- Use short-lived JWTs or opaque tokens in the `Authorization: Bearer <token>` header.
- For server-rendered SPAs (Next.js, Inertia.js), use `httpOnly` Secure cookies instead of Bearer tokens. See `SECURITY.md` — Browser Storage Policy.
- Tokens must be validated on every request. Do not cache authentication decisions.
- Return `401` for missing or invalid tokens. Return `403` for valid tokens without sufficient permissions.

### API key authentication

- For service-to-service communication or third-party integrations, use API keys.
- API keys are sent in the `X-API-Key` header, not in query parameters (query parameters appear in logs and browser history).
- Each API key has a defined scope and rate limit.

### Authorisation

- Every endpoint must check authorisation, not just authentication.
- Scope all queries to the authenticated user or tenant. See `DATA-STRUCTURES.md` — Multi-Tenancy Patterns and `SECURITY.md` — IDOR prevention.
- Return `403` with a generic message. Do not reveal whether a resource exists if the user is not authorised to see it — use `404` instead of `403` where appropriate to prevent enumeration.

---

## Rate Limiting

All APIs must be rate limited to prevent abuse and protect infrastructure.

### Default limits

| Endpoint type | Limit | Window |
|--------------|-------|--------|
| Authentication (login, register, password reset) | 5 requests | 1 minute |
| Standard API (authenticated) | 60 requests | 1 minute |
| Search / heavy queries | 20 requests | 1 minute |
| Webhooks (inbound) | 100 requests | 1 minute |
| Public / unauthenticated | 30 requests | 1 minute |

### Response headers

Include rate limit information in every response:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1710500000
```

When the limit is exceeded, return `429 Too Many Requests` with a `Retry-After` header.

### Implementation

**Laravel:**

```php
// routes/api.php
Route::middleware(['auth:sanctum', 'throttle:60,1'])->group(function () {
    Route::apiResource('orders', OrderController::class);
});

Route::middleware('throttle:5,1')->post('/login', [AuthController::class, 'login']);
```

**Django REST Framework:**

```python
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.UserRateThrottle",
        "rest_framework.throttling.AnonRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {
        "user": "60/min",
        "anon": "30/min",
    },
}
```

---

## Versioning

APIs must be versioned from the first release. Breaking changes require a new version.

### URL-based versioning (preferred)

```
/api/v1/orders
/api/v2/orders
```

### What constitutes a breaking change

- Removing a field from a response.
- Renaming a field.
- Changing a field's type.
- Removing an endpoint.
- Changing the authentication mechanism.
- Changing the error response format.

### What is NOT a breaking change

- Adding a new field to a response.
- Adding a new endpoint.
- Adding a new optional query parameter.
- Adding a new optional field to a request body.

### Rules

- Maintain the previous version for at least 6 months after a new version is released.
- Document the deprecation timeline and communicate it to consumers.
- Add a `Deprecation` header to responses from deprecated endpoints.
- In GraphQL, prefer additive schema evolution over versioned endpoints. Deprecate fields with `@deprecated(reason: "Use newField instead")` and remove them in a future release.

---

## Webhooks

For outbound webhooks (notifying external systems of events), follow these rules:

### Payload design

```json
{
  "event": "order.confirmed",
  "timestamp": "2026-03-15T10:30:00Z",
  "data": {
    "id": "ord_abc123",
    "status": "confirmed",
    "total": "49.99",
    "currency": "GBP"
  }
}
```

- Use past-tense event names: `order.confirmed`, `payment.failed`, `user.created`.
- Include a `timestamp` in ISO 8601 UTC.
- Include the affected resource in `data` with enough information for the consumer to process the event without a follow-up API call.

### Security

- Sign every webhook payload with HMAC-SHA256 using a shared secret. Include the signature in a header (`X-Webhook-Signature`).
- Consumers must verify the signature before processing. See `SECURITY.md` — Cryptography and Encryption Standards.
- Use HTTPS for all webhook delivery.
- Implement retry with exponential back-off for failed deliveries (5 attempts over 24 hours).
- Log all webhook delivery attempts and their outcomes.

### Inbound webhooks

- Always verify the sender's signature before processing.
- Return `200` immediately after signature verification and enqueue processing asynchronously. Do not block the response on business logic.
- Idempotency: handle duplicate deliveries gracefully (use the event ID to detect and skip duplicates).

---

## API Documentation

Every API must be documented. Documentation lives alongside the code and is kept in sync.

### REST

- Use OpenAPI 3.0+ (Swagger) for REST API documentation.
- Generate the spec from code where possible (DRF Spectacular, Laravel Scramble or Scribe).
- Include request/response examples for every endpoint.
- Document error responses with the same detail as success responses.
- Host the interactive documentation at `/api/docs` or `/api/v1/docs`.

### GraphQL

- The schema is self-documenting via introspection (development only — see `SECURITY.md`).
- Every type, field, query, and mutation must have a `description`.
- Publish versioned schema documentation separately from live introspection.

---

## Client-Side Consumption Patterns

### API client layer

All API calls should go through a single client module, never called directly from components:

```typescript
// lib/api/client.ts — single point for all API calls
const API_BASE = process.env.NEXT_PUBLIC_API_URL;

async function request<T>(path: string, options?: RequestInit): Promise<T> {
  const response = await fetch(`${API_BASE}${path}`, {
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
    ...options,
  });

  if (!response.ok) {
    const error = await response.json();
    throw new ApiError(response.status, error);
  }

  return response.json();
}

// Typed endpoint functions
export async function getOrders(params?: OrderFilters): Promise<PaginatedResponse<Order>> {
  const query = new URLSearchParams(params as Record<string, string>);
  return request(`/api/v1/orders?${query}`);
}

export async function createOrder(input: CreateOrderInput): Promise<{ data: Order }> {
  return request("/api/v1/orders", {
    method: "POST",
    body: JSON.stringify(input),
  });
}
```

### Error handling on the client

```typescript
class ApiError extends Error {
  constructor(
    public status: number,
    public body: { error: { code: string; message: string; details?: FieldError[] } },
  ) {
    super(body.error.message);
  }

  get code(): string { return this.body.error.code; }
  get fieldErrors(): FieldError[] { return this.body.error.details ?? []; }
}
```

### Data transformation at the boundary

Transform API responses into domain types at the client layer, not in components. See `DATA-STRUCTURES.md` — TypeScript Interfaces as Domain Contracts.

---

## API Design Checklist

Before releasing any API endpoint:

- [ ] URL follows REST conventions (plural nouns, no verbs, correct HTTP methods)
- [ ] Response uses the standard envelope (`{ "data": ... }` or `{ "data": [...], "meta": ... }`)
- [ ] Error responses use the standard format (`{ "error": { "code": ..., "message": ... } }`)
- [ ] Correct HTTP status codes are used for success and error cases
- [ ] Authentication is required (or the endpoint is explicitly documented as public)
- [ ] Authorisation checks scope data to the authenticated user/tenant
- [ ] Pagination is implemented for all collection endpoints
- [ ] Rate limiting is configured
- [ ] Dates are in ISO 8601 UTC format
- [ ] Monetary values are strings, not floats
- [ ] No internal details (stack traces, SQL, file paths) are leaked in error responses
- [ ] The endpoint is documented in OpenAPI or GraphQL schema with descriptions
- [ ] The endpoint has integration tests covering happy path, auth failure, and validation failure
