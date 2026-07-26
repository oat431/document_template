---
document_type: Coverage Report
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Guard"
project_id: "flowero-guard"
classification: "Internal"
tags: [coverage, integration-coverage, keycloak, oauth, swebok]
standard_ref:
  - SWEBOK v4 — Testing
---

# Coverage Report — Flowero Guard

> **Project:** Flowero Guard (Keycloak IAM)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Measures test coverage for Flowero Guard. Since Guard is a containerized Keycloak instance (no custom code), coverage focuses on configuration validation, OAuth flow coverage, and integration testing.

## 2. Coverage Types

| Type | Measurement | Target | Tool |
|------|-----------|:------:|------|
| Configuration Coverage | Realm config properties validated | 100% | Manual / CI |
| OAuth Flow Coverage | OAuth2/OIDC flows tested | 100% | Integration tests |
| Requirements Coverage | User stories with tests | 100% | Manual / RTM |
| Security Coverage | OWASP categories tested | 100% | Manual |

## 3. Configuration Coverage

| Configuration Area | Properties | Covered | Coverage |
|-------------------|:---------:|:-------:|:--------:|
| Realm Settings | 12 | 12 | 100% |
| Client Configuration | 8 | 8 | 100% |
| User Roles | 4 | 4 | 100% |
| Token Settings | 6 | 6 | 100% |
| Session Settings | 4 | 4 | 100% |
| **Total** | **34** | **34** | **100%** |

## 4. OAuth Flow Coverage

| Flow | Tested | Test Cases | Status |
|------|:------:|------------|:------:|
| Authorization Code | ✅ | TC-G001, TC-G002 | 🟢 |
| Authorization Code + PKCE | ✅ | TC-G002 | 🟢 |
| Client Credentials | ✅ | GUARD-TC-010 | 🟢 |
| Refresh Token | ✅ | TC-G003 | 🟢 |
| Token Revocation | ✅ | TC-G004 | 🟢 |
| Token Introspection | ✅ | GUARD-TC-006 | 🟢 |
| End Session (Logout) | ✅ | GUARD-TC-007 | 🟢 |
| UserInfo Endpoint | ✅ | GUARD-TC-005 | 🟢 |
| **Coverage** | **8/8** | | **100%** |

## 5. Requirements Coverage

| Category | Requirements | Covered | Coverage |
|----------|:-----------:|:-------:|:--------:|
| User Stories (US-001 to US-006) | 6 | 6 | 100% |
| Acceptance Criteria | 30 | 30 | 100% |
| Architecture Decisions (ADR-001) | 1 | 1 | 100% |
| **Total** | **37** | **37** | **100%** |

## 6. Security Coverage

| OWASP Category | Tested | Test Case | Status |
|---------------|:------:|-----------|:------:|
| A01 — Broken Access Control | ✅ | GUARD-TC-008, GUARD-TC-022 | 🟢 |
| A02 — Cryptographic Failures | ✅ | GUARD-TC-011, GUARD-TC-040 | 🟢 |
| A07 — Auth Failures | ✅ | TC-G001 to GUARD-TC-011 | 🟢 |
| A09 — Logging Failures | ✅ | GUARD-TC-043 | 🟢 |
| **Coverage** | **4/4** | | **100%** |

## 7. Integration Test Coverage

| Integration Point | Tested | Test Case | Status |
|------------------|:------:|-----------|:------:|
| Guard → PostgreSQL | ✅ | GUARD-TC-040 | 🟢 |
| Gate → Guard (JWKS) | ✅ | GUARD-TC-005 | 🟢 |
| Gate → Guard (Token) | ✅ | TC-G001 | 🟢 |
| Nginx → Guard | ✅ | Platform test PLAT-TC-206 | 🟢 |
| **Coverage** | **4/4** | | **100%** |

## 8. Coverage Gaps

| # | Gap | Impact | Action | Owner |
|---|-----|--------|--------|-------|
| 1 | No automated OAuth flow tests | Manual testing only | Add Postman/Newman collection | Dev |
| 2 | No load testing | Performance under load unknown | Add k6 or JMeter tests | QA |
| 3 | No MFA testing | MFA flows untested | Add MFA test cases | QA |
| 4 | Realm export not validated in CI | Config drift possible | Add JSON schema validation | DevOps |

## 9. Coverage Trends

| Period | Config | OAuth Flows | Integration | Security |
|--------|:------:|:----------:|:----------:|:--------:|
| Sprint 1 | 80% | 60% | 50% | 75% |
| Sprint 2 (current) | 100% | 100% | 100% | 100% |
| **Trend** | **↑** | **↑** | **↑** | **↑** |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[042_test_cases]] | Test cases providing coverage |
| [[041_test_plan]] | Test plan governing coverage targets |
| [[../03_construction/035_coding_standards_development]] | Configuration standards |

---

> **Template Standard:** Based on SWEBOK v4
> **Usage:** Guard is config-only. Focus on flow coverage and configuration validation.
