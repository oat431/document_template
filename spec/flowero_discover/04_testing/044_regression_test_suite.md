---
document_type: Regression Test Suite
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Discover"
project_id: "flowero-discover"
classification: "Internal"
tags: [regression-testing, test-suite, eureka, discovery, swebok]
standard_ref:
  - SWEBOK v4 — Testing
---

# Regression Test Suite — Flowero Discover

> **Project:** Flowero Discover (Spring Cloud Netflix Eureka)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Regression testing ensures that changes to Flowero Discover don't break service registration, discovery, or health tracking.

## 2. Regression Strategy

| Scope | When | Tests | Duration |
|-------|------|:-----:|:--------:|
| Smoke | Every deployment | 4 | < 2 min |
| Targeted | Affected modules | Variable | < 10 min |
| Full Regression | Pre-release | 15 | < 30 min |

## 3. Smoke Tests (Every Deployment)

| # | Test ID | Test Name | Expected | Automated |
|---|---------|-----------|----------|:---------:|
| 1 | SMOKE-D-001 | Health endpoint accessible | `GET /actuator/health` → `{"status":"UP"}` | ✅ |
| 2 | SMOKE-D-002 | Dashboard accessible | `GET /` → Eureka dashboard HTML | ✅ |
| 3 | SMOKE-D-003 | Registered services visible | `GET /eureka/apps` → XML with registered apps | ✅ |
| 4 | SMOKE-D-004 | Self-preservation status | Dashboard shows correct mode | ✅ |

## 4. Full Regression Suite

### 4.1 Registration (5 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 1 | REG-D-REG-001 | Service registers on startup | TC-001 |
| 2 | REG-D-REG-002 | Service deregisters on graceful shutdown | TC-002 |
| 3 | REG-D-REG-003 | Service re-registers after restart | TC-003 |
| 4 | REG-D-REG-004 | Registration within 30 seconds | TC-004 |
| 5 | REG-D-REG-005 | Multiple instances of same service | TC-005 |

### 4.2 Discovery (4 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 6 | REG-D-DISC-001 | Service discovery by name | TC-101 |
| 7 | REG-D-DISC-002 | Discovery returns all instances | TC-102 |
| 8 | REG-D-DISC-003 | Discovery query < 100ms p95 | TC-103 |
| 9 | REG-D-DISC-004 | Stale entries evicted within 90s | TC-104 |

### 4.3 Dashboard & Health (3 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 10 | REG-D-DASH-001 | Dashboard shows all services | TC-201 |
| 11 | REG-D-DASH-002 | Dashboard accessible via Nginx | TC-202 |
| 12 | REG-D-DASH-003 | Health status reflects reality | TC-203 |

### 4.4 Configuration (3 tests)

| # | Test ID | Test Name | Traces To |
|---|---------|-----------|-----------|
| 13 | REG-D-CFG-001 | Standalone mode configured | DISC-TC-301 |
| 14 | REG-D-CFG-002 | Self-preservation configurable | DISC-TC-302 |
| 15 | REG-D-CFG-003 | Eviction timeout configurable | DISC-TC-303 |

## 5. Regression Triggers

| Trigger | Suite | Automation |
|---------|-------|:----------:|
| PR to main | Smoke (4 tests) | GitHub Actions |
| Merge to main | Smoke + Registration | GitHub Actions |
| Nightly | Full Regression | Scheduled |
| Pre-release | Full Regression | Manual trigger |

## 6. Regression Metrics

| Metric | Target | Current | Status |
|--------|--------|:-------:|:------:|
| Regression pass rate | ≥ 95% | — | ⬜ |
| Regression execution time | < 30 min | — | ⬜ |
| Smoke test execution time | < 2 min | — | ⬜ |

## 7. Flaky Test Management

| Test | Issue | Frequency | Action |
|------|-------|:---------:|--------|
| TC-104 | Eviction timing varies | 12% | Increase wait to 120s |
| TC-004 | Registration timing depends on JVM startup | 8% | Retry with backoff |

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
