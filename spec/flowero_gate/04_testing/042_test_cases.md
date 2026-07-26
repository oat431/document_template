---
document_type: Test Cases
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Gate"
project_id: "flowero-gate"
classification: "Internal"
tags: [test-cases, gateway, spring-cloud-gateway, jwt, rate-limiting]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Test Cases — Flowero Gate

> **Project:** Flowero Gate (Spring Cloud Gateway)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Detailed test cases for Flowero Gate, the API Gateway service. Covers JWT validation, rate limiting, routing, CORS, circuit breaking, and claim forwarding.

## 2. Test Case Index

| Module | Total | Automated | Manual | Status |
|--------|:-----:|:---------:|:------:|:------:|
| JWT Authentication | 12 | 10 | 2 | ⬜ |
| Rate Limiting | 8 | 6 | 2 | ⬜ |
| Routing & Load Balancing | 10 | 8 | 2 | ⬜ |
| CORS Configuration | 6 | 4 | 2 | ⬜ |
| Circuit Breaker | 5 | 4 | 1 | ⬜ |
| Claim Forwarding | 4 | 4 | 0 | ⬜ |
| **Total** | **45** | **36** | **9** | |

---

## 3. JWT Authentication

### GATE-TC-001: Valid JWT Accesses Protected Endpoint

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-001 |
| **Title** | Valid JWT token accesses protected API endpoint |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

**Preconditions:**
| # | Condition |
|---|-----------|
| 1 | Gate is running and healthy |
| 2 | Keycloak is running and issuing valid JWTs |
| 3 | Test user exists in Keycloak |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain valid JWT from Keycloak | JWT received |
| 2 | Call `GET /api/v1/users/me` with `Authorization: Bearer <jwt>` | 200 OK (or 404 if route not configured, but NOT 401/403) |

### GATE-TC-002: Missing JWT Returns 401

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-002 |
| **Title** | Request without JWT returns 401 Unauthorized |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /api/v1/users/me` without Authorization header | 401 Unauthorized |
| 2 | Verify response body | JSON with `error: "Unauthorized"`, `status: 401` |

### GATE-TC-003: Invalid JWT Returns 401

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-003 |
| **Title** | Request with invalid JWT returns 401 |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /api/v1/users/me` with `Authorization: Bearer invalid-token` | 401 Unauthorized |

### GATE-TC-004: Expired JWT Returns 401

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-004 |
| **Title** | Request with expired JWT returns 401 |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain JWT, wait for expiration (5 minutes) | Token expired |
| 2 | Call protected endpoint with expired JWT | 401 Unauthorized |

### GATE-TC-005: Public Endpoint Accessible Without JWT

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-005 |
| **Title** | Public endpoints accessible without authentication |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /actuator/health` without JWT | 200 OK |
| 2 | Call `GET /fallback/not-found` without JWT | 404 Not Found (not 401) |
| 3 | Call `GET /api/v1/public/**` without JWT | 200 OK (if route configured) |

### GATE-TC-006: JWT from Wrong Issuer Rejected

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-006 |
| **Title** | JWT from different issuer is rejected |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain JWT from different Keycloak realm | JWT with wrong issuer |
| 2 | Call protected endpoint | 401 Unauthorized (issuer mismatch) |

### GATE-TC-007: Insufficient Role Returns 403

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-007 |
| **Title** | Valid JWT with insufficient role returns 403 |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as `viewer` role user | JWT with `viewer` role |
| 2 | Call admin-only endpoint | 403 Forbidden |
| 3 | Verify response body | JSON with `error: "Forbidden"`, `status: 403` |

### GATE-TC-008: Browser Login Redirect

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-008 |
| **Title** | Browser navigation redirects to Keycloak login |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to protected endpoint with `Accept: text/html` | 302 redirect to `/oauth2/authorization/keycloak` |
| 2 | Follow redirect | Keycloak login page displayed |

### GATE-TC-009: API Request Returns JSON 401

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-009 |
| **Title** | API request (XHR) returns JSON 401, not redirect |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call protected endpoint with `Accept: application/json` | 401 Unauthorized (JSON response, not 302 redirect) |

### GATE-TC-010: JWKS Cache Refresh

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-010 |
| **Title** | JWKS cache refreshes after Keycloak key rotation |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | ADR-007 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Rotate Keycloak signing keys | New keys active |
| 2 | Wait for JWKS cache TTL (5 minutes) | Cache refreshed |
| 3 | Call protected endpoint with new JWT | 200 OK |

### GATE-TC-011: Multiple JWT Algorithms

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-011 |
| **Title** | Gate accepts only RS256 algorithm |
| **Priority** | 🟡 High |
| **Type** | Security |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain JWT signed with RS256 | Valid JWT |
| 2 | Call protected endpoint | 200 OK |
| 3 | Obtain JWT signed with HS256 (symmetric) | JWT with wrong algorithm |
| 4 | Call protected endpoint | 401 Unauthorized |

### GATE-TC-012: JWT with Missing Required Claims

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-012 |
| **Title** | JWT missing required claims is rejected |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Craft JWT missing `sub` claim | Invalid JWT |
| 2 | Call protected endpoint | 401 Unauthorized |

---

## 4. Rate Limiting

### GATE-TC-101: Rate Limit Enforced Per IP

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-101 |
| **Title** | Rate limit enforced per IP address |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send 100 requests from same IP within 1 minute | All succeed |
| 2 | Send 101st request | 429 Too Many Requests |
| 3 | Wait for rate window to reset | Requests succeed again |

### GATE-TC-102: Rate Limit Enforced Per User

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-102 |
| **Title** | Rate limit enforced per authenticated user |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as user A, send 100 requests | All succeed |
| 2 | Login as user B, send 100 requests | All succeed (different bucket) |
| 3 | User A sends 101st request | 429 (user A's bucket exhausted) |

### GATE-TC-103: Rate Limit Persists Across Gate Restart

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-103 |
| **Title** | Rate limit counters persist across Gate restart |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | ADR-008 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send 90 requests (near limit) | All succeed |
| 2 | Restart Gate container | Gate reboots |
| 3 | Send 15 more requests immediately | 429 returned (counter persisted in Valkey) |

### GATE-TC-104: Rate Limit Headers Present

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-104 |
| **Title** | Rate limit headers present in response |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call protected endpoint | Response includes `X-RateLimit-Remaining`, `X-RateLimit-Limit` headers |

### GATE-TC-105: 429 Response Format

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-105 |
| **Title** | 429 response has correct format |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Exceed rate limit | 429 Too Many Requests |
| 2 | Verify response body | JSON with `error: "Too Many Requests"`, `status: 429` |

### GATE-TC-106: Valkey Unavailable — Fail Open

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-106 |
| **Title** | Gate handles Valkey unavailability gracefully |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop Valkey container | Valkey down |
| 2 | Call protected endpoint | 200 OK (rate limiting disabled, fail-open) |
| 3 | Restart Valkey | Rate limiting resumes |

### GATE-TC-107: Per-Route Rate Limit

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-107 |
| **Title** | Different routes have different rate limits |
| **Priority** | 🟢 Medium |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Configure `/api/v1/public/**` with higher limit (1000/min) | Config applied |
| 2 | Send 500 requests to public route | All succeed |
| 3 | Send 500 requests to protected route | 429 returned (lower limit) |

### GATE-TC-108: Anonymous User Rate Limit

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-108 |
| **Title** | Anonymous users rate limited by IP |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call public endpoint without JWT | Rate limited by IP |
| 2 | Exceed limit | 429 returned |

---

## 5. Routing & Load Balancing

### GATE-TC-201: Route to Business Service

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-201 |
| **Title** | Gate routes to registered business service |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Register business service in Eureka | Service appears in registry |
| 2 | Call `GET /api/v1/todo/tasks` with valid JWT | Gate resolves `lb://tiny-mchwa`, proxies request, returns 200 |

### GATE-TC-202: Route Not Found Returns 404

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-202 |
| **Title** | Undefined route returns 404 |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /api/v1/undefined/path` | 404 Not Found |
| 2 | Verify fallback controller handles response | JSON error response |

### GATE-TC-203: Load Balancer Distributes Traffic

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-203 |
| **Title** | Load balancer distributes traffic across service instances |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-204 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Register 2 instances of same service in Eureka | Both instances UP |
| 2 | Send 100 requests | Requests distributed across both instances |

### GATE-TC-204: Service Unavailable Returns 503

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-204 |
| **Title** | Unregistered service returns 503 |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-204 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop business service | Service down |
| 2 | Wait for Eureka eviction (90 seconds) | Service removed from registry |
| 3 | Call route for that service | 503 Service Unavailable |

### GATE-TC-205: Strip Prefix Filter

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-205 |
| **Title** | StripPrefix filter removes route prefix |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /api/v1/todo/tasks` | Gate strips `/api/v1`, forwards `/tasks` to service |
| 2 | Verify service receives correct path | Service logs show `/tasks` |

### GATE-TC-206: Route Timeout

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-206 |
| **Title** | Route timeout enforced |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Configure route with 5 second timeout | Config applied |
| 2 | Call endpoint that takes 10 seconds | 504 Gateway Timeout after 5 seconds |

### GATE-TC-207: Route Metadata Forwarded

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-207 |
| **Title** | Route metadata forwarded to downstream service |
| **Priority** | 🟢 Medium |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Configure route with custom metadata | Metadata in route config |
| 2 | Call route | Service receives metadata as headers |

### GATE-TC-208: WebSocket Route

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-208 |
| **Title** | WebSocket connections routed correctly |
| **Priority** | 🟢 Medium |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Establish WebSocket connection through Gate | Connection upgraded |
| 2 | Send messages | Messages proxied to service |

### GATE-TC-209: Route Priority

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-209 |
| **Title** | More specific routes take priority |
| **Priority** | 🟢 Medium |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Configure `/api/v1/admin/**` and `/api/v1/**` | Both routes defined |
| 2 | Call `/api/v1/admin/users` | Routes to admin service (more specific) |

### GATE-TC-210: Dynamic Route Addition

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-210 |
| **Title** | New service automatically routable after Eureka registration |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-204 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Start new business service | Service registers with Eureka |
| 2 | Wait 30 seconds | Eureka propagates registration |
| 3 | Call route for new service | 200 OK (route resolved) |

---

## 6. CORS Configuration

### GATE-TC-301: CORS Preflight Allowed

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-301 |
| **Title** | CORS preflight (OPTIONS) returns correct headers |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `OPTIONS /api/v1/users/me` with CORS headers | 200 OK with `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods` |

### GATE-TC-302: CORS Origin Validation

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-302 |
| **Title** | CORS rejects requests from untrusted origins |
| **Priority** | 🟡 High |
| **Type** | Security |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send request from `https://evil.com` | CORS headers not present (or rejected) |

### GATE-TC-303: CORS Credentials Allowed

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-303 |
| **Title** | CORS allows credentials for trusted origins |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send request with `Origin: https://blog.panomete.com` | `Access-Control-Allow-Credentials: true` |

### GATE-TC-304: CORS Max-Age

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-304 |
| **Title** | CORS preflight cached for max-age duration |
| **Priority** | 🟢 Medium |
| **Type** | Functional |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send OPTIONS request | `Access-Control-Max-Age: 3600` present |

### GATE-TC-305: CORS Allowed Methods

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-305 |
| **Title** | CORS allows only configured methods |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send OPTIONS with `Access-Control-Request-Method: PATCH` | Method not allowed (if not configured) |

### GATE-TC-306: CORS Allowed Headers

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-306 |
| **Title** | CORS allows only configured headers |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send request with custom header `X-Custom-Header` | Header not allowed (if not configured) |

---

## 7. Circuit Breaker

### GATE-TC-401: Circuit Breaker Opens on Failure

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-401 |
| **Title** | Circuit breaker opens after consecutive failures |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop downstream service | Service down |
| 2 | Send 5 requests (failure threshold) | 502/503 returned |
| 3 | Send 6th request | Circuit breaker opens, immediate 503 |

### GATE-TC-402: Circuit Breaker Half-Open

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-402 |
| **Title** | Circuit breaker transitions to half-open |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Circuit breaker open | All requests fail fast |
| 2 | Wait for wait-duration (30 seconds) | Circuit transitions to half-open |
| 3 | Send test request | Request forwarded to service |

### GATE-TC-403: Circuit Breaker Closes on Success

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-403 |
| **Title** | Circuit breaker closes after successful request |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Circuit breaker half-open | Testing requests |
| 2 | Send successful request | Circuit closes |
| 3 | Subsequent requests succeed | Normal operation |

### GATE-TC-404: Fallback Response

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-404 |
| **Title** | Fallback response returned when circuit open |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Circuit breaker open | All requests fail fast |
| 2 | Check response | Fallback response from `FallbackController` |

### GATE-TC-405: Circuit Breaker Metrics

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-405 |
| **Title** | Circuit breaker metrics exposed via Actuator |
| **Priority** | 🟢 Medium |
| **Type** | Functional |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /actuator/metrics/resilience4j.circuitbreaker.state` | Metrics returned |
| 2 | Verify state transitions logged | State changes visible in metrics |

---

## 8. Claim Forwarding

### GATE-TC-501: User ID Header Forwarded

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-501 |
| **Title** | `X-User-Id` header forwarded to downstream service |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as user with `sub: user123` | JWT issued |
| 2 | Call protected endpoint | Downstream service receives `X-User-Id: user123` |

### GATE-TC-502: User Email Header Forwarded

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-502 |
| **Title** | `X-User-Email` header forwarded |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as user with `email: test@panomete.com` | JWT issued |
| 2 | Call protected endpoint | Downstream service receives `X-User-Email: test@panomete.com` |

### GATE-TC-503: User Roles Header Forwarded

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-503 |
| **Title** | `X-User-Roles` header forwarded |
| **Priority** | 🔴 Critical |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as user with roles `[admin, editor]` | JWT issued |
| 2 | Call protected endpoint | Downstream service receives `X-User-Roles: admin,editor` |

### GATE-TC-504: User Scope Header Forwarded

| Field | Value |
|-------|-------|
| **ID** | GATE-TC-504 |
| **Title** | `X-User-Scope` header forwarded |
| **Priority** | 🟡 High |
| **Type** | Functional |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login with scope `openid profile email` | JWT issued |
| 2 | Call protected endpoint | Downstream service receives `X-User-Scope: openid profile email` |

---

## 9. Test Execution Summary

| Module | Executed | Passed | Failed | Blocked | Pass Rate |
|--------|:--------:|:------:|:------:|:-------:|:---------:|
| JWT Authentication | 12 | — | — | — | — |
| Rate Limiting | 8 | — | — | — | — |
| Routing & Load Balancing | 10 | — | — | — | — |
| CORS Configuration | 6 | — | — | — | — |
| Circuit Breaker | 5 | — | — | — | — |
| Claim Forwarding | 4 | — | — | — | — |
| **Total** | **45** | **—** | **—** | **—** | **—** |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Test plan governing these cases |
| [[043_defect_report]] | Defects found during testing |
| [[../01_requirement/012_user_stories]] | User stories (oracle) |
| [[../01_requirement/013_acceptance_criteria]] | Acceptance criteria (oracle) |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119
> **Usage:** Every user story needs test cases. Every acceptance criteria needs verification.
