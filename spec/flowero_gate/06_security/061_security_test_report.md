---
document_type: Security Test Report
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Gate"
project_id: "flowero-gate"
classification: "Confidential"
tags: [security-testing, vulnerability, gateway, owasp]
standard_ref:
  - SWEBOK v4 — Testing
  - OWASP Testing Guide
---

# Security Test Report — Flowero Gate

> **Project:** Flowero Gate (Spring Cloud Gateway)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Reports security testing results for Flowero Gate, the API Gateway. Covers JWT validation, rate limiting, CORS, input validation, and OWASP API Security Top 10.

## 2. Security Test Summary

| Field | Detail |
|-------|--------|
| **Test Date** | 2026-07-26 |
| **Test Type** | Code review + manual testing |
| **Tools** | OWASP ZAP, curl, manual inspection |
| **Tester** | QA Engineer |
| **Scope** | Flowero Gate (Spring Cloud Gateway) |
| **Overall Risk** | 🟡 Medium |

## 3. Vulnerability Summary

| Severity | Found | Fixed | Remaining | Status |
|----------|:-----:|:-----:|:---------:|:------:|
| 🔴 Critical | 0 | 0 | 0 | ✅ Clean |
| 🟠 High | 2 | 0 | 2 | 🟠 Open |
| 🟡 Medium | 3 | 0 | 3 | 🟡 Open |
| 🟢 Low | 1 | 0 | 1 | 🟢 Acceptable |
| **Total** | **6** | **0** | **6** | **🟡** |

## 4. OWASP API Security Top 10 Assessment

| # | Category | Status | Notes |
|---|---------|:------:|-------|
| API1 | Broken Object Level Authorization | ✅ Pass | RBAC via JWT roles; downstream services enforce resource-level auth |
| API2 | Broken Authentication | 🟡 Minor | JWKS cache not refreshed (GSEC-001); CORS preflight issue (GSEC-002) |
| API3 | Broken Object Property Level Authorization | ✅ Pass | JWT claim forwarding is read-only; no mass assignment |
| API4 | Unrestricted Resource Consumption | 🟡 Minor | Rate limiting via Valkey; but Valkey restart resets counters (GSEC-003) |
| API5 | Broken Function Level Authorization | ✅ Pass | Path-based authorization in `SecurityConfig`; public vs protected routes |
| API6 | Unrestricted Access to Sensitive Business Flows | ✅ Pass | Rate limiting applied to all routes |
| API7 | Server Side Request Forgery | ✅ Pass | No SSRF surface; routes are pre-configured |
| API8 | Security Misconfiguration | 🟡 Minor | CORS preflight returns 401 (GSEC-002); security headers not configured |
| API9 | Improper Inventory Management | ✅ Pass | Routes defined in YAML; no dynamic route injection |
| API10 | Unsafe Consumption of APIs | ✅ Pass | JWT validated locally; no unsafe downstream calls |

## 5. Findings

### GSEC-001: High — JWKS Cache Never Refreshes

| Field | Detail |
|-------|--------|
| **ID** | GSEC-001 |
| **Severity** | 🟠 High |
| **Category** | API2 — Broken Authentication |
| **Component** | `SecurityConfig.java` |
| **Vulnerability** | Gate caches Keycloak JWKS on startup and never refreshes. After key rotation, all new tokens rejected until Gate restarts. |
| **Impact** | Complete authentication failure after key rotation. Requires manual Gate restart. |
| **Remediation** | Configure `NimbusReactiveJwtDecoder` with `.cache(Duration.ofMinutes(5))`. |
| **Status** | ⬜ Open |

### GSEC-002: High — CORS Preflight Returns 401

| Field | Detail |
|-------|--------|
| **ID** | GSEC-002 |
| **Severity** | 🟠 High |
| **Category** | API8 — Security Misconfiguration |
| **Component** | `SecurityConfig.java`, `CorsConfig.java` |
| **Vulnerability** | OPTIONS preflight requests to authenticated routes return 401. Browsers block the actual request. |
| **Impact** | Cross-origin browser clients completely blocked from calling Gate APIs. |
| **Remediation** | Add `HttpMethod.OPTIONS` to permitted matchers or order `CorsWebFilter` before security. |
| **Status** | ⬜ Open |

### GSEC-003: Medium — Rate Limit Counters Lost on Valkey Restart

| Field | Detail |
|-------|--------|
| **ID** | GSEC-003 |
| **Severity** | 🟡 Medium |
| **Category** | API4 — Unrestricted Resource Consumption |
| **Component** | `ResilientRedisRateLimiter.java`, Valkey |
| **Vulnerability** | Rate limit counters reset when Valkey restarts. Rate-limited clients can resume full rates. |
| **Impact** | Transient security gap after Valkey restart. |
| **Remediation** | Enable Valkey persistence (RDB or AOF). |
| **Status** | ⬜ Open |

### GSEC-004: Medium — Missing Security Headers

| Field | Detail |
|-------|--------|
| **ID** | GSEC-004 |
| **Severity** | 🟡 Medium |
| **Category** | API8 — Security Misconfiguration |
| **Component** | `SecurityConfig.java` |
| **Vulnerability** | No security headers configured: missing `X-Content-Type-Options`, `X-Frame-Options`, `X-XSS-Protection`, `Strict-Transport-Security`, `Content-Security-Policy`. |
| **Impact** | Reduced defense-in-depth. Browser-level protections not activated. |
| **Remediation** | Add `headers()` configuration to `SecurityWebFilterChain`. |
| **Status** | ⬜ Open |

### GSEC-005: Medium — TraceIdFilter Overwrites Existing Trace ID

| Field | Detail |
|-------|--------|
| **ID** | GSEC-005 |
| **Severity** | 🟡 Medium |
| **Category** | API8 — Security Misconfiguration |
| **Component** | `TraceIdFilter.java` |
| **Vulnerability** | Always generates new trace ID, overwriting any upstream trace ID from Nginx/Cloudflare. |
| **Impact** | Breaks distributed tracing chain. Makes debugging harder. |
| **Remediation** | Check for existing `X-Request-Id` or `X-B3-TraceId` before generating. |
| **Status** | ⬜ Open |

### GSEC-006: Low — SensitiveDataMasker Misses IP Addresses

| Field | Detail |
|-------|--------|
| **ID** | GSEC-006 |
| **Severity** | 🟢 Low |
| **Category** | API8 — Security Misconfiguration |
| **Component** | `SensitiveDataMasker.java` |
| **Vulnerability** | Does not mask IP addresses in `X-Forwarded-For` header in logs. |
| **Impact** | Client IP addresses visible in logs. Low risk for internal traffic. |
| **Remediation** | Mask all but last octet of IP addresses. |
| **Status** | ⬜ Open |

## 6. Penetration Test Results

| Test | Result | Notes |
|------|:------:|-------|
| JWT Forgery | ✅ Pass | RS256 signature validated; HS256 rejected |
| Token Replay | ✅ Pass | Expired tokens rejected (5-min TTL) |
| Authentication Bypass | ✅ Pass | Cannot bypass JWT validation |
| Authorization Bypass | ✅ Pass | RBAC enforced; 403 for insufficient roles |
| Rate Limit Bypass | ✅ Pass | Per-IP and per-user limiting |
| Route Injection | ✅ Pass | Only pre-configured routes accepted |
| Header Injection | ✅ Pass | Input validation at filter level |
| CORS Misconfiguration | 🟡 Minor | Preflight issue (GSEC-002) |

## 7. Security Recommendations

| # | Recommendation | Priority | Status |
|---|---------------|:--------:|:------:|
| 1 | Configure JWKS cache TTL | 🔴 | ⬜ Open |
| 2 | Fix CORS preflight authentication | 🔴 | ⬜ Open |
| 3 | Enable Valkey persistence | 🟡 | ⬜ Open |
| 4 | Add security headers | 🟡 | ⬜ Open |
| 5 | Preserve upstream trace IDs | 🟡 | ⬜ Open |
| 6 | Mask IP addresses in logs | 🟢 | ⬜ Open |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[043_defect_report]] | Gate-specific defects |
| [[062_coding_standards_security]] | Security coding standards |
| [[../../panomete_platform/06_security/061_security_test_report]] | Platform-wide security report |

---

> **Template Standard:** Based on SWEBOK v4, OWASP API Security Top 10
> **Usage:** Address all high-severity findings before production release.
