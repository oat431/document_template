---
document_type: Regression Test Suite
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Guard"
project_id: "flowero-guard"
classification: "Internal"
tags: [regression-testing, test-suite, keycloak, oauth, swebok]
standard_ref:
  - SWEBOK v4 — Testing
---

# Regression Test Suite — Flowero Guard

> **Project:** Flowero Guard (Keycloak IAM)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Regression testing ensures that changes to Keycloak configuration, realm exports, or container setup don't break OAuth flows, token validation, or user management.

## 2. Regression Strategy

| Scope | When | Tests | Duration |
|-------|------|:-----:|:--------:|
| Smoke | Every deployment | 5 | < 3 min |
| Targeted | Affected flows | Variable | < 15 min |
| Full Regression | Pre-release | 20 | < 45 min |

## 3. Smoke Tests (Every Deployment)

| # | Test ID | Test Name | Expected | Automated |
|---|---------|-----------|----------|:---------:|
| 1 | SMOKE-GU-001 | Guard health check | `GET :8001/health/ready` → 200 | ✅ |
| 2 | SMOKE-GU-002 | OIDC discovery endpoint | `GET /.well-known/openid-configuration` → 200 | ✅ |
| 3 | SMOKE-GU-003 | JWKS endpoint accessible | `GET /realms/panomete/protocol/openid-connect/certs` → 200 | ✅ |
| 4 | SMOKE-GU-004 | Token endpoint functional | `POST /token` with valid credentials → 200 | ✅ |
| 5 | SMOKE-GU-005 | Admin console accessible | `GET /admin` → login page | ✅ |

## 4. Full Regression Suite

### 4.1 OAuth2 Flows (8 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 1 | REG-GU-OAUTH-001 | Authorization Code flow complete | TC-G001 |
| 2 | REG-GU-OAUTH-002 | Client Credentials flow | TC-G010 |
| 3 | REG-GU-OAUTH-003 | Refresh Token flow | TC-G003 |
| 4 | REG-GU-OAUTH-004 | Token revocation | TC-G004 |
| 5 | REG-GU-OAUTH-005 | Token introspection | TC-G006 |
| 6 | REG-GU-OAUTH-006 | Invalid client rejected | TC-G008 |
| 7 | REG-GU-OAUTH-007 | Expired token rejected | TC-G009 |
| 8 | REG-GU-OAUTH-008 | Wrong issuer rejected | TC-G011 |

### 4.2 User Management (4 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 9 | REG-GU-USER-001 | User creation | TC-G020 |
| 10 | REG-GU-USER-002 | Role assignment | TC-G022 |
| 11 | REG-GU-USER-003 | User login | TC-G024 |
| 12 | REG-GU-USER-004 | User deletion | TC-G026 |

### 4.3 SSO & Sessions (4 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 13 | REG-GU-SSO-001 | SSO across services | TC-G030 |
| 14 | REG-GU-SSO-002 | Session timeout | TC-G031 |
| 15 | REG-GU-SSO-003 | Logout invalidates session | TC-G032 |
| 16 | REG-GU-SSO-004 | Concurrent sessions | TC-G033 |

### 4.4 Realm Configuration (4 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 17 | REG-GU-RC-001 | Realm import on startup | TC-G040 |
| 18 | REG-GU-RC-002 | Client registration | TC-G041 |
| 19 | REG-GU-RC-003 | Role configuration | TC-G042 |
| 20 | REG-GU-RC-004 | Realm export validation | TC-G043 |

## 5. Regression Triggers

| Trigger | Suite | Automation |
|---------|-------|:----------:|
| Realm JSON change | Smoke + OAuth flows | GitHub Actions |
| Keycloak version update | Full Regression | Manual trigger |
| Pre-release | Full Regression | Manual trigger |
| Nightly | Smoke | Scheduled |

## 6. Regression Metrics

| Metric | Target | Current | Status |
|--------|--------|:-------:|:------:|
| Regression pass rate | ≥ 95% | — | ⬜ |
| Regression execution time | < 45 min | — | ⬜ |
| Smoke test execution time | < 3 min | — | ⬜ |

## 7. Flaky Test Management

| Test | Issue | Frequency | Action |
|------|-------|:---------:|--------|
| TC-G031 | Session timeout timing varies | 10% | Increase tolerance to ±30s |
| TC-G040 | Realm import timing depends on DB | 8% | Add health check wait |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[042_test_cases]] | Full test case details |
| [[041_test_plan]] | Test plan governing regression |
| [[../05_devops/051_CICD_pipeline_configuration]] | CI/CD integration |

---

> **Template Standard:** Based on SWEBOK v4
> **Usage:** Run smoke tests on every deploy. Full regression before releases.
