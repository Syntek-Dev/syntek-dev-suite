# Security

**Last Updated**: 15/03/2026
**Version**: 1.8.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Overview](#overview)
- [Secrets Management](#secrets-management)
- [Authentication and Authorisation](#authentication-and-authorisation)
- [Input Validation and Sanitisation](#input-validation-and-sanitisation)
- [Database Security](#database-security)
- [Cryptography and Encryption Standards](#cryptography-and-encryption-standards)
- [Transport Security](#transport-security)
- [API Security](#api-security)
- [GraphQL Security](#graphql-security)
- [File Upload Security](#file-upload-security)
- [Browser Storage Policy](#browser-storage-policy)
- [Container Security](#container-security)
- [OWASP Top 10:2025 Mitigations](#owasp-top-102025-mitigations)
- [Stack-Specific Security](#stack-specific-security)
  - [Laravel / TALL Stack](#laravel--tall-stack)
  - [Django Stack](#django-stack)
  - [React / TypeScript Stack](#react--typescript-stack)
- [Dependency Security](#dependency-security)
- [Supply Chain Security](#supply-chain-security)
- [Data Classification](#data-classification)
- [Security Logging and Monitoring](#security-logging-and-monitoring)
- [Incident Response (Developer-Facing)](#incident-response-developer-facing)
- [Security Testing](#security-testing)
- [Security Checklist](#security-checklist)

---

## Overview

Security is not optional and not a phase — it is a continuous requirement embedded in every decision. All agents and developers must apply the rules in this document when writing, reviewing, or modifying any code in this project.

The guiding principle is **defence in depth**: multiple independent layers of protection, so that a single failure does not compromise the system.

This document aligns with the [OWASP Top 10:2025](https://owasp.org/Top10/2025/), the [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/), and [NIST SP 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html) for authentication guidance.

---

## Secrets Management

**Never hardcode secrets.** This is an absolute rule with no exceptions.

- All secrets, API keys, tokens, and credentials must be stored in environment variables.
- Environment variable files (`.env.*`) must never be committed to version control. They are listed in `.gitignore`.
- Each environment (development, test, staging, production) uses a separate set of credentials.
- Provide `.env.*.example` files with placeholder values and documentation to show what variables are required — never the real values.
- In production, use a secrets manager (HashiCorp Vault, AWS Secrets Manager, or equivalent) rather than plaintext environment files.
- Rotate all secrets immediately if a breach is suspected.

### Repository hygiene

- Run `git diff --cached` before every commit and verify no secrets are staged.
- Use pre-commit hooks (`gitleaks`, `detect-secrets`, or similar) to block accidental secret commits.
- If a secret is accidentally committed, treat the secret as compromised and rotate it immediately — rewriting git history does not make the secret safe.

---

## Authentication and Authorisation

### Authentication

- Use the framework's built-in authentication system. Do not roll your own.
- Passwords must be hashed using a memory-hard algorithm. Use the following preference order per the [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html):
  1. **argon2id** (preferred) — minimum 19 MiB memory, 2 iterations, 1 degree of parallelism. Use 46 MiB / 1 iteration / 1 parallelism where resources allow.
  2. **scrypt** (when argon2id is unavailable) — minimum N=2^17 (128 MiB), r=8, p=1.
  3. **bcrypt** (legacy systems only) — work factor >= 13. Note: bcrypt has a 72-byte input limit; pre-hash with HMAC-SHA384 and a pepper if longer passwords are needed.
- If FIPS-140 compliance is required, use PBKDF2-HMAC-SHA256 with >= 600,000 iterations.
- Never store plaintext or reversibly encrypted passwords.
- Consider using a [pepper](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#peppering) stored separately from the database for additional defence in depth.
- Enforce strong password requirements: minimum 12 characters, reject common passwords (use a blocklist). Do not impose composition rules (mandatory uppercase, special characters) — they reduce usability without meaningfully improving security. Do not require periodic password changes unless a breach is detected (per NIST SP 800-63B).
- Allow all characters including Unicode and whitespace. Do not silently truncate passwords.
- Implement multi-factor authentication (MFA/2FA) for admin and privileged user accounts.
- Session tokens must be regenerated on privilege change (login, role change, password reset).
- Implement account lockout or exponential back-off after repeated failed login attempts.
- Upgrade work factors over time. When a user authenticates, re-hash their password with current parameters if the stored hash uses older, weaker settings.

### Authorisation

- Implement role-based access control (RBAC) or policy-based authorisation using the framework's built-in tools.
- Apply the **principle of least privilege**: every user, role, and API key has only the permissions it needs and nothing more.
- Authorisation checks must happen on the server — never trust client-side role claims.
- Every API endpoint must explicitly declare who can access it. A missing policy is a bug, not a valid "open to all" state.
- **IDOR prevention**: always scope queries to the authenticated user's resources. Never trust user-supplied IDs alone.

```php
// WRONG — attacker can access any order by guessing the ID
$order = Order::find($request->order_id);

// CORRECT — scoped to the authenticated user
$order = Order::where('user_id', auth()->id())->findOrFail($request->order_id);
```

```python
# WRONG
order = Order.objects.get(id=order_id)

# CORRECT
order = Order.objects.get(id=order_id, user=request.user)
```

---

## Input Validation and Sanitisation

Assume all external input is hostile until proven otherwise. This includes:

- HTTP request bodies, query strings, and URL parameters.
- Uploaded files.
- Webhook payloads.
- Third-party API responses.
- Data from databases when it originated from user input.

### Rules

1. **Validate before use.** Reject invalid input at the earliest possible boundary.
2. **Use framework validation.** Use Laravel's Form Requests, Django's serialisers/forms, or Zod/Yup schemas in TypeScript. Do not write custom validation from scratch.
3. **Allowlist, not blocklist.** Specify what is permitted, not what is forbidden.
4. **Sanitise output, not input.** Do not strip HTML on input; escape it on output. This preserves the original data for auditing while preventing injection on display.
5. **Validate file uploads:** check MIME type via content inspection (not file extension), enforce maximum file size, store uploads outside the webroot, and scan for malware in production. See [File Upload Security](#file-upload-security) for the full policy.

---

## Database Security

- **Parameterised queries always.** Never concatenate user input into SQL strings. Use the ORM or prepared statements.
- **Principle of least privilege for database users.** The application database user should have only `SELECT`, `INSERT`, `UPDATE`, `DELETE` on the tables it needs — not `DROP`, `CREATE`, or `GRANT`.
- Use separate database users for the application and for migrations/admin tasks.
- Encrypt sensitive columns at rest (PII, payment data, health records) using column-level encryption. See [Cryptography and Encryption Standards](#cryptography-and-encryption-standards) for approved algorithms.
- Never log raw SQL queries in production — they may contain sensitive parameter values.
- Run `EXPLAIN` on slow queries before adding indexes. Do not add indexes speculatively.

---

## Cryptography and Encryption Standards

Use well-established, vetted cryptographic libraries. Never implement cryptographic primitives from scratch.

### Approved algorithms

| Purpose | Algorithm | Notes |
|---------|-----------|-------|
| Password hashing (1st choice) | argon2id | Minimum: m=19456 (19 MiB), t=2, p=1. Recommended: m=47104 (46 MiB), t=1, p=1. High security: m=131072 (128 MiB), t=3, p=4. |
| Password hashing (2nd choice) | scrypt | When argon2id is unavailable. Minimum: N=2^17, r=8, p=1 (128 MiB). |
| Password hashing (3rd choice / legacy) | bcrypt | Legacy systems only. Work factor >= 13 (2026 minimum). 72-byte input limit — pre-hash with HMAC-SHA384 + pepper if needed. |
| Password hashing (FIPS-140) | PBKDF2-HMAC-SHA256 | Only when FIPS-140 compliance is required. >= 600,000 iterations. |
| Symmetric encryption | AES-256-GCM, ChaCha20-Poly1305 | Always use authenticated encryption (AEAD) |
| Hashing (non-password) | SHA-256, SHA-384, SHA-512, BLAKE3 | For checksums, HMACs, content addressing |
| Key derivation | HKDF, PBKDF2 (>= 600,000 iterations) | HKDF preferred; PBKDF2 only when HKDF is unavailable or FIPS is required |
| Asymmetric encryption | RSA-OAEP (>= 2048-bit), X25519/Ed25519 | Prefer Curve25519 for new projects |
| Digital signatures | Ed25519, RSA-PSS (>= 2048-bit) | Ed25519 preferred |
| HMAC | HMAC-SHA256, HMAC-SHA512 | For message authentication, webhook verification |

### Banned algorithms

These must never be used for security purposes. Their presence in a codebase is a security finding:

- **MD5** — broken collision resistance. Acceptable only for non-security checksums (e.g., cache busting) where collision resistance is not required.
- **SHA-1** — broken collision resistance. Do not use for signatures, certificates, or integrity verification.
- **DES / 3DES** — insufficient key length.
- **AES-ECB** — no diffusion; identical plaintext blocks produce identical ciphertext blocks. Always use a mode with an IV/nonce (GCM, CBC with HMAC, CTR).
- **RC4** — multiple known vulnerabilities.
- **Blowfish** (for encryption) — use AES-256-GCM instead. Note: bcrypt (based on Blowfish) remains acceptable for password hashing in legacy systems only (see [Authentication](#authentication)).

### Key management

- Encryption keys are secrets and must follow all rules in [Secrets Management](#secrets-management). Never hardcode keys.
- Encryption keys must be separate from application secrets (database passwords, API keys). A compromised API key should not expose encrypted data.
- Use a dedicated key management system (HashiCorp Vault, AWS KMS, or equivalent) in production.
- Rotate encryption keys on a defined schedule. For symmetric keys protecting data at rest, support key versioning so that old data can be decrypted with the old key and re-encrypted with the new one during rotation.
- When a key is retired, ensure all data encrypted with that key is re-encrypted before the old key is destroyed.
- Zeroize keys in memory after use where the language permits (Python: limited by garbage collection, but clear variables explicitly; TypeScript: limited by runtime, but overwrite buffers).

### Rules

- Always use authenticated encryption (AEAD). Encrypt-then-MAC is acceptable if AEAD is unavailable, but MAC-then-encrypt and encrypt-only are not.
- Never reuse a nonce/IV with the same key. For AES-256-GCM, use a 96-bit random nonce per encryption operation. If nonce collision risk is unacceptable (high-volume systems), use AES-256-GCM-SIV or XChaCha20-Poly1305 with a 192-bit nonce.
- Ciphertext must include the nonce and authentication tag. The decryption function must verify the tag before returning plaintext.
- Use constant-time comparison for HMAC verification and token comparison to prevent timing attacks. In Python: `hmac.compare_digest()`. In TypeScript: `crypto.timingSafeEqual()`. In PHP: `hash_equals()`.

---

## Transport Security

All communication between clients, services, and backing systems must be encrypted in transit.

### TLS configuration

- **Minimum version: TLS 1.2.** Prefer TLS 1.3 where supported. TLS 1.0 and 1.1 are deprecated and must be disabled.
- Disable insecure cipher suites: RC4, DES, 3DES, export ciphers, NULL ciphers.
- Prefer forward-secrecy cipher suites (ECDHE) so that a compromised server key does not expose past traffic.
- Certificates must be obtained from a trusted Certificate Authority. Self-signed certificates are acceptable only in local development.

### HSTS (HTTP Strict Transport Security)

Enable HSTS on all production deployments:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

- Set `max-age` to at least one year (31536000 seconds).
- Include `includeSubDomains` to cover all subdomains.
- Submit to the HSTS preload list for browser-level enforcement where appropriate.
- Never deploy HSTS without first confirming that all resources (assets, APIs, subdomains) are accessible over HTTPS.

### Internal service communication

- Services communicating within a private network should still use TLS or mTLS (mutual TLS). "Internal" does not mean "trusted".
- If mTLS is impractical, use encrypted tunnels (WireGuard, SSH tunnels) between services.
- Never send credentials, tokens, or PII over unencrypted channels, even within a private network.

---

## API Security

- **Authenticate every endpoint** unless it is explicitly designed to be public (e.g., a public product catalogue).
- **Rate limit all endpoints**, especially authentication endpoints. Use exponential back-off for repeated failures.
- **Return consistent error shapes.** Do not leak internal details (stack traces, file paths, database errors) in error responses.
- Set appropriate HTTP security headers on all responses:

| Header | Value | Purpose |
|--------|-------|---------|
| `Content-Security-Policy` | Restrictive policy | Prevents XSS |
| `X-Content-Type-Options` | `nosniff` | Prevents MIME sniffing |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevents clickjacking |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforces HTTPS |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limits referrer leakage |
| `Permissions-Policy` | Restrictive policy | Limits browser feature access |

- **CORS:** explicitly configure allowed origins. Do not use wildcard `*` for authenticated APIs. List specific origins. If the API serves multiple frontends, validate the `Origin` header against a server-side allowlist.
- **CSRF protection:** enable CSRF tokens for all state-changing requests in server-rendered applications. Verify `Origin` or `Referer` headers for SPA/API deployments.
- **Webhook verification:** always verify webhook signatures before processing. Reject unsigned or incorrectly signed payloads. Use constant-time comparison (see [Cryptography and Encryption Standards](#cryptography-and-encryption-standards)).

---

## GraphQL Security

GraphQL exposes a single endpoint and allows clients to construct arbitrary queries. Without explicit controls, this creates a distinct attack surface from REST.

### Query depth limiting

Set a maximum query depth to prevent deeply nested queries that exhaust server resources:

```python
# Strawberry (Python)
import strawberry
from strawberry.extensions import QueryDepthLimiter

schema = strawberry.Schema(
    query=Query,
    mutation=Mutation,
    extensions=[QueryDepthLimiter(max_depth=10)],
)
```

Choose a depth limit based on the schema's actual needs. Start restrictive (8-12) and increase only if legitimate queries require it.

### Query complexity analysis

Depth alone is insufficient — a shallow query can still be expensive if it requests large lists at multiple levels. Assign cost weights to fields and reject queries that exceed a total complexity budget:

```python
# Strawberry cost annotation example
@strawberry.type
class Query:
    @strawberry.field(extensions=[Cost(weight=10)])
    def orders(self, first: int = 10) -> list[Order]:
        ...
```

### Disable introspection in production

Introspection exposes the entire schema, making reconnaissance trivial. Disable it in production and staging:

```python
# Strawberry
schema = strawberry.Schema(
    query=Query,
    extensions=[DisableIntrospection()],  # enable only in development
)
```

If external clients need schema documentation, publish it through versioned documentation rather than live introspection.

### Field-level authorisation

Every resolver that returns sensitive data or performs a mutation must check the caller's permissions. A schema that exposes a `user.email` field must verify that the requesting user has permission to see that email — not just that they are authenticated.

```python
@strawberry.type
class User:
    @strawberry.field
    def email(self, info: strawberry.types.Info) -> str:
        if info.context.user.id != self.id and not info.context.user.is_admin:
            raise PermissionError("Cannot access another user's email")
        return self._email
```

### Batching and alias attack prevention

GraphQL allows multiple operations in a single request and query aliases that duplicate the same field or mutation. Without limits, an attacker can batch hundreds of login attempts in one request, bypassing rate limits:

```graphql
# Attack: 100 login attempts in a single request
mutation {
  a1: login(email: "user@example.com", password: "attempt1") { token }
  a2: login(email: "user@example.com", password: "attempt2") { token }
  # ... repeated 100 times
}
```

Mitigations:

- Limit the number of aliases per query.
- Limit the number of operations per request (or disable batched queries entirely unless needed).
- Apply rate limiting per mutation type, not just per HTTP request.

### Persisted queries

For production APIs, consider using persisted queries (a whitelist of pre-approved query hashes). This prevents arbitrary query construction entirely and eliminates most GraphQL-specific attack vectors.

---

## File Upload Security

File uploads are a high-risk attack surface. Every project that accepts file uploads must follow these rules.

### Validation

- **Content-type validation via magic bytes.** Do not trust the `Content-Type` header or the file extension. Read the file's magic bytes (file signature) to determine the actual type. In Python, use `python-magic`. In Node.js, use `file-type`.
- **Allowlist permitted types.** Define an explicit list of accepted MIME types per upload field. Reject everything else.
- **Enforce maximum file size.** Set limits at the web server level (Nginx `client_max_body_size`), the application framework level, and the storage layer. Defence in depth.
- **Filename sanitisation.** Never use the original filename for storage. Generate a random filename (UUID or similar) and store the original name as metadata if needed. If the original name must be preserved, strip path separators, null bytes, Unicode control characters, and double extensions (e.g., `image.php.jpg`).

### Storage

- **Store uploads outside the webroot.** Uploaded files must never be directly accessible via a URL that maps to the filesystem. Serve them through an application endpoint that checks authorisation.
- **Use signed URLs for private files.** Generate time-limited, signed URLs that expire after a short period (minutes, not hours). In Django, use a view with authentication. In Laravel, use `Storage::temporaryUrl()`. In S3-compatible storage, use pre-signed URLs.
- **Separate storage from application servers.** Use object storage (S3, MinIO, or equivalent) rather than the local filesystem where possible. This prevents path traversal attacks and simplifies scaling.

### Processing

- **Never execute uploaded files.** Ensure the storage location has no execute permissions. Configure the web server to serve uploaded files with `Content-Disposition: attachment` and an explicit, safe `Content-Type`.
- **Strip metadata from images.** EXIF data can contain GPS coordinates, device information, and other PII. Strip it on upload using a library like `Pillow` (Python) or `sharp` (Node.js).
- **Scan for malware in production.** Use ClamAV or an equivalent scanner for any project that accepts uploads from untrusted users.

---

## Browser Storage Policy

Client-side storage is visible to any JavaScript running on the page, including third-party scripts and XSS payloads. Store data in the most restrictive location appropriate.

### Authentication tokens

- **Always use `httpOnly`, `Secure`, `SameSite=Lax` (or `Strict`) cookies** for authentication tokens (session IDs, JWTs used for auth). These cookies are inaccessible to JavaScript, protecting them from XSS.
- **Never store authentication tokens in `localStorage` or `sessionStorage`.** Both are accessible to any JavaScript on the page.

### Other data

| Storage | Acceptable use | Never store |
|---------|---------------|-------------|
| `httpOnly` Secure cookies | Auth tokens, session IDs, CSRF tokens | - |
| `sessionStorage` | Ephemeral UI state (form wizard progress, scroll position) that is not sensitive | Tokens, PII, secrets |
| `localStorage` | Non-sensitive user preferences (theme, language, dismissed notices) | Tokens, PII, secrets, anything that grants access |
| `IndexedDB` | Large client-side datasets (offline-first apps, cached content) | Tokens, PII, secrets |

### Rules

- If a value grants access to anything (a token, a session, an API key), it must be in an `httpOnly` cookie. No exceptions.
- If a value contains PII (name, email, address, health data), it must not be stored client-side at all unless the application is explicitly designed for offline use — and even then, it must be encrypted at rest in IndexedDB with a key derived from user authentication.
- `SameSite=Lax` is the minimum for all cookies. Use `SameSite=Strict` where the cookie does not need to be sent on cross-site navigations.
- Set an explicit `Path` and `Domain` on cookies to prevent them being sent more broadly than intended.
- Set `Secure` on all cookies. There is no valid reason for a cookie to be sent over HTTP in production.

---

## Container Security

All projects using Docker or OCI containers must follow these rules.

### Image security

- **Use minimal base images.** Prefer `*-slim`, `alpine`, or distroless images. Fewer packages mean fewer vulnerabilities.
- **Pin base image versions.** Use digest pinning (`FROM python:3.12-slim@sha256:abc...`) or at minimum pin to a specific tag. Never use `latest` in production Dockerfiles.
- **Scan images for vulnerabilities** before deployment. Use `trivy`, `grype`, or the Docker Scout integration. Block deployment of images with critical or high-severity CVEs that have available fixes.
- **Do not install unnecessary tools** in production images. Build tools, compilers, debuggers, and package managers should be in a multi-stage build step and excluded from the final image.

### Runtime security

- **Run as a non-root user.** Every Dockerfile must include a `USER` directive that switches to a non-root user before the `CMD` or `ENTRYPOINT`.

```dockerfile
RUN addgroup --system app && adduser --system --ingroup app app
USER app
```

- **Use read-only filesystems** where possible. Mount the container's root filesystem as read-only and use explicit volume mounts for directories that need write access (temp files, logs, uploads).

```yaml
# docker-compose.yml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
    volumes:
      - uploads:/app/uploads
```

- **Never run containers in privileged mode.** `--privileged` disables all security isolation. If a container needs specific capabilities, grant only what is needed with `--cap-add` and drop all others with `--cap-drop ALL`.
- **Do not mount the Docker socket** into application containers. Access to `/var/run/docker.sock` grants root-equivalent access to the host.

### Network isolation

- **Isolate services by network.** Place services that do not need to communicate on separate Docker networks. The database should not be on the same network as the public-facing web server unless the web server needs direct database access.
- **Do not expose ports unless required.** Use `expose` (container-to-container) rather than `ports` (host-bound) where possible. Only bind to `127.0.0.1` if the port must be accessible from the host but not externally.

### Secrets in containers

- **Never bake secrets into images.** Do not use `ENV` in Dockerfiles for secrets, and never `COPY` secret files into the image. Use runtime environment variables, mounted secret files, or a secrets manager.
- Build arguments (`ARG`) are stored in image layers and are visible in `docker history`. Never pass secrets as build arguments.

---

## OWASP Top 10:2025 Mitigations

The [OWASP Top 10:2025](https://owasp.org/Top10/2025/) is the eighth edition, released November 2025. It introduces two new categories (Software Supply Chain Failures and Mishandling of Exceptional Conditions), consolidates SSRF into Broken Access Control, and re-ranks several existing categories to reflect current data and threat trends. The 2025 edition shifts focus from symptoms to root causes.

| # | Category | Mitigation |
|---|----------|-----------|
| **A01:2025** | **Broken Access Control** | RBAC/policy enforcement on every endpoint; scope all queries to authenticated user; validate SSRF targets against allowlists; block requests to private IP ranges |
| **A02:2025** | **Security Misconfiguration** | Review default framework settings; disable debug mode and unnecessary features in production; remove default credentials; harden IaC templates; review cloud permissions for unnecessary public exposure |
| **A03:2025** | **Software Supply Chain Failures** | Pin dependency versions with lock files; verify package integrity and provenance; audit transitive dependencies; scan for malicious packages; secure CI/CD pipelines (see [Supply Chain Security](#supply-chain-security)) |
| **A04:2025** | **Cryptographic Failures** | Use approved algorithms only (see [Cryptography section](#cryptography-and-encryption-standards)); TLS everywhere; encrypt PII at rest; use memory-hard password hashing (argon2id -> scrypt -> bcrypt); never use MD5, SHA-1, DES, or ECB |
| **A05:2025** | **Injection** | Parameterised queries always; framework validation on all inputs; escape/encode output based on context (HTML, JavaScript, URL, SQL); disable dangerous interpreter features |
| **A06:2025** | **Insecure Design** | Threat model new features; apply least privilege; security review before launch; use secure design patterns; separate trust boundaries |
| **A07:2025** | **Authentication Failures** | Framework auth with memory-hard hashing; regenerate sessions on login; enforce MFA for admin; implement credential stuffing protection; do not require periodic password changes (per NIST) |
| **A08:2025** | **Software and Data Integrity Failures** | Pin dependency versions; verify package integrity; use signed commits; validate CI/CD pipeline integrity; verify webhook signatures |
| **A09:2025** | **Security Logging & Alerting Failures** | Log all auth events, admin actions, and errors; configure alerting for suspicious patterns; ensure logs are immutable and tamper-resistant (see [Security Logging](#security-logging-and-monitoring)) |
| **A10:2025** | **Mishandling of Exceptional Conditions** | Handle all error paths explicitly; fail closed (deny by default) rather than open; do not leak sensitive data in error messages; test error handling paths including unexpected inputs and resource exhaustion |

---

## Stack-Specific Security

### Laravel / TALL Stack

**Enable all built-in protections — do not disable them:**

```php
// CSRF protection is enabled by default — never exclude routes without good reason
// Do not add routes to the $except array in VerifyCsrfToken unless absolutely necessary
```

**Mass assignment protection:** always use `$fillable` or `$guarded` on models. Never use `$guarded = []`.

```php
// WRONG
protected $guarded = [];

// CORRECT
protected $fillable = ['name', 'email', 'role'];
```

**Authorisation policies:** every controller action that modifies data must call `$this->authorize()` or use a policy.

```php
public function update(Request $request, Order $order): JsonResponse
{
    $this->authorize('update', $order); // throws 403 if user cannot update this order
    // ...
}
```

**Query scopes for user isolation:**

```php
// Apply global scopes or always chain ->where('user_id', auth()->id())
// Use model observers or policies to prevent cross-user data access
```

**Signed URLs for file access:** never expose direct storage URLs for private files. Use Laravel's `Storage::temporaryUrl()` with short expiry times.

**Rate limiting:** apply `throttle` middleware to all authentication and sensitive endpoints.

```php
Route::middleware(['auth:sanctum', 'throttle:60,1'])->group(function () {
    // authenticated routes
});
Route::middleware('throttle:5,1')->post('/login', [AuthController::class, 'login']);
```

### Django Stack

**Security settings that must be enabled in staging and production:**

```python
# settings/production.py
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
X_FRAME_OPTIONS = "DENY"
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
```

**Never disable CSRF protection on views that handle state-changing requests.**

**DRF permissions:** always set a default permission class and override per view where needed:

```python
# settings/base.py
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": ["rest_framework.permissions.IsAuthenticated"],
    "DEFAULT_AUTHENTICATION_CLASSES": ["rest_framework_simplejwt.authentication.JWTAuthentication"],
    "DEFAULT_THROTTLE_CLASSES": ["rest_framework.throttling.UserRateThrottle"],
    "DEFAULT_THROTTLE_RATES": {"user": "1000/day"},
}
```

**Object-level permissions:** always check `self.check_object_permissions(request, obj)` in views, or use Django Guardian for object-level ACL.

**SECRET_KEY:** never hardcode. Load from environment:

```python
import os
SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]  # raises KeyError if not set — intentional
```

### React / TypeScript Stack

**Never trust client-side data for security decisions.** All authorisation happens on the server.

**XSS prevention:**
- Never use `dangerouslySetInnerHTML` unless the input is sanitised server-side.
- If HTML sanitisation is required client-side, use `DOMPurify`.

```tsx
// WRONG
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// CORRECT — only if absolutely necessary and input is sanitised
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

**Sensitive data in state:** never store tokens, session cookies, or PII in `localStorage`. Use `httpOnly` cookies set by the server for authentication tokens. See [Browser Storage Policy](#browser-storage-policy) for the full policy.

**Environment variables:** Next.js/React env vars prefixed with `NEXT_PUBLIC_` or `VITE_` are exposed to the client bundle. Never put secrets in these variables.

```bash
# WRONG — exposed to the browser
NEXT_PUBLIC_STRIPE_SECRET_KEY=sk_live_...

# CORRECT — server-side only
STRIPE_SECRET_KEY=sk_live_...
```

**Content Security Policy:** configure CSP headers in the Next.js middleware or server configuration. Start restrictive and loosen only as needed.

---

## Dependency Security

Run security audits regularly and before every deployment:

```bash
# Laravel / PHP
composer audit

# Django / Python
pip-audit  # or: safety check

# Node.js / TypeScript
npm audit
# or: pnpm audit
```

**When a vulnerability is found:**

1. Check if your usage is actually affected by the vulnerability.
2. Update to the patched version immediately if affected.
3. If no patched version exists, evaluate a mitigation or replacement.
4. Do not leave known vulnerabilities unresolved in production.

**Dependency update policy:**

- Run security audits weekly (automate with GitHub Dependabot, Renovate, or equivalent).
- Apply security patches within 7 days of release for critical/high severity.
- Review and test all dependency updates before merging to production.

---

## Supply Chain Security

Dependency auditing catches known vulnerabilities, but supply chain attacks target the delivery mechanism itself — compromised packages, hijacked maintainer accounts, and tampered build pipelines.

### Lock file integrity

- **Commit lock files** (`pnpm-lock.yaml`, `poetry.lock`, `Cargo.lock`, `composer.lock`) to version control. Lock files pin the exact dependency tree, including transitive dependencies.
- **Review lock file diffs** in code review. A lock file change should correspond to an intentional dependency update. Unexplained changes are a red flag.
- **Run `--frozen-lockfile` in CI** (pnpm) or equivalent (`pip install --require-hashes`, `cargo install --locked`). This ensures CI installs exactly what was committed, not a silently updated version.

### Commit signing

- Sign commits with GPG or SSH keys. This verifies that a commit was authored by the claimed person and has not been tampered with in transit.
- Require signed commits on protected branches where the project's hosting platform supports it.
- Store signing keys securely. Do not share private keys between developers.

### CI/CD pipeline security

- **Treat CI configuration as security-critical code.** Changes to workflow files (`.github/workflows/`, `.gitlab-ci.yml`, Jenkinsfile) must go through the same review process as application code.
- **Pin CI action versions** by commit SHA, not tag. Tags can be moved; commit SHAs cannot.

```yaml
# WRONG — tag can be moved to a compromised version
- uses: actions/checkout@v4

# CORRECT — pinned to a specific, verified commit
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
```

- **Limit CI permissions.** CI runners should have the minimum permissions needed. Do not grant write access to production secrets, deployment credentials, or package registries unless the specific job requires it.
- **Do not run CI on untrusted code** with elevated permissions. Pull requests from forks should run in a restricted context without access to repository secrets.

### Package provenance

- Where available, verify package provenance (npm provenance, Sigstore for Python).
- Prefer packages from verified publishers over anonymous uploads.
- For critical dependencies, review the source code rather than trusting the published package blindly.

---

## Data Classification

Every piece of data handled by the application belongs to a classification tier. The tier determines the security controls required.

| Tier | Examples | Controls |
|------|----------|----------|
| **Public** | Marketing content, public API documentation, open-source code | No special controls. Integrity protection (checksums, signed releases) where relevant. |
| **Internal** | Internal documentation, non-sensitive configuration, team communications | Access restricted to authenticated team members. Not exposed to public endpoints. |
| **Confidential** | User PII (names, emails, addresses), financial data, client project details, internal business metrics | Encrypted at rest and in transit. Access logged. Access restricted by role. Retention and deletion policies enforced. Subject to GDPR/data protection obligations. |
| **Restricted** | Encryption keys, passwords, authentication tokens, payment card data, health records | Encrypted at rest with dedicated key management. Access restricted to the minimum necessary services and personnel. Access logged and alerted. Zeroized in memory after use. Subject to regulatory requirements (PCI DSS, GDPR Article 9, etc.). |

### Rules

- Every new data field or storage location must be classified before implementation.
- Data must not be stored at a lower classification tier than it requires. If a log file contains user emails, the log file is Confidential, not Internal.
- Data classification determines logging policy: Restricted data must never appear in logs. Confidential data may appear in logs only if it is masked or redacted (e.g., email: `s***@example.com`).
- When data is shared with third parties (analytics, monitoring, error tracking), verify that the third party meets the security controls required for the data's classification tier.
- Data retention and deletion must follow the classification tier's requirements. Confidential and Restricted data must be deletable on request (GDPR right to erasure) and must not persist beyond the defined retention period.

---

## Security Logging and Monitoring

Security logging is distinct from application logging (see `CODING-PRINCIPLES.md` for general logging rules). Security events require specific retention, alerting, and tamper protection.

### Events that must be logged

| Event category | Specific events |
|---------------|-----------------|
| Authentication | Successful login, failed login, logout, MFA success, MFA failure, password reset request, password change |
| Authorisation | Access denied (403), privilege escalation attempt, role change, permission grant/revoke |
| Admin actions | User creation/deletion, role assignment, configuration change, secret rotation |
| Data access | Access to Restricted-tier data, bulk data export, API key creation/revocation |
| Security incidents | Rate limit triggered, account lockout, invalid CSRF token, malformed request rejected, WAF block |

### What must never be logged

- Passwords (plaintext or hashed).
- Authentication tokens, session IDs, API keys, or any other credential.
- Full credit card numbers, CVVs, or bank account numbers.
- Unmasked PII beyond what is necessary for identification in the log (e.g., log user ID, not full name + email + address).

### Log format and integrity

- Use structured logging (JSON) with consistent fields: `timestamp`, `event_type`, `actor` (user ID or service name), `resource`, `action`, `outcome` (success/failure), `ip_address`, `request_id`.
- Include a correlation ID / request ID in every log entry so that events from the same request can be traced across services.
- Store security logs separately from application logs where possible. Security logs should have a longer retention period and stricter access controls.
- Security logs must be append-only. Application code must not be able to delete or modify existing log entries. In production, forward logs to a centralised system (ELK, Loki, CloudWatch, or equivalent) that enforces immutability.

### Alerting

- Alert immediately on: multiple failed login attempts from the same IP (threshold: 10 in 5 minutes), successful login from a new country/region, privilege escalation, access to Restricted-tier data outside normal patterns, and any rate limit breach on authentication endpoints.
- Review security logs weekly for anomalies even in the absence of alerts.

---

## Incident Response (Developer-Facing)

When you discover a security vulnerability or suspect a breach, follow this process. Speed matters — the faster the response, the smaller the impact.

### 1. Contain

- If a secret is compromised, rotate it immediately. Do not wait for approval — rotate first, document later.
- If a vulnerability is actively exploited, disable the affected endpoint or feature. A temporarily degraded service is better than an actively compromised one.
- If user data may have been exposed, preserve all relevant logs before they rotate out.

### 2. Notify

- Notify the project lead and security contact within 1 hour of discovery.
- Do not discuss the vulnerability in public channels (Slack, GitHub issues, public commits) until it is resolved.
- If the vulnerability affects user data, the project lead will determine whether regulatory notification is required (e.g., ICO notification under UK GDPR within 72 hours).

### 3. Investigate

- Determine the scope: what data was exposed, how many users were affected, how the vulnerability was introduced, and how long it has been present.
- Review access logs, deployment history, and git history for the affected components.
- Document findings as you go — the incident report will be assembled from these notes.

### 4. Fix

- Develop and test the fix in a private branch if the vulnerability is not yet public.
- The fix must include a test that verifies the vulnerability is closed (see Security Testing).
- Deploy the fix to production as a priority. Follow the normal deployment process but escalate review times.

### 5. Document

- Write an incident report covering: timeline, root cause, impact, fix applied, and follow-up actions to prevent recurrence.
- Store the incident report in the project's internal documentation (not in the public repository).
- Update security controls, checklists, and this document if the incident reveals a gap.

### Rules

- Never attempt to "quietly fix" a security issue without notifying the team. Silent fixes hide the scope of the problem and prevent systemic improvements.
- Never blame an individual. Incident reports focus on process, systems, and controls — not people.
- Every incident results in at least one follow-up action (improved monitoring, additional testing, updated controls). An incident that produces no follow-up action was not investigated thoroughly.

---

## Security Testing

The following security scenarios must be covered by automated tests. See `TESTING.md` for how to write these tests.

| Scenario | Test Type |
|----------|-----------|
| Unauthenticated access to protected endpoint returns 401 | Integration |
| Unauthorised user cannot access another user's resource (IDOR) | Integration |
| Privilege escalation: lower-role user cannot perform higher-role action | Integration |
| SQL injection characters in input are rejected or escaped | Unit / Integration |
| XSS: script tags in user-provided text are escaped on output | Integration / E2E |
| Mass assignment: protected fields cannot be set via API | Integration |
| CSRF token missing returns 419/403 | Integration |
| Rate limiting blocks excessive requests | Integration |
| GraphQL query exceeding max depth is rejected | Integration |
| GraphQL batched alias attack is rate-limited | Integration |
| File upload with disallowed MIME type is rejected | Integration |
| File upload exceeding size limit is rejected | Integration |
| Tampered ciphertext fails authentication (GCM tag failure) | Unit |
| Expired or revoked token is rejected | Integration |
| Password below minimum length is rejected | Unit |
| Common password from blocklist is rejected | Unit |
| Error responses do not leak stack traces, file paths, or internal details | Integration |
| Application fails closed on unexpected exceptions (denies access, does not fail open) | Integration |
| Misconfigured security headers are detected (missing CSP, HSTS, X-Content-Type-Options) | Integration / E2E |
| SSRF: requests to private/internal IP ranges are blocked | Integration |

---

## Security Checklist

Before deploying or merging any change to staging or production:

- [ ] No secrets, API keys, or credentials in the diff
- [ ] All new endpoints have authentication and authorisation
- [ ] All user-controlled inputs are validated before use
- [ ] All database queries use parameterised statements or the ORM
- [ ] No `dangerouslySetInnerHTML` without sanitisation (React)
- [ ] No `$guarded = []` on Eloquent models (Laravel)
- [ ] Debug mode is disabled in staging and production
- [ ] HTTP security headers are set
- [ ] Dependencies have been audited (`composer audit` / `pip-audit` / `npm audit`)
- [ ] Uploaded files are validated, size-limited, and stored outside the webroot
- [ ] Sensitive data is encrypted at rest where required
- [ ] Rate limiting is applied to authentication endpoints
- [ ] Logging does not include passwords, tokens, or PII
- [ ] New cryptographic usage uses only approved algorithms
- [ ] Password hashing uses argon2id (or scrypt/bcrypt per the approved hierarchy)
- [ ] Error handling fails closed — no sensitive data leaked in error responses (A10:2025)
- [ ] GraphQL endpoints have depth and complexity limits enabled
- [ ] No secrets baked into container images or passed as build arguments
- [ ] New data fields have been classified and controls applied accordingly
- [ ] Security-relevant events are logged with structured format
