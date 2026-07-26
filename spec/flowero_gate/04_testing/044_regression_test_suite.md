---
document_type: Regression Test Suite
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Gate"
project_id: "flowero-gate"
classification: "Internal"
tags: [regression-testing, test-suite, gateway, swebok]
standard_ref:
  - SWEBOK v4 — Testing
---

# Regression Test Suite — Flowero Gate

> **Project:** Flowero Gate (Spring Cloud Gateway)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Regression testing ensures that changes to Flowero Gate don't break existing functionality. Covers JWT auth, rate limiting, routing, CORS, and circuit breaker.

## 2. Regression Strategy

| Scope | When | Tests | Duration |
|-------|------|:-----:|:--------:|
| Smoke | Every deployment | 6 | < 3 min |
| Targeted | Affected modules | Variable | < 15 min |
| Full Regression | Pre-release | 25 | < 1 hour |

## 3. Smoke Tests (Every Deployment)

| # | Test ID | Test Name | Expected | Automated |
|---|---------|-----------|----------|:---------:|
| 1 | SMOKE-G-001 | Health endpoint accessible | `GET /actuator/health` → 200 | ✅ |
| 2 | SMOKE-G-002 | Protected endpoint rejects no-token | `GET /api/v1/**` → 401 | ✅ |
| 3 | SMOKE-G-003 | Protected endpoint accepts valid JWT | `GET /api/v1/**` with JWT → not 401/403 | ✅ |
| 4 | SMOKE-G-004 | Rate limiting enforced | Exceed limit → 429 | ✅ |
| 5 | SMOKE-G-005 | Route to business service | `GET /api/v1/todo/**` → proxied | ✅ |
| 6 | SMOKE-G-006 | Eureka registration | Gate appears as UP in Discover | ✅ |

## 4. Full Regression Suite

### 4.1 Authentication (10 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 1 | REG-G-AUTH-001 | Valid JWT accesses protected endpoint | GATE-TC-001 |
| 2 | REG-G-AUTH-002 | Missing JWT returns 401 | GATE-TC-002 |
| 3 | REG-G-AUTH-003 | Invalid JWT returns 401 | GATE-TC-003 |
| 4 | REG-G-AUTH-004 | Expired JWT returns 401 | GATE-TC-004 |
| 5 | REG-G-AUTH-005 | Public endpoint accessible without JWT | GATE-TC-005 |
| 6 | REG-G-AUTH-006 | Insufficient role returns 403 | GATE-TC-007 |
| 7 | REG-G-AUTH-007 | Browser redirect to Keycloak | GATE-TC-008 |
| 8 | REG-G-AUTH-008 | API returns JSON 401 (not redirect) | GATE-TC-009 |
| 9 | REG-G-AUTH-009 | Wrong issuer rejected | GATE-TC-006 |
| 10 | REG-G-AUTH-010 | Only RS256 accepted | GATE-TC-011 |

### 4.2 Rate Limiting (5 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 11 | REG-G-RL-001 | Rate limit per IP enforced | GATE-TC-101 |
| 12 | REG-G-RL-002 | Rate limit per user enforced | GATE-TC-102 |
| 13 | REG-G-RL-003 | Rate limit persists across restart | GATE-TC-103 |
| 14 | REG-G-RL-004 | 429 response format correct | GATE-TC-105 |
| 15 | REG-G-RL-005 | Valkey unavailable — fail open | GATE-TC-106 |

### 4.3 Routing (5 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 16 | REG-G-RT-001 | Route to business service | GATE-TC-201 |
| 17 | REG-G-RT-002 | Undefined route returns 404 | GATE-TC-202 |
| 18 | REG-G-RT-003 | Load balancer distributes traffic | GATE-TC-203 |
| 19 | REG-G-RT-004 | StripPrefix filter works | GATE-TC-205 |
| 20 | REG-G-RT-005 | Route timeout enforced | GATE-TC-206 |

### 4.4 CORS & Security (3 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 21 | REG-G-CORS-001 | CORS preflight allowed | GATE-TC-301 |
| 22 | REG-G-CORS-002 | Untrusted origin rejected | GATE-TC-302 |
| 23 | REG-G-CORS-003 | Credentials allowed for trusted origins | GATE-TC-303 |

### 4.5 Claim Forwarding (2 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 24 | REG-G-CF-001 | X-User-Id header forwarded | GATE-TC-501 |
| 25 | REG-G-CF-002 | X-User-Roles header forwarded | GATE-TC-503 |

## 5. Regression Triggers

| Trigger | Suite | Automation |
|---------|-------|:----------:|
| PR to main | Smoke (6 tests) | GitHub Actions |
| Merge to main | Smoke + Auth + Routing | GitHub Actions |
| Nightly | Full Regression | Scheduled |
| Pre-release | Full Regression | Manual trigger |
| Hotfix | Smoke | GitHub Actions |

## 6. Regression Metrics

| Metric | Target | Current | Status |
|--------|--------|:-------:|:------:|
| Regression pass rate | ≥ 95% | — | ⬜ |
| Regression execution time | < 1 hour | — | ⬜ |
| Smoke test execution time | < 3 min | — | ⬜ |
| Defects caught by regression | > 50% | — | ⬜ |

## 7. Flaky Test Management

| Test | Issue | Frequency | Action |
|------|-------|:---------:|--------|
| GATE-TC-103 | Valkey restart timing varies | 8% | Add retry with backoff |
| GATE-TC-203 | Load balancer distribution is probabilistic | 5% | Run 200 iterations |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[042_test_cases]] | Full test case details |
| [[041_test_plan]] | Test plan governing regression |
| [[../05_devops/051_CICD_pipeline_configuration]] | CI/CD integration |

---

> **Template Standard:** Based on SWEBOK v4
> **Usage:** Run smoke tests on every deploy. Fix flaky tests immediately.
