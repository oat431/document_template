---
document_type: Test Cases
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Panomete Platform"
project_id: "PAN-PLAT-001"
classification: "Internal"
tags: [test-cases, platform, integration, e2e, cross-service]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Test Cases — Panomete Platform (Integration & E2E)

> **Project:** Panomete Platform
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Cross-service and end-to-end test cases for the Panomete Platform. These tests verify interactions between Flowero Guard (Keycloak), Flowero Discover (Eureka), Flowero Gate (Spring Cloud Gateway), and Nginx/Cloudflare edge infrastructure.

## 2. Test Case Index

| Category | Total | Automated | Manual | Status |
|----------|:-----:|:---------:|:------:|--------|
| Authentication Flow (E2E) | 8 | 5 | 3 | ⬜ |
| Service Discovery & Registration | 6 | 4 | 2 | ⬜ |
| Gateway Routing & Integration | 10 | 8 | 2 | ⬜ |
| Security (Platform-Wide) | 8 | 6 | 2 | ⬜ |
| Infrastructure & Deployment | 6 | 4 | 2 | ⬜ |
| Observability (Phase 2) | 4 | 2 | 2 | ⬜ |
| **Total** | **42** | **29** | **13** | |

---

## 3. Authentication Flow (E2E)

### PLAT-TC-001: Full Login → API Call → Logout Flow

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-001 |
| **Title** | Complete SSO login flow through Guard → Gate → Business Service |
| **Priority** | 🔴 Critical |
| **Type** | E2E |
| **Requirement** | OBJ-01, US-003, US-203 |

**Preconditions:**
| # | Condition |
|---|-----------|
| 1 | All 3 foundation services deployed and healthy |
| 2 | Test user `testuser` exists in Keycloak `panomete` realm |
| 3 | At least 1 business service registered in Discover |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to `https://auth.panomete.com` | Keycloak login page loads |
| 2 | Login as `testuser` | JWT token issued, SSO cookie set |
| 3 | Call `GET https://api.panomete.com/api/todo/tasks` with `Authorization: Bearer <token>` | Gate validates JWT locally, forwards to business service, returns 200 |
| 4 | Inspect response headers forwarded to business service | `X-User-Id` and `X-User-Roles` headers present |
| 5 | Call same endpoint without Authorization header | Gate returns 401 Unauthorized |
| 6 | Logout via Keycloak | SSO session destroyed, token invalidated |

### PLAT-TC-002: SSO Across Multiple Services

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-002 |
| **Title** | Single Sign-On works across all platform services |
| **Priority** | 🔴 Critical |
| **Type** | E2E |
| **Requirement** | OBJ-01, US-003 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login at `auth.panomete.com` | JWT issued |
| 2 | Call Gate API for blog service | 200 (authenticated) |
| 3 | Call Gate API for todo service | 200 (same token, no re-auth) |
| 4 | Call Gate API for URL service | 200 (same token, no re-auth) |
| 5 | Verify all 3 calls used the same JWT | Same `sub` claim in forwarded headers |

### PLAT-TC-003: Expired Token Rejection

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-003 |
| **Title** | Gate rejects expired JWT tokens |
| **Priority** | 🔴 Critical |
| **Type** | E2E |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain valid JWT from Keycloak | Token received |
| 2 | Wait 6 minutes (access token lifetime = 5 min) | Token expired |
| 3 | Call Gate API with expired token | Gate returns 401 Unauthorized |
| 4 | Use refresh token to obtain new access token | New token issued |
| 5 | Call Gate API with new token | 200 (authenticated) |

### PLAT-TC-004: Token from Different Realm Rejected

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-004 |
| **Title** | Gate rejects JWT from non-panomete realm |
| **Priority** | 🟡 High |
| **Type** | E2E |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Create a second realm `other-realm` in Keycloak | Realm created |
| 2 | Obtain JWT from `other-realm` | Token issued |
| 3 | Call Gate API with other-realm token | Gate returns 401 (issuer mismatch) |

### PLAT-TC-005: Guard Unavailable — Gate Behavior

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-005 |
| **Title** | Gate continues to work with cached JWKS when Guard is temporarily down |
| **Priority** | 🟡 High |
| **Type** | E2E |
| **Requirement** | ADR-007 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain valid JWT from Keycloak | Token received |
| 2 | Stop Guard container | Guard down |
| 3 | Call Gate API with valid token | 200 (JWKS cached, local validation) |
| 4 | Call Gate API without token | 401 (no dependency on Guard) |
| 5 | Restart Guard | Guard recovers |

### PLAT-TC-006: Client Credentials (Service-to-Service)

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-006 |
| **Title** | Service-to-service authentication via client credentials grant |
| **Priority** | 🟡 High |
| **Type** | E2E |
| **Requirement** | US-005 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `POST /realms/panomete/protocol/openid-connect/token` with `grant_type=client_credentials` | Access token returned |
| 2 | Use token to call Gate API | 200 (service account authenticated) |

### PLAT-TC-007: Refresh Token Flow

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-007 |
| **Title** | Token refresh produces valid new access token |
| **Priority** | 🔴 Critical |
| **Type** | E2E |
| **Requirement** | US-003 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login and obtain access + refresh tokens | Both tokens received |
| 2 | Call `POST /realms/panomete/protocol/openid-connect/token` with `grant_type=refresh_token` | New access token issued |
| 3 | Use new access token at Gate | 200 (authenticated) |
| 4 | Old refresh token is invalidated | Reuse returns error |

### PLAT-TC-008: RBAC Enforcement Through Gate

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-008 |
| **Title** | Role-based access control enforced end-to-end |
| **Priority** | 🟡 High |
| **Type** | E2E |
| **Requirement** | US-004 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as `viewer` role user | JWT with `realm_access.roles = ["viewer"]` |
| 2 | Call admin-only API through Gate | 403 Forbidden (or business service rejects) |
| 3 | Login as `admin` role user | JWT with `realm_access.roles = ["admin"]` |
| 4 | Call same admin API through Gate | 200 (authorized) |

---

## 4. Service Discovery & Registration

### PLAT-TC-101: All Foundation Services Register with Discover

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-101 |
| **Title** | Guard, Discover, and Gate register in Eureka |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | OBJ-02, US-102 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Start all foundation services | Services boot in order |
| 2 | Check Eureka dashboard at `discovery.panomete.com` | Gate and Guard listed as UP |
| 3 | Query Eureka API: `GET /eureka/apps` | Returns registered service instances |
| 4 | Verify registration within 30 seconds of boot | Registration timestamp < 30s after boot |

### PLAT-TC-102: Service Deregistration on Shutdown

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-102 |
| **Title** | Stopped services are removed from Eureka |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | US-102 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Verify Gate is UP in Eureka | Listed as UP |
| 2 | Stop Gate container | Container stopped |
| 3 | Wait 90 seconds | Eureka eviction timeout |
| 4 | Check Eureka dashboard | Gate listed as DOWN or removed |

### PLAT-TC-103: Gate Resolves Routes via Discover

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-103 |
| **Title** | Gate uses `lb://` URIs to resolve business services from Eureka |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | US-204 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Register a business service in Discover | Service appears in Eureka |
| 2 | Call `GET https://api.panomete.com/api/todo/tasks` with valid JWT | Gate resolves `lb://tiny-mchwa` via Eureka, proxies request, returns 200 |
| 3 | Stop the business service, wait for Eureka eviction | Service removed |
| 4 | Call same endpoint | Gate returns 503 Service Unavailable |

### PLAT-TC-104: Discover Dashboard Accessible via Nginx

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-104 |
| **Title** | Eureka dashboard accessible at `discovery.panomete.com` |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | US-104 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to `https://discovery.panomete.com` | Eureka dashboard loads |
| 2 | Check service instances listed | All registered services shown |

### PLAT-TC-105: Eureka Self-Preservation Mode

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-105 |
| **Title** | Eureka enters self-preservation when heartbeat threshold drops |
| **Priority** | 🟢 Medium |
| **Type** | Integration |
| **Requirement** | US-101 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Rapidly stop multiple services | Heartbeats drop |
| 2 | Check Eureka dashboard | Self-preservation mode warning displayed |
| 3 | Services not immediately evicted | Eureka protects registry |

### PLAT-TC-106: Discover Restart Rebuilds Registry

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-106 |
| **Title** | Eureka registry rebuilds after Discover restart |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | US-101, US-102 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Restart Discover container | Registry cleared (in-memory) |
| 2 | Wait 60 seconds | Services re-register |
| 3 | Check Eureka dashboard | All services back to UP status |

---

## 5. Gateway Routing & Integration

### PLAT-TC-201: All Business API Routes Configured

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-201 |
| **Title** | Gate routes all defined business API paths |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | US-202 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET /api/blog/**` with valid JWT | Routes to Cute Gufo |
| 2 | Call `GET /api/short/**` with valid JWT | Routes to Fluffy Mouton |
| 3 | Call `GET /api/todo/**` with valid JWT | Routes to Tiny Mchwa |
| 4 | Call `GET /api/ledger/**` with valid JWT | Routes to Big Schwein |
| 5 | Call `GET /api/recipe/**` with valid JWT | Routes to Shy Ardilla |
| 6 | Call `GET /api/hora/**` with valid JWT | Routes to White Jelen |

### PLAT-TC-202: Non-API Paths Rejected by Gate

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-202 |
| **Title** | Gate returns 404 for undefined paths |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | ADR-009 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call `GET https://api.panomete.com/admin/something` | 404 Not Found |
| 2 | Call `GET https://api.panomete.com/` | 404 Not Found (no root route) |

### PLAT-TC-203: Rate Limiting via Valkey

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-203 |
| **Title** | Valkey-backed rate limiting enforces per-IP limits |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | US-205, ADR-008 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send requests below rate limit (e.g., 50/min) | All return 200 |
| 2 | Exceed rate limit (100+ req/min from same IP) | 429 Too Many Requests returned |
| 3 | Wait for rate window to reset | Requests succeed again |
| 4 | Restart Gate container | Rate limits persist (Valkey-backed) |
| 5 | Verify Valkey contains rate limit keys | Keys visible in `valkey-cli` |

### PLAT-TC-204: Gate Adds P95 Latency < 50ms

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-204 |
| **Title** | Gate adds minimal latency to proxied requests |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | OBJ-03 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Measure direct response time to business service | Baseline: X ms |
| 2 | Measure response time through Gate | Gate path: Y ms |
| 3 | Calculate added latency | Y - X < 50ms at p95 |

### PLAT-TC-205: CORS Headers on Gate

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-205 |
| **Title** | Gate returns correct CORS headers for allowed origins |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | US-201 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `OPTIONS` preflight from `blog.panomete.com` | CORS headers returned with allowed origin |
| 2 | Send `OPTIONS` preflight from unknown origin | CORS rejected or wildcard as configured |

### PLAT-TC-206: Nginx Subdomain Routing to Foundation Services

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-206 |
| **Title** | Nginx correctly routes subdomains to foundation services |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | ADR-009 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET https://auth.panomete.com/health/ready` | Routes to Guard :8001 |
| 2 | `GET https://discovery.panomete.com/` | Routes to Discover :3999 |
| 3 | `GET https://api.panomete.com/actuator/health` | Routes to Gate :8000 |

### PLAT-TC-207: Cloudflare TLS Termination

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-207 |
| **Title** | External traffic is TLS-terminated by Cloudflare |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | ADR-006 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `curl -v https://auth.panomete.com` | TLS 1.2/1.3 handshake visible |
| 2 | Verify internal traffic is HTTP | `curl http://localhost:8001/health/ready` works |
| 3 | Direct access to internal ports blocked externally | Cannot reach :8000, :8001, :8999 from outside |

### PLAT-TC-208: Structured JSON Logging at Gate

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-208 |
| **Title** | Gate emits structured JSON request logs |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | US-201, OBJ-05 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Make an API request through Gate | Request processed |
| 2 | Check Gate container logs | JSON log entry with `timestamp`, `method`, `path`, `status`, `latency_ms`, `route_id`, `client_ip`, `user_id` |

### PLAT-TC-209: Forwarded User Claims as Headers

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-209 |
| **Title** | Gate forwards JWT claims as headers to business services |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as user with `admin` role | JWT issued |
| 2 | Call API through Gate | Business service receives `X-User-Id: <sub>` and `X-User-Roles: admin` headers |

### PLAT-TC-210: Valkey Unavailable — Gate Degraded Mode

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-210 |
| **Title** | Gate handles Valkey unavailability gracefully |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Requirement** | US-205 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop Valkey container | Valkey down |
| 2 | Call API through Gate | Requests still proxied (rate limiting disabled or fail-open) |
| 3 | Restart Valkey | Rate limiting resumes |

---

## 6. Security (Platform-Wide)

### PLAT-TC-301: No Direct Access to Internal Ports

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-301 |
| **Title** | Internal service ports not accessible from external network |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Requirement** | Security Architecture §5.3 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Attempt to reach `remote.panomete.com:8000` externally | Connection refused / blocked by UFW |
| 2 | Attempt to reach `remote.panomete.com:8001` externally | Connection refused / blocked by UFW |
| 3 | Attempt to reach `remote.panomete.com:8999` externally | Connection refused / blocked by UFW |
| 4 | Access through Cloudflare/Nginx works | Services accessible via proper subdomains |

### PLAT-TC-302: Fail2ban Protects SSH

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-302 |
| **Title** | Fail2ban blocks brute-force SSH attempts |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Requirement** | Infrastructure |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Check Fail2ban status | `sudo fail2ban-client status sshd` shows active jail |
| 2 | Verify ban threshold configured | Max retries set (e.g., 3-5 attempts) |

### PLAT-TC-303: Secrets Not in Git

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-303 |
| **Title** | No secrets committed to version control |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Requirement** | Coding Standards §9 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `grep -r "password" --include="*.yml" --include="*.env" .` in repo | No plaintext passwords found |
| 2 | `.gitignore` includes `.env` | `.env` is ignored |
| 3 | GitHub repo secrets configured | `TS_AUTH_KEY`, `HOMELAB_SSH_KEY` in secrets |

### PLAT-TC-304: JWT Claims Contain Required Fields

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-304 |
| **Title** | JWT tokens from Guard contain required claims |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Requirement** | US-203 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain JWT from Keycloak | Token received |
| 2 | Decode JWT (jwt.io) | Contains: `sub`, `iss`, `aud`, `exp`, `iat`, `realm_access.roles` |
| 3 | Verify issuer | `iss: https://auth.panomete.com/realms/panomete` |
| 4 | Verify signature algorithm | RS256 (RSA) |

### PLAT-TC-305: Rate Limit Persistence Across Gate Restart

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-305 |
| **Title** | Rate limit counters persist across Gate container restart |
| **Priority** | 🟡 High |
| **Type** | Security |
| **Requirement** | ADR-008 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send 90 requests (near limit) | All succeed |
| 2 | Restart Gate container | Gate reboots |
| 3 | Send 15 more requests immediately | 429 returned (counter persisted in Valkey) |

### PLAT-TC-306: Docker Network Isolation

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-306 |
| **Title** | Services communicate only on the shared Docker network |
| **Priority** | 🟡 High |
| **Type** | Security |
| **Requirement** | Security Architecture §5.3 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `docker network inspect db-network` | Only expected containers listed |
| 2 | Containers bound to `127.0.0.1` only | No `0.0.0.0` bindings |

### PLAT-TC-307: UFW Firewall Rules

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-307 |
| **Title** | UFW firewall allows only required ports |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Requirement** | Infrastructure |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `sudo ufw status` | Only ports 22 (SSH), 80 (HTTP), 443 (HTTPS) allowed |
| 2 | Internal ports (8000, 8001, 8999) not exposed | Not in UFW rules |

### PLAT-TC-308: Keycloak Admin Console Protected

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-308 |
| **Title** | Keycloak admin console accessible only with admin credentials |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Requirement** | Security Architecture |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to `https://auth.panomete.com/admin` | Login page displayed |
| 2 | Attempt login with wrong credentials | Access denied |
| 3 | Login with admin credentials | Admin console accessible |

---

## 7. Infrastructure & Deployment

### PLAT-TC-401: Docker Compose Startup Order

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-401 |
| **Title** | Services start in correct dependency order |
| **Priority** | 🔴 Critical |
| **Type** | Infrastructure |
| **Requirement** | Deployment Plan §5 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `docker compose -f docker-compose.platform.yml up -d` | All services start |
| 2 | Guard starts before Gate | Verified by container start timestamps |
| 3 | All 3 services healthy within 60 seconds | Health endpoints return 200 |

### PLAT-TC-402: Full Platform Restart Recovery

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-402 |
| **Title** | Platform recovers after full restart |
| **Priority** | 🔴 Critical |
| **Type** | Infrastructure |
| **Requirement** | Runbook §5.2 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop all platform services | All stopped |
| 2 | Start all platform services | Start in dependency order |
| 3 | Wait 60 seconds | All health checks pass |
| 4 | Services re-register with Eureka | Dashboard shows all UP |

### PLAT-TC-403: PostgreSQL Backup and Restore

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-403 |
| **Title** | PostgreSQL backup can be restored |
| **Priority** | 🔴 Critical |
| **Type** | Infrastructure |
| **Requirement** | Runbook §4.3 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `pg_dumpall -U postgres > backup.sql` | Backup created |
| 2 | Verify backup file size > 0 | Non-empty file |
| 3 | Verify backup contains keycloak schema | `grep keycloak backup.sql` finds entries |

### PLAT-TC-404: Rollback Procedure

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-404 |
| **Title** | Services can be rolled back to previous version |
| **Priority** | 🟡 High |
| **Type** | Infrastructure |
| **Requirement** | Deployment Plan §7 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Note current image SHA | SHA recorded |
| 2 | Deploy new version | New image running |
| 3 | Rollback to previous SHA | Previous image restored |
| 4 | Health checks pass | All services healthy |

### PLAT-TC-405: Existing Services Unaffected

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-405 |
| **Title** | Platform deployment doesn't disrupt existing services |
| **Priority** | 🔴 Critical |
| **Type** | Infrastructure |
| **Requirement** | ADR-006 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Verify AdGuard is accessible before deploy | `https://adguard.panomete.com` returns 200 |
| 2 | Deploy platform services | Platform starts |
| 3 | Verify AdGuard still accessible | `https://adguard.panomete.com` returns 200 |
| 4 | Verify Portainer still accessible | `https://container.panomete.com` returns 200 |

### PLAT-TC-406: Resource Limits Enforced

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-406 |
| **Title** | Docker memory limits are enforced |
| **Priority** | 🟡 High |
| **Type** | Infrastructure |
| **Requirement** | SAD §6.3 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `docker stats --no-stream flowero-guard` | Memory limit ≤ 1G |
| 2 | `docker stats --no-stream flowero-discover` | Memory limit ≤ 384M |
| 3 | `docker stats --no-stream flowero-gate` | Memory limit ≤ 512M |

---

## 8. Observability (Phase 2)

### PLAT-TC-501: Health Endpoints Respond

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-501 |
| **Title** | All services expose functioning health endpoints |
| **Priority** | 🔴 Critical |
| **Type** | Observability |
| **Requirement** | OBJ-05 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `curl http://localhost:8001/health/ready` | 200 within 1s |
| 2 | `curl http://localhost:8999/actuator/health` | `{"status":"UP"}` within 1s |
| 3 | `curl http://localhost:8000/actuator/health` | `{"status":"UP"}` within 1s |

### PLAT-TC-502: Structured JSON Logs

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-502 |
| **Title** | All services emit valid JSON logs |
| **Priority** | 🟡 High |
| **Type** | Observability |
| **Requirement** | OBJ-05 |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `docker logs flowero-gate --tail 5` | Each line is valid JSON |
| 2 | JSON contains `timestamp`, `level`, `service` fields | Fields present |

### PLAT-TC-503: Prometheus Scrapes Actuator

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-503 |
| **Title** | Prometheus successfully scrapes `/actuator/prometheus` |
| **Priority** | 🟡 High |
| **Type** | Observability |
| **Requirement** | OBJ-05 (Phase 2) |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Check Prometheus targets | All services UP |
| 2 | Query `jvm_memory_used_bytes` | Metrics returned |

### PLAT-TC-504: Grafana Dashboard Displays Metrics

| Field | Value |
|-------|-------|
| **ID** | PLAT-TC-504 |
| **Title** | Grafana shows platform metrics |
| **Priority** | 🟢 Medium |
| **Type** | Observability |
| **Requirement** | OBJ-05 (Phase 2) |

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Open `https://grafana.panomete.com` | Dashboard loads |
| 2 | Check JVM metrics panel | Data displayed |
| 3 | Check request rate panel | Data displayed |

---

## 9. Test Execution Summary

| Phase | Executed | Passed | Failed | Blocked | Pass Rate |
|-------|:--------:|:------:|:------:|:-------:|:---------:|
| Auth Flow (E2E) | 8 | — | — | — | — |
| Service Discovery | 6 | — | — | — | — |
| Gateway Routing | 10 | — | — | — | — |
| Security | 8 | — | — | — | — |
| Infrastructure | 6 | — | — | — | — |
| Observability | 4 | — | — | — | — |
| **Total** | **42** | **—** | **—** | **—** | **—** |

> Tests pending execution. Status will be updated after test run.

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Test plan governing these cases |
| [[../flowero_gate/04_testing/042_test_cases]] | Gate-level test cases |
| [[../flowero_discover/04_testing/042_test_cases]] | Discover-level test cases |
| [[../flowero_guard/04_testing/042_test_cases]] | Guard-level test cases |
| [[../01_requirement/011_business_objective]] | Business objectives (oracle) |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119
> **Usage:** Every cross-service interaction needs a test case. These cases trace to platform-level objectives (OBJ-01 through OBJ-05).
