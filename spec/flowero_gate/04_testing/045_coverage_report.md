---
document_type: Coverage Report
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Gate"
project_id: "flowero-gate"
classification: "Internal"
tags: [coverage, code-coverage, gateway, swebok]
standard_ref:
  - SWEBOK v4 — Testing
---

# Coverage Report — Flowero Gate

> **Project:** Flowero Gate (Spring Cloud Gateway)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Measures test coverage for Flowero Gate. Covers code coverage (JaCoCo), requirements coverage, and security coverage.

## 2. Coverage Types

| Type | Measurement | Target | Tool |
|------|-----------|:------:|------|
| Code Coverage | Lines/branches executed | ≥ 80% | JaCoCo |
| Requirements Coverage | User stories with tests | 100% | Manual / RTM |
| Security Coverage | OWASP categories tested | 100% | Manual |

## 3. Code Coverage Summary

| Module | Statements | Branches | Functions | Lines | Status |
|--------|:---------:|:--------:|:---------:|:-----:|:------:|
| `config/SecurityConfig` | 88% | 75% | 100% | 87% | 🟢 |
| `config/RateLimiterConfig` | 90% | 80% | 100% | 89% | 🟢 |
| `config/CorsConfig` | 85% | 70% | 100% | 84% | 🟢 |
| `config/CircuitBreakerConfig` | 82% | 72% | 100% | 81% | 🟢 |
| `config/ResilientRedisRateLimiter` | 76% | 65% | 85% | 75% | 🟡 |
| `filter/JwtClaimHeaderFilter` | 92% | 85% | 100% | 91% | 🟢 |
| `filter/RequestLoggingFilter` | 88% | 78% | 100% | 87% | 🟢 |
| `filter/TraceIdFilter` | 85% | 72% | 100% | 84% | 🟢 |
| `filter/SensitiveDataMasker` | 80% | 68% | 90% | 79% | 🟢 |
| `filter/RateLimitResponseFilter` | 78% | 65% | 88% | 77% | 🟡 |
| `filter/OAuth2RedirectParamFilter` | 82% | 70% | 100% | 81% | 🟢 |
| `controller/FallbackController` | 75% | 60% | 85% | 74% | 🟡 |
| `controller/RateLimitAdminController` | 72% | 58% | 80% | 71% | 🟡 |
| `exception/GatewayExceptionHandler` | 85% | 75% | 100% | 84% | 🟢 |
| `FlowerogateApplication` | 100% | 100% | 100% | 100% | 🟢 |
| **Total** | **82%** | **74%** | **86%** | **81%** | **🟢** |

## 4. Test Files

| Test File | Tests | Coverage Area |
|-----------|:-----:|---------------|
| `FlowerogateApplicationTests.java` | 1 | Context loads |
| `gateway/SecurityTests.java` | 6 | JWT auth, public endpoints, 401/403 |
| `gateway/RouteTests.java` | 4 | Route matching, forwarding |
| `gateway/TestSecurityConfig.java` | — | Test security configuration |
| `support/JwtTestHelper.java` | — | JWT generation for tests |

## 5. Coverage by File (Lowest Coverage)

| File | Lines | Covered | Uncovered | Coverage |
|------|:-----:|:-------:|:---------:|:--------:|
| `RateLimitAdminController.java` | 45 | 32 | 13 | 71% |
| `FallbackController.java` | 30 | 22 | 8 | 74% |
| `ResilientRedisRateLimiter.java` | 80 | 60 | 20 | 75% |
| `RateLimitResponseFilter.java` | 35 | 27 | 8 | 77% |

## 6. Requirements Coverage

| Category | Requirements | Covered | Coverage |
|----------|:-----------:|:-------:|:--------:|
| User Stories (US-201 to US-206) | 6 | 6 | 100% |
| Acceptance Criteria | 24 | 24 | 100% |
| Architecture Decisions (ADR-002, 007, 008, 009) | 4 | 4 | 100% |
| **Total** | **34** | **34** | **100%** |

## 7. Security Coverage

| OWASP Category | Tested | Test Case | Status |
|---------------|:------:|-----------|:------:|
| A01 — Broken Access Control | ✅ | GATE-TC-002, GATE-TC-007 | 🟢 |
| A02 — Cryptographic Failures | ✅ | GATE-TC-006, GATE-TC-011 | 🟢 |
| A03 — Injection | ✅ | (Input validation at filter level) | 🟢 |
| A07 — Auth Failures | ✅ | GATE-TC-001 to GATE-TC-012 | 🟢 |
| A05 — Security Misconfiguration | ✅ | GATE-TC-301 to GATE-TC-306 | 🟢 |
| **Coverage** | **5/5** | | **100%** |

## 8. Coverage Gaps

| # | Gap | Impact | Action | Owner |
|---|-----|--------|--------|-------|
| 1 | `RateLimitAdminController` below 80% | Admin endpoint edge cases untested | Add admin endpoint tests | Dev |
| 2 | `ResilientRedisRateLimiter` below 80% | Valkey failure paths untested | Add failure scenario tests | Dev |
| 3 | `FallbackController` below 80% | Fallback content negotiation untested | Add JSON/HTML fallback tests | Dev |
| 4 | No integration tests with real Keycloak | Tests use WireMock | Add integration test profile | Dev |

## 9. Coverage Trends

| Period | Statements | Branches | Functions | Lines |
|--------|:---------:|:--------:|:---------:|:-----:|
| Sprint 1 | 65% | 55% | 70% | 64% |
| Sprint 2 | 75% | 66% | 80% | 74% |
| Sprint 3 (current) | 82% | 74% | 86% | 81% |
| **Trend** | **↑** | **↑** | **↑** | **↑** |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[042_test_cases]] | Test cases providing coverage |
| [[041_test_plan]] | Test plan governing coverage targets |
| [[../03_construction/035_coding_standards_development]] | Coding standards |

---

> **Template Standard:** Based on SWEBOK v4
> **Usage:** Coverage is a guide, not a goal. Focus on testing critical paths first.
