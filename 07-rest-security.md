# REST APIs, Validation, and Spring Security

## REST API design

### 1. What is REST?

**Interview-ready answer:**

REST is an architectural style for resource-oriented systems with a uniform interface, stateless requests, cacheable responses where appropriate, and layered components. An HTTP JSON API is not automatically RESTful; resource modeling and HTTP semantics matter.

### 2. Explain HTTP method semantics.

**Interview-ready answer:**

`GET` reads, `POST` normally creates or triggers a non-idempotent operation, `PUT` replaces a resource and is idempotent, `PATCH` partially updates, and `DELETE` removes. Safe methods do not intend state changes; idempotent methods can be repeated with the same intended effect.

### 3. What is idempotency?

**Interview-ready answer:**

An idempotent operation has the same intended server state after repeated identical requests as after one. For non-idempotent workflows such as payment creation, I accept an idempotency key, store the first outcome atomically, and return it for retries.

### 4. Common HTTP status codes

| Code | Use |
|---|---|
| 200 | successful read or operation with body |
| 201 | resource created; include `Location` when useful |
| 202 | accepted for asynchronous processing |
| 204 | success with no response body |
| 400 | malformed request or basic validation failure |
| 401 | missing or invalid authentication |
| 403 | authenticated but not authorized |
| 404 | resource not found |
| 409 | state conflict or duplicate |
| 422 | semantically invalid request when adopted by the API |
| 429 | rate limit exceeded |
| 500 | unexpected server failure |
| 503 | temporarily unavailable |

### 5. PUT versus PATCH

**Interview-ready answer:**

PUT represents full replacement at a resource URI and should be idempotent. PATCH applies a partial change and may use JSON Merge Patch or JSON Patch semantics. I document null, missing-field, and concurrency behavior clearly.

### 6. API versioning strategies

**Interview-ready answer:**

Common choices are URI, header, or media-type versioning. URI versioning is visible and simple; header-based versioning keeps URIs stable but is less obvious. I prefer backward-compatible additive evolution and version only for breaking contract changes.

### 7. What should an error response contain?

**Interview-ready answer:**

A stable machine-readable code, safe human message, HTTP status, timestamp, request path, correlation ID, and structured field errors when relevant. It should not expose stack traces, SQL, credentials, or internal class names.

### 8. How do you prevent duplicate updates?

**Interview-ready answer:**

For concurrent updates I use versioning through an ETag/`If-Match` contract or an entity version and return a conflict when stale. For repeated creates I use an idempotency key backed by a unique constraint and stored result.

### 9. How do you design bulk APIs?

**Interview-ready answer:**

I set batch-size limits, define whether processing is atomic or partial, return per-item outcomes for partial processing, make retry behavior clear, and consider asynchronous processing for expensive work. I protect memory and downstream services with bounded concurrency.

### 10. How do you document APIs?

**Interview-ready answer:**

I maintain an OpenAPI contract including schemas, status codes, examples, authentication, pagination, and error formats. I verify the contract in tests and treat breaking changes as an explicit review decision.

## Security

### 11. Authentication versus authorization

**Interview-ready answer:**

Authentication verifies who the caller is. Authorization decides what that authenticated identity may do. Both are required; a valid token does not automatically grant access to every resource.

### 12. How does Spring Security work?

**Interview-ready answer:**

Spring Security applies a filter chain before the controller. Authentication filters extract credentials, an authentication manager/provider validates them, and the resulting `Authentication` is stored in the security context. Authorization rules then evaluate access at request or method level.

### 13. What is JWT?

**Interview-ready answer:**

A JWT is a signed token containing claims. The server verifies signature, issuer, audience, expiry, and allowed algorithm before trusting claims. A signed JWT provides integrity, not confidentiality, so sensitive data should not be placed in its payload.

### 14. JWT structure

**Interview-ready answer:**

A JWT has base64url-encoded header, payload, and signature sections. The header identifies type and algorithm, the payload contains claims, and the signature detects tampering. Encoding is not encryption.

### 15. Access token versus refresh token

**Interview-ready answer:**

An access token is short-lived and sent to APIs. A refresh token is longer-lived and exchanged at the authorization server for new access tokens. Refresh tokens need stronger storage, rotation, and revocation controls and should not be sent to resource APIs.

### 16. OAuth 2.0 versus OpenID Connect

**Interview-ready answer:**

OAuth 2.0 is an authorization framework for delegated access. OpenID Connect adds an identity layer and ID tokens for authentication. For user-facing applications, Authorization Code with PKCE is a standard secure flow; machine-to-machine services commonly use Client Credentials.

### 17. Session authentication versus token authentication

**Interview-ready answer:**

Session authentication stores login state server-side and sends a session identifier, making revocation straightforward but requiring shared or routed state at scale. Self-contained access tokens reduce server-side lookup but require careful expiry and revocation design. The choice depends on clients and architecture.

### 18. What is CSRF?

**Interview-ready answer:**

CSRF tricks a browser into sending authenticated requests using automatically attached credentials such as cookies. CSRF protection is important for cookie-based sessions. A stateless API using bearer tokens from an authorization header is generally not vulnerable in the same way, but XSS and token storage remain concerns.

### 19. What is CORS?

**Interview-ready answer:**

CORS is a browser policy controlling whether scripts from one origin can access another origin. The server returns allowed origins, methods, and headers, and some requests use a preflight. CORS is not authentication and does not protect non-browser clients.

### 20. How do you store passwords?

**Interview-ready answer:**

I never store plaintext or reversible encrypted passwords. I use an adaptive password hash such as bcrypt, scrypt, or Argon2 with a unique salt and suitable work factor, then upgrade parameters over time.

### 21. Role-based versus permission-based access

**Interview-ready answer:**

Roles group broad responsibilities, while permissions represent specific actions. I use least privilege and often map roles to granular authorities. I also enforce ownership or tenant checks at the service/data boundary because endpoint roles alone may not prevent access to another user’s resource.

### 22. Method security

**Interview-ready answer:**

Method security applies authorization at service methods, for example with `@PreAuthorize`. It provides defense in depth when a service can be called from more than one entry point. Rules should remain understandable and business ownership checks may need explicit code.

### 23. How do you secure service-to-service communication?

**Interview-ready answer:**

I authenticate workload identity using short-lived OAuth client credentials, mTLS, or the platform’s workload identity. I authorize scopes or service permissions, encrypt traffic, rotate credentials, and never treat an internal network as automatically trusted.

### 24. Major API security controls

**Interview-ready answer:**

Strong authentication, object-level authorization, input validation, parameterized queries, output encoding, rate limiting, payload limits, secure headers, secret management, dependency scanning, audit logs, and protection against mass assignment. I avoid logging tokens or personal data.

## Scenario: secure an order endpoint

**Interview-ready answer:**

I validate the access token at the resource server, require an appropriate order scope, and check that the requested order belongs to the authenticated customer unless an admin permission applies. I validate input DTOs, use parameterized persistence, return safe errors, audit sensitive changes, and rate-limit risky operations. Transport uses TLS and logs exclude tokens and payment data.

