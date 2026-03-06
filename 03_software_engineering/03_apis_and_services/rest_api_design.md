---
layer: 03_software_engineering
type: engineering
tool: REST
status: growing
tags: [rest, api-design, http, openapi, authentication, pagination]
created: 2026-03-05
---

# REST API Design

## Purpose

REST (Representational State Transfer) is an architectural style for designing networked APIs. It provides a uniform, stateless interface over HTTP that enables loose coupling between clients and servers, independent evolvability, and broad interoperability across languages and platforms. REST API design governs how resources are named, how HTTP semantics are applied, how clients handle errors and pagination, and how identity is verified.

## Architecture

### REST Constraints (Fielding, 2000)

**Stateless**: each request from client to server must contain all information needed to process it. The server holds no session state between requests. Session data lives in client-side tokens (cookies, JWTs) or server-side stores identified by a token in the request. Enables horizontal scaling: any server can handle any request.

**Uniform interface**: four sub-constraints:
1. Resource identification in requests (URIs identify resources, not actions)
2. Manipulation of resources through representations (JSON/XML body, not direct object mutation)
3. Self-descriptive messages (Content-Type, status codes convey meaning)
4. Hypermedia as the engine of application state (HATEOAS — links in responses; rarely implemented in practice)

**Layered system**: client cannot tell whether it is talking to the origin server or an intermediary (proxy, CDN, load balancer). Enables transparent caching and security layers.

**Cacheable**: responses must define themselves as cacheable or non-cacheable. HTTP caching headers (`Cache-Control`, `ETag`, `Last-Modified`) allow intermediaries and clients to reuse responses.

**Client-server**: separation of concerns. Client handles UI; server handles data storage. They evolve independently as long as the interface contract is preserved.

**Code on demand (optional)**: server can return executable code (e.g., JavaScript). Rarely used in REST APIs.

### HTTP Methods and Semantics

| Method | Semantics | Idempotent | Safe | Body |
|--------|-----------|-----------|------|------|
| GET | Retrieve resource | Yes | Yes | No |
| POST | Create resource / trigger action | No | No | Yes |
| PUT | Replace resource entirely | Yes | No | Yes |
| PATCH | Partial update | No* | No | Yes |
| DELETE | Remove resource | Yes | No | No |
| HEAD | Like GET, headers only | Yes | Yes | No |
| OPTIONS | List supported methods | Yes | Yes | No |

*PATCH idempotency depends on patch semantics (JSON Patch vs. JSON Merge Patch).

**Idempotent**: calling N times has the same effect as calling once. Important for safe retries.

**Safe**: has no side effects on server state. Clients and intermediaries can freely retry/cache.

### HTTP Status Codes

**2xx Success**
- `200 OK`: general success
- `201 Created`: resource successfully created; include `Location: /resources/{id}` header
- `202 Accepted`: async processing started; return a job ID or polling URL
- `204 No Content`: success with no response body (DELETE, some PATCHes)

**3xx Redirection**
- `301 Moved Permanently`: resource has a new canonical URL
- `304 Not Modified`: conditional GET; client's cached version is still valid

**4xx Client Errors**
- `400 Bad Request`: malformed syntax or invalid field values
- `401 Unauthorized`: authentication required or token invalid
- `403 Forbidden`: authenticated but not authorized for this resource
- `404 Not Found`: resource does not exist (or deliberately hidden)
- `405 Method Not Allowed`: HTTP method not supported on this endpoint
- `409 Conflict`: state conflict (e.g., optimistic locking failure, duplicate creation)
- `410 Gone`: resource existed but has been deleted permanently
- `422 Unprocessable Entity`: syntactically valid but semantically invalid (validation errors)
- `429 Too Many Requests`: rate limit exceeded; include `Retry-After` header

**5xx Server Errors**
- `500 Internal Server Error`: unexpected server-side failure
- `502 Bad Gateway`: upstream service returned invalid response
- `503 Service Unavailable`: server temporarily overloaded or in maintenance
- `504 Gateway Timeout`: upstream did not respond in time

### URL Structure and Versioning

**Resource naming conventions**:
- Use nouns, not verbs: `/orders`, not `/getOrders`
- Plural collection names: `/users`, `/products`
- Nested resources for clear ownership: `/users/{userId}/orders`
- Avoid deep nesting (> 2 levels); flatten with query parameters or separate top-level endpoints
- Use kebab-case for multi-word path segments: `/billing-accounts`
- Lowercase paths: `/Users` → `/users`

**Versioning strategies**:

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| URI versioning | `/v1/users` | Explicit, easy to route | Pollutes URLs, hard to deprecate |
| Header versioning | `Accept: application/vnd.api.v2+json` | Clean URLs | Less discoverable, harder to test in browser |
| Query param | `/users?version=2` | Easy to test | Not semantic, cache pollution |
| Subdomain | `v2.api.example.com` | Full isolation | Infrastructure overhead |

**Recommendation**: URI versioning for public APIs (most practical); header versioning for internal APIs with controlled clients.

**Deprecation**: use `Sunset` header (`Sunset: Sat, 01 Jan 2027 00:00:00 GMT`) and `Deprecation` header. Log usage of deprecated endpoints. Communicate timeline widely before removal.

## Implementation Notes

### Pagination

**Offset-based pagination**
- `GET /posts?offset=40&limit=20`
- Easy to jump to arbitrary pages
- Problems: records shift if new items are inserted (user sees duplicates or misses items); expensive for large offsets (DB scans and discards N rows)
- Appropriate for: admin interfaces, stable datasets

**Cursor-based (keyset) pagination**
- `GET /posts?after=eyJpZCI6MTAwfQ&limit=20` (cursor = base64 of last-seen key)
- Cursor encodes the last seen row's sort key (e.g., `{id: 100}` or `{created_at: "...", id: ...}`)
- Stable: inserts/deletes do not affect position; efficient: DB uses index seek
- Problems: cannot jump to arbitrary page; cursor is opaque to client
- Appropriate for: infinite scroll feeds, large datasets

**Page-number pagination**
- `GET /posts?page=3&per_page=20`
- Equivalent to offset under the hood; same offset problems
- Familiar UX pattern; fine for small datasets

**Response envelope for pagination**:
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTIwfQ",
    "has_more": true,
    "total": 4821
  }
}
```

### Filtering and Sorting

- Simple filtering: `GET /products?category=electronics&in_stock=true`
- Range filters: `GET /orders?created_after=2026-01-01&created_before=2026-02-01`
- Sorting: `GET /products?sort=price&order=desc` or `sort=-price` (minus = descending)
- Sparse fieldsets: `GET /users?fields=id,name,email` — reduces payload, speeds up response
- Complex queries: POST to a search endpoint with a JSON body (search-as-a-service pattern)

### Content Negotiation

- Client sets `Accept: application/json` or `Accept: application/xml`
- Server responds with `Content-Type: application/json; charset=utf-8`
- If server cannot satisfy `Accept`, return `406 Not Acceptable`
- API versioning via `Accept: application/vnd.myapi.v2+json`

### Error Response Format

Consistent error format reduces client parsing burden. Adopt a standard like [RFC 7807 Problem Details](https://datatracker.ietf.org/doc/html/rfc7807):

```json
{
  "type": "https://api.example.com/errors/validation-error",
  "title": "Validation Error",
  "status": 422,
  "detail": "The request body contains invalid fields.",
  "instance": "/orders/create",
  "errors": [
    { "field": "email", "message": "must be a valid email address" },
    { "field": "quantity", "message": "must be a positive integer" }
  ],
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736"
}
```

Key principles:
- Always return the same structure regardless of error type
- Include `trace_id` / `request_id` for correlation with server logs
- `errors` array for multiple validation failures (not just the first one)
- Machine-readable `type` URI for error classification

### OpenAPI / Swagger

OpenAPI 3.x is the de facto standard for documenting REST APIs:
- Describes endpoints, parameters, request/response schemas, authentication
- Enables code generation (client SDKs, server stubs, mock servers)
- Powers interactive documentation (Swagger UI, Redoc)
- Integrate with FastAPI (auto-generated from Python type hints) or write spec-first

Key sections: `paths`, `components/schemas`, `components/securitySchemes`, `info`, `servers`

### Authentication Patterns

**API Keys**
- Client sends `Authorization: ApiKey <key>` or `X-API-Key: <key>` header
- Simple to implement and use; no expiry without key rotation
- Use for: server-to-server integrations, public APIs with usage tracking
- Store only hashed key in database (treat like a password)

**OAuth 2.0**
- Industry standard for delegated authorization
- Flows: Authorization Code (web/mobile apps), Client Credentials (server-to-server), Device Code (CLIs/TVs)
- Issues short-lived access tokens + long-lived refresh tokens
- Use for: third-party integrations, user-delegated access, scoped permissions

**JWT (JSON Web Token)**
- Self-contained signed token: `{header}.{payload}.{signature}`
- Server verifies signature without database lookup → stateless, scalable
- Payload contains: `sub` (user ID), `exp` (expiry), `iat` (issued at), custom claims (`roles`, `scope`)
- Signing: HS256 (shared secret, single service) vs RS256/ES256 (public/private key, multi-service)
- Stateless revocation is hard: short expiry (15 min) + refresh token rotation is the standard pattern
- Never put sensitive data in payload — it is base64-encoded, not encrypted (use JWE if encryption needed)

## Trade-offs

| Decision | Pro | Con |
|----------|-----|-----|
| URI versioning | Explicit, easy to test and route | URL pollution, hard to sunset gracefully |
| Cursor pagination | Stable, efficient at scale | No random page access, opaque cursor |
| JWT auth | Stateless, scales without session store | Hard to revoke; long-lived tokens are risky |
| Offset pagination | Simple, supports page jumps | Unstable under inserts, slow for large offsets |
| Strict REST (HATEOAS) | Self-discoverable API | Rarely worth the implementation complexity |

## References

- Fielding dissertation (2000): https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- RFC 7807 Problem Details: https://datatracker.ietf.org/doc/html/rfc7807
- RFC 9457 (updated Problem Details): https://datatracker.ietf.org/doc/html/rfc9457
- OpenAPI Specification: https://spec.openapis.org/oas/latest.html
- OAuth 2.0 RFC 6749: https://datatracker.ietf.org/doc/html/rfc6749

## Links
- [[fastapi_patterns|FastAPI]]
