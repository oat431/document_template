---
document_type: Security Test Report
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Panomete Platform"
project_id: "PAN-PLAT-001"
classification: "Confidential"
tags: [security-testing, vulnerability, pen-test, platform, owasp]
standard_ref:
  - SWEBOK v4 — Testing
  - OWASP Testing Guide
---

# Security Test Report — Panomete Platform

> **Project:** Panomete Platform
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Reports security testing results for the Panomete Platform at the integration and platform level. Covers authentication, authorization, network security, configuration security, and OWASP Top 10 assessment.

## 2. Security Test Summary

| Field | Detail |
|-------|--------|
| **Test Date** | 2026-07-26 |
| **Test Type** | Manual security review + automated scanning |
| **Tools** | OWASP ZAP, curl, manual inspection |
| **Tester** | QA Engineer |
| **Scope** | Platform-wide (Guard, Discover, Gate, Nginx, Cloudflare) |
| **Overall Risk** | 🟡 Medium |

## 3. Vulnerability Summary

| Severity | Found | Fixed | Remaining | Status |
|----------|:-----:|:-----:|:---------:|:------:|
| 🔴 Critical | 0 | 0 | 0 | ✅ Clean |
| 🟠 High | 3 | 0 | 3 | 🟠 Open |
| 🟡 Medium | 4 | 0 | 4 | 🟡 Open |
| 🟢 Low | 1 | 0 | 1 | 🟢 Acceptable |
| **Total** | **8** | **0** | **8** | **🟡** |

## 4. OWASP Top 10 Assessment

| # | Category | Status | Notes |
|---|---------|:------:|-------|
| A01 | Broken Access Control | ✅ Pass | RBAC implemented at Gate and service level; UFW firewall blocks direct port access |
| A02 | Cryptographic Failures | ✅ Pass | TLS 1.3 via Cloudflare; JWT signed with RS256; secrets in env vars, not git |
| A03 | Injection | ✅ Pass | Parameterized queries (PostgreSQL); input validation at Gate |
| A04 | Insecure Design | ✅ Pass | Architecture reviewed; threat modeling completed; ADRs document decisions |
| A05 | Security Misconfiguration | 🟡 Minor | JWKS cache not refreshed on key rotation (PDEF-002); Eureka self-preservation delays deregistration |
| A06 | Vulnerable Components | ✅ Pass | Spring Boot 4.1.0 (latest); Spring Cloud 2025.1.2 (latest); no known CVEs |
| A07 | Auth Failures | 🟡 Minor | CORS preflight returns 401 (PDEF-006); client_credentials grants missing claim headers (PDEF-007) |
| A08 | Data Integrity Failures | ✅ Pass | Secrets not in git; .env in .gitignore; Docker network isolation |
| A09 | Logging Failures | ✅ Pass | Structured JSON logs at Gate; audit trail for auth events |
| A10 | SSRF | ✅ Pass | No server-side request functionality exposed |

## 5. Findings

### Finding 1: High — JWKS Cache Not Refreshed

| Field | Detail |
|-------|--------|
| **ID** | PSEC-001 |
| **Severity** | 🟠 High |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Flowero Gate (`SecurityConfig.java`) |
| **Vulnerability** | Gate caches Keycloak JWKS on startup and never refreshes. After key rotation, all tokens signed with new keys are rejected until Gate restarts. |
| **Impact** | Service disruption after Keycloak key rotation. Users cannot authenticate until Gate is manually restarted. |
| **Remediation** | Configure `JwkSetUriReactiveJwtDecoder` with cache TTL (e.g., 5 minutes). Implement JWKS refresh endpoint or auto-refresh on 401. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-002 |

### Finding 2: High — CORS Preflight Returns 401

| Field | Detail |
|-------|--------|
| **ID** | PSEC-002 |
| **Severity** | 🟠 High |
| **Category** | A07 — Auth Failures |
| **Component** | Flowero Gate (`SecurityConfig.java`, `CorsConfig.java`) |
| **Vulnerability** | Browser CORS preflight (OPTIONS) requests to authenticated routes return 401. Browsers don't send JWT on preflight, so the preflight fails and blocks the actual request. |
| **Impact** | Frontend applications on different origins cannot call Gate APIs. Complete blocker for cross-origin browser clients. |
| **Remediation** | Ensure `CorsWebFilter` is ordered before `SecurityWebFilterChain`. Add OPTIONS to permitted path matchers in `SecurityConfig`. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-006 |

### Finding 3: High — Rate Limit Counters Lost on Valkey Restart

| Field | Detail |
|-------|--------|
| **ID** | PSEC-003 |
| **Severity** | 🟠 High |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Valkey, Flowero Gate (`RateLimiterConfig.java`) |
| **Vulnerability** | Valkey rate limit counters are lost when Valkey restarts. Clients that were rate-limited can immediately resume full request rates. |
| **Impact** | Transient security gap. Rate-limited clients bypass limits after Valkey restart. |
| **Remediation** | Enable Valkey RDB persistence (`save 60 1000`) or AOF (`appendonly yes`). Alternatively, accept as known limitation for homelab. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-003 |

### Finding 4: Medium — Client Credentials Grants Missing Claim Headers

| Field | Detail |
|-------|--------|
| **ID** | PSEC-004 |
| **Severity** | 🟡 Medium |
| **Category** | A07 — Auth Failures |
| **Component** | Flowero Gate (`JwtClaimHeaderFilter.java`) |
| **Vulnerability** | `JwtClaimHeaderFilter` does not forward claims for client_credentials tokens (no `email` or `sub` user claim). Downstream services cannot identify the calling service. |
| **Impact** | Service-to-service calls lack audit trail. Downstream services cannot enforce per-client rate limits or permissions. |
| **Remediation** | Extend `JwtClaimHeaderFilter` to extract `client_id` claim and forward as `X-Client-Id` header. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-007 |

### Finding 5: Medium — Eureka Self-Preservation Delays Deregistration

| Field | Detail |
|-------|--------|
| **ID** | PSEC-005 |
| **Severity** | 🟡 Medium |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Flowero Discover (Eureka configuration) |
| **Vulnerability** | Eureka self-preservation mode prevents stale entries from being evicted for up to 90 seconds after service crash. Gate may route traffic to dead services. |
| **Impact** | 502 Bad Gateway errors during service crashes. Poor user experience. |
| **Remediation** | Configure Docker health checks to detect crashes faster. Consider `eureka.server.enable-self-preservation=false` for single-instance deployments. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-004, PDEF-008 |

### Finding 6: Medium — Nginx 502 During Guard Startup

| Field | Detail |
|-------|--------|
| **ID** | PSEC-006 |
| **Severity** | 🟡 Medium |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Nginx, Flowero Guard |
| **Vulnerability** | Nginx returns 502 during Keycloak startup (15-30 seconds). Requests to `auth.panomete.com` fail during this window. |
| **Impact** | Authentication unavailable during platform startup. |
| **Remediation** | Add Nginx `proxy_next_upstream` with retry. Configure Docker Compose health check to delay Nginx routing until Guard is ready. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-005 |

### Finding 7: Medium — Gate Returns 502 for Unregistered Services

| Field | Detail |
|-------|--------|
| **ID** | PSEC-007 |
| **Severity** | 🟡 Medium |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Flowero Gate, Flowero Discover |
| **Vulnerability** | When a business service is starting but not yet registered in Eureka, Gate returns 502 Bad Gateway instead of 503 Service Unavailable. |
| **Impact** | Clients cannot distinguish between "service starting" and "service down." |
| **Remediation** | Configure circuit breaker to return 503 with `Retry-After` header when Eureka returns no instances. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-001 |

### Finding 8: Low — Eureka Self-Preservation Mode Enabled

| Field | Detail |
|-------|--------|
| **ID** | PSEC-008 |
| **Severity** | 🟢 Low |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Flowero Discover |
| **Vulnerability** | Eureka self-preservation mode is enabled by default. In small deployments, network jitter can trigger it, preventing timely deregistration. |
| **Impact** | Stale service entries in registry. |
| **Remediation** | Document as expected behavior. Consider disabling for homelab environment. |
| **Status** | ⬜ Open |
| **Related Defect** | PDEF-008 |

## 6. Penetration Test Results

| Test | Result | Notes |
|------|:------:|-------|
| SQL Injection | ✅ Pass | Parameterized queries (PostgreSQL) |
| XSS | ✅ Pass | No user input rendered in HTML |
| CSRF | ✅ Pass | CSRF disabled for API (stateless JWT) |
| Authentication Bypass | ✅ Pass | Cannot bypass JWT validation at Gate |
| Authorization Bypass | ✅ Pass | RBAC enforced at Gate and service level |
| Session Management | ✅ Pass | Stateless JWT; no session hijacking |
| File Upload | ✅ Pass | No file upload functionality at platform level |
| API Security | 🟡 Minor | CORS preflight issue (PSEC-002) |
| Network Security | ✅ Pass | UFW firewall; Docker network isolation |
| Secret Management | ✅ Pass | Secrets in env vars; not in git |

## 7. Security Recommendations

| # | Recommendation | Priority | Status |
|---|---------------|:--------:|:------:|
| 1 | Configure JWKS cache TTL at Gate | 🔴 | ⬜ Open |
| 2 | Fix CORS preflight authentication | 🔴 | ⬜ Open |
| 3 | Enable Valkey persistence for rate limit counters | 🔴 | ⬜ Open |
| 4 | Extend `JwtClaimHeaderFilter` for client_credentials | 🟡 | ⬜ Open |
| 5 | Disable Eureka self-preservation or improve health checks | 🟡 | ⬜ Open |
| 6 | Add Nginx retry logic for Guard startup | 🟡 | ⬜ Open |
| 7 | Configure Gate circuit breaker for unregistered services | 🟡 | ⬜ Open |
| 8 | Document Eureka self-preservation behavior | 🟢 | ⬜ Open |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[043_defect_report]] | Platform-level defects |
| [[062_coding_standards_security]] | Security coding standards |
| [[../flowero_gate/06_security/061_security_test_report]] | Gate-specific security report |
| [[../flowero_discover/06_security/061_security_test_report]] | Discover-specific security report |
| [[../flowero_guard/06_security/061_security_test_report]] | Guard-specific security report |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Testing Guide
> **Usage:** Security testing is not optional. Address all high-severity findings before production release.
