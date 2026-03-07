---
layer: 04_software_engineering
type: pattern
status: growing
tags: [pattern, safety]
created: 2026-03-07
---

# Authentication and Authorization

## Authentication vs Authorization

**Authentication (AuthN):** verifies *who* the caller is. Answers: "Are you really the entity you claim to be?" Mechanisms: passwords, tokens, certificates, biometrics.

**Authorization (AuthZ):** verifies *what* an authenticated caller is allowed to do. Answers: "You are who you say you are — but do you have permission to do this?" Mechanisms: RBAC, ABAC, policy engines (OPA).

The two are orthogonal. A request can be:
- Authenticated but unauthorized (valid token, wrong role).
- Unauthenticated (no token) — should be rejected before authZ even runs.

Never short-circuit: always authenticate first, then authorize. Conflating the two leads to confused deputy problems.

---

## JWT Tokens

### Structure

A JSON Web Token is a Base64URL-encoded triple:

```
<Header>.<Payload>.<Signature>
```

**Header** — algorithm and token type:
```json
{ "alg": "HS256", "typ": "JWT" }
```

**Payload** — registered + custom claims:
```json
{
  "sub": "user_42",
  "iss": "https://auth.example.com",
  "aud": "api.example.com",
  "exp": 1709900000,
  "iat": 1709896400,
  "roles": ["analyst", "viewer"]
}
```

**Signature** (HS256 example):
```
HMACSHA256(base64url(header) + "." + base64url(payload), secret)
```

For public-key schemes (RS256, ES256), the server signs with a private key and verifies with a public key — enabling stateless verification by any party holding the public key.

### Validation checklist

Every JWT consumer must validate **all** of the following:

| Check | Failure mode if skipped |
|---|---|
| Signature valid | Attacker forges arbitrary claims |
| `exp` not passed | Stolen tokens remain valid forever |
| `iss` matches expected issuer | Cross-issuer token injection |
| `aud` matches this service | Token replay across services |
| Algorithm is allowed | `alg: none` attack — unsigned tokens accepted |

```python
import jwt  # PyJWT

ALLOWED_ALGORITHMS = ["RS256"]  # never include "none"

def verify_token(token: str, public_key: str, audience: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            public_key,
            algorithms=ALLOWED_ALGORITHMS,
            audience=audience,
            options={"require": ["exp", "iss", "aud"]},
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise AuthError("Token expired")
    except jwt.InvalidTokenError as e:
        raise AuthError(f"Invalid token: {e}")
```

### Token lifetime and refresh

- **Access token:** short-lived (5–15 min). Sent with every request. Stateless — loss requires waiting for expiry.
- **Refresh token:** long-lived (days–weeks). Stored server-side; can be revoked. Never sent with API requests — only to the token endpoint.

A stolen access token is valid until expiry. Shorten lifetimes when the resource is sensitive.

---

## OAuth2 Flows

### Client Credentials Flow (machine-to-machine)

Used when a service authenticates itself, not a user. There is no human in the loop.

```
Client → POST /token (client_id + client_secret) → Auth Server
Auth Server → access_token → Client
Client → API Request + Bearer token → Resource Server
```

```python
import httpx

resp = httpx.post(
    "https://auth.example.com/oauth/token",
    data={
        "grant_type": "client_credentials",
        "client_id": CLIENT_ID,
        "client_secret": CLIENT_SECRET,
        "scope": "read:data",
    },
)
token = resp.json()["access_token"]
```

Client secrets must be rotated and stored in a secrets manager (see [[secrets_management]]), never in source code.

### Authorization Code Flow (user-facing apps)

Used when a human delegates access to an application. PKCE (Proof Key for Code Exchange) is mandatory for public clients (SPAs, mobile apps) to prevent code interception.

```
Browser → GET /authorize?response_type=code&code_challenge=... → Auth Server
User authenticates in Auth Server
Auth Server → Redirect to app with ?code=... → App
App → POST /token (code + code_verifier) → Auth Server
Auth Server → access_token + refresh_token → App
```

Never use the Implicit Flow (deprecated). Always use Authorization Code + PKCE.

### Token storage

| Client type | Storage | Risk |
|---|---|---|
| Server-side web app | HttpOnly cookie | CSRF (mitigate with SameSite=Strict) |
| SPA | Memory (JS variable) | Lost on refresh; never in `localStorage` |
| Mobile app | Secure keystore | OS-level extraction on rooted devices |

---

## API Key Patterns

API keys are long-lived shared secrets — simpler than OAuth2 but higher risk if leaked.

**Issuance:**
- Generate with a CSPRNG: `secrets.token_urlsafe(32)` (Python) — 256 bits of entropy.
- Issue per client (not shared), with metadata: owner, scopes, expiry.

**Storage (server-side):**
- Never store the raw key. Store a hash: `hashlib.sha256(key.encode()).hexdigest()`.
- On each request, hash the presented key and compare to stored hash.

```python
import hashlib, secrets

def create_api_key() -> tuple[str, str]:
    """Return (raw_key_for_user, hash_for_storage)."""
    raw = secrets.token_urlsafe(32)
    digest = hashlib.sha256(raw.encode()).hexdigest()
    return raw, digest

def verify_api_key(presented: str, stored_hash: str) -> bool:
    digest = hashlib.sha256(presented.encode()).hexdigest()
    return secrets.compare_digest(digest, stored_hash)  # timing-safe
```

**Transmission:** API keys must travel only over TLS. Accept in `Authorization: Bearer <key>` header, never in query parameters (URLs appear in server logs).

---

## RBAC

Role-Based Access Control assigns permissions to roles, and roles to users — decoupling users from permissions.

```
User ──has──▶ Role ──has──▶ Permission ──on──▶ Resource
```

**Design principles:**
- Principle of least privilege: assign the minimum role needed.
- Role explosion: avoid creating a unique role per user; group by job function.
- Separation of duties: the same role should not be able to both initiate and approve a sensitive action.

**Minimal Python enforcement pattern:**

```python
from functools import wraps

ROLE_PERMISSIONS: dict[str, set[str]] = {
    "viewer":  {"data:read"},
    "analyst": {"data:read", "model:run"},
    "admin":   {"data:read", "data:write", "model:run", "model:deploy"},
}

def require_permission(permission: str):
    def decorator(fn):
        @wraps(fn)
        def wrapper(request, *args, **kwargs):
            user_roles = request.user.roles
            allowed = any(
                permission in ROLE_PERMISSIONS.get(r, set())
                for r in user_roles
            )
            if not allowed:
                raise PermissionError(f"Missing permission: {permission}")
            return fn(request, *args, **kwargs)
        return wrapper
    return decorator
```

For complex policies (multi-tenancy, attribute-based rules), delegate to **Open Policy Agent (OPA)** rather than encoding logic in application code.

---

## Common Vulnerabilities

### Token leakage
Tokens logged in plaintext (server logs, error messages, browser history via query params). Mitigate: never log Authorization headers; never put tokens in URLs; scrub sensitive headers in middleware.

### Replay attacks
A stolen token reused before expiry. Mitigate: short-lived access tokens; bind tokens to client IP or TLS channel binding (RFC 8705) for high-value operations; token revocation endpoint for refresh tokens.

### Algorithm confusion (`alg: none` and RS256→HS256)
An attacker modifies the JWT header to `"alg": "none"` and removes the signature, or downgrades RS256 to HS256 and re-signs with the public key as the HMAC secret.
Mitigate: **always** specify allowed algorithms explicitly in `jwt.decode()`; never accept `"none"`.

### Insecure Direct Object Reference (IDOR)
An authenticated user accesses another user's resources by guessing an ID: `GET /reports/1234` → increments to `/reports/1235`. Mitigate: always check that the authenticated user's identity matches the resource owner in the authZ layer; use UUIDs or opaque tokens as resource IDs.

### Overly permissive CORS
`Access-Control-Allow-Origin: *` on an API that accepts cookies or Authorization headers. Mitigate: whitelist specific origins; never combine wildcard origin with `Access-Control-Allow-Credentials: true` (the spec rejects it, but misconfiguration occurs).

### Refresh token rotation not enforced
If refresh tokens are not rotated on each use, a stolen refresh token provides indefinite access. Implement single-use rotation with immediate revocation of the old token.

---

## References

- [RFC 7519 — JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- [RFC 6749 — OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [RFC 7636 — PKCE for OAuth](https://datatracker.ietf.org/doc/html/rfc7636)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [PyJWT documentation](https://pyjwt.readthedocs.io/en/stable/)

---

## Links

- [[04_software_engineering/08_security/index|Security]]
- [[04_software_engineering/04_apis_and_services/index|APIs and Services]]
- [[secrets_management|Secrets Management]] — safe storage and rotation of client secrets and API keys
- [[filesystem_sandboxing|Filesystem Sandboxing]] — related trust boundary enforcement
