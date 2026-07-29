---
document_type: Regression Test Suite
version: "0.1"
status: Draft
author: "QA Engineer"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [regression-test-suite, smoke-test, ci-cd, swebok, iso-29119, vrm, deerngo-bot]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Regression Test Suite

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

Defines reusable regression test suites — smoke tests for CI/CD and full regression for release verification. All test cases reference [[042_test_cases]] for detailed steps.

## 2. Suite Structure

| Suite | Purpose | Trigger | Duration | Tests |
|-------|---------|---------|:--------:|:-----:|
| **Smoke** | Critical path sanity check | Every PR merge | ~5 min | 12 |
| **Core Regression** | All 🔴 Must Have ACs | Per sprint release | ~30 min | 31 |
| **Full Regression** | All ACs (🔴 + 🟡) | Per phase release | ~60 min | 59 |
| **Manual Checklist** | streamer.bot + UI | Per stream / release | ~20 min | 18 |

---

## 3. Smoke Test Suite (CI/CD)

> Runs on every PR merge to `main`. Blocks deploy if any test fails. Target: < 5 minutes.

### 3.1 Smoke Test Selection

| # | Test Case | Module | Why In Smoke |
|---|-----------|--------|-------------|
| 1 | TC-007 | E-01 | Valid subscriber creation — core CRUD works |
| 2 | TC-009 | E-01 | Validation rejects bad input — safety net |
| 3 | TC-003 | E-01 | Upsert works — no duplicate subscribers |
| 4 | TC-036 | E-03 | Exact name match — core matching logic |
| 5 | TC-038 | E-03 | Anonymous donation excluded — edge case guard |
| 6 | TC-042 | E-03 | Points query returns correct total |
| 7 | TC-044 | E-03 | Non-existent handle returns 0 (not 404) |
| 8 | TC-032 | E-03 | Idempotent donation sync — no double-count |
| 9 | TC-052 | E-04 | Scoreboard API returns ranked data |
| 10 | TC-053 | E-04 | Empty scoreboard returns `[]` (not error) |
| 11 | TC-057 | Cross | Rate limiting triggers at 101 requests |
| 12 | TC-045 | E-03 | Only matched donations count toward points |

### 3.2 Smoke Test Execution

```bash
# Docker Compose — run smoke tests
docker compose -f docker-compose.test.yml up -d
go test -tags=smoke -v -timeout=5m ./...
docker compose -f docker-compose.test.yml down
```

### 3.3 Pass/Fail Criteria

| Criteria | Rule |
|----------|------|
| **Pass** | All 12 smoke tests pass |
| **Fail** | Any 1 smoke test fails → block PR merge |
| **Flaky** | 0 flaky tests allowed in smoke suite |

---

## 4. Core Regression Suite (Per Sprint Release)

> All 🔴 Must Have test cases. Runs before each sprint release.

### 4.1 Test Selection

| Epic | Test Cases | Count |
|------|-----------|:-----:|
| E-01 Subscriber Capture | TC-001, TC-002, TC-003, TC-004, TC-007, TC-008, TC-009, TC-012, TC-013, TC-014, TC-016, TC-017, TC-018 | 13 |
| E-02 Bot Commands | TC-019, TC-020, TC-023, TC-024, TC-025, TC-027, TC-028, TC-029 | 8 |
| E-03 Points Engine | TC-031, TC-032, TC-033, TC-036, TC-037, TC-038, TC-039, TC-042, TC-043, TC-044, TC-045 | 11 |
| E-04 Scoreboard | — | 0 |
| Cross-Cutting | TC-057, TC-058 | 2 |
| **Total** | | **34** |

> Note: E-02 Bot Commands includes 5 manual tests (TC-019, TC-020, TC-023, TC-024, TC-025, TC-027, TC-028, TC-029) that cannot run in CI.

### 4.2 Automated vs Manual Split

| Category | Count | Execution |
|----------|:-----:|-----------|
| Automated (CI) | 22 | `go test -tags=regression-core` |
| Manual | 12 | Manual checklist before release |
| **Total** | **34** | |

### 4.3 Pass/Fail Criteria

| Criteria | Rule |
|----------|------|
| **Pass** | All 34 tests pass (automated + manual) |
| **Fail** | Any 🔴 test fails → block release |
| **Known Issues** | Only 🟡 tests with documented DEF-* can be waived |

---

## 5. Full Regression Suite (Phase Release)

> All test cases (🔴 + 🟡). Runs before Phase 1 go-live.

### 5.1 Test Selection

| Module | Tests | Auto | Manual |
|--------|:-----:|:----:|:------:|
| E-01 Subscriber Capture | 18 | 14 | 4 |
| E-02 Bot Commands | 12 | 4 | 8 |
| E-03 Points Engine | 16 | 14 | 2 |
| E-04 Scoreboard | 10 | 6 | 4 |
| Cross-Cutting | 3 | 2 | 1 |
| **Total** | **59** | **40** | **19** |

### 5.2 Execution Order

| Order | Module | Tests | Reason |
|:-----:|--------|:-----:|--------|
| 1 | E-01 Subscriber Capture | 18 | Foundation — data must exist before other tests |
| 2 | E-03 Points Engine | 16 | Depends on subscriber data |
| 3 | E-02 Bot Commands | 12 | Depends on points data |
| 4 | E-04 Scoreboard | 10 | Depends on points data |
| 5 | Cross-Cutting | 3 | Rate limiting + HMAC — independent |

### 5.3 Pass/Fail Criteria

| Criteria | Rule |
|----------|------|
| **Go-Live** | All 59 tests pass |
| **Known Issues** | Max 3 waived 🟡 tests with DEF-* tracking |
| **Blockers** | Any 🔴 test failure blocks go-live |

---

## 6. Manual Test Checklist

> For streamer.bot integration (not automatable) and UI responsive design.

### 6.1 Pre-Stream Checklist (Per Live Stream)

| # | Test Case | Check | ☐ |
|---|-----------|-------|:-:|
| 1 | TC-019 | `:deer: donate` → bot responds with link | ☐ |
| 2 | TC-023 | `:deer: point` → bot shows points | ☐ |
| 3 | TC-024 | `:deer: point` for new viewer → shows 0 | ☐ |
| 4 | TC-027 | Donate action triggers correctly | ☐ |
| 5 | TC-028 | Point action calls API + shows result | ☐ |
| 6 | TC-030 | Actions logged in streamer.bot | ☐ |

### 6.2 Pre-Release Checklist

| # | Test Case | Check | ☐ |
|---|-----------|-------|:-:|
| 1 | TC-005 | Backend down → streamer.bot doesn't crash | ☐ |
| 2 | TC-020 | Bot offline → no error spam in chat | ☐ |
| 3 | TC-025 | Backend down → friendly error in chat | ☐ |
| 4 | TC-026 | API timeout → timeout error after 5s | ☐ |
| 5 | TC-029 | API 500 → fallback error message | ☐ |
| 6 | TC-049 | Scoreboard responsive on mobile | ☐ |
| 7 | TC-059 | HMAC key rotation works | ☐ |

---

## 7. Flaky Test Management

| Policy | Rule |
|--------|------|
| **Definition** | Test that passes/fails intermittently on same code |
| **Quarantine** | Move to separate suite, fix within 1 sprint |
| **Smoke tolerance** | 0 flaky tests in smoke suite |
| **Regression tolerance** | Max 2 flaky tests (marked with `# flaky` tag) |

---

## 8. CI/CD Integration

### 8.1 Pipeline Stages

```yaml
# GitHub Actions / CI pipeline
stages:
  - name: unit
    command: go test -v -race -cover ./...
    trigger: every push

  - name: integration
    command: go test -tags=integration -v -timeout=10m ./...
    trigger: every push

  - name: smoke
    command: go test -tags=smoke -v -timeout=5m ./...
    trigger: every PR merge to main

  - name: regression-core
    command: go test -tags=regression-core -v -timeout=30m ./...
    trigger: sprint release

  - name: regression-full
    command: go test -tags=regression-full -v -timeout=60m ./...
    trigger: phase release
```

### 8.2 Test Tags

| Tag | Purpose | Suite |
|-----|---------|-------|
| `smoke` | 12 critical path tests | CI/CD |
| `regression-core` | 22 automated 🔴 tests | Sprint release |
| `regression-full` | 40 automated tests | Phase release |
| `integration` | DB + API tests | Every push |
| `unit` | Service layer tests | Every push |

---

## 9. Test Data Reset

| Action | When | Method |
|--------|------|--------|
| Truncate + re-seed | Before each test run | `TRUNCATE subscribers, donations, viewer_points CASCADE;` + seed SQL |
| Fresh Docker Compose | Before CI run | `docker compose down -v && docker compose up -d` |
| Mock reset | Before each test | YouTube + EasyDonate mocks return to default state |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Test strategy governing these suites |
| [[042_test_cases]] | Detailed test cases referenced by suite |
| [[043_defect_report]] | Waived tests tracked here |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119
> **Usage:** Regression suites are the *gate* for releases. If the suite doesn't pass, the release doesn't ship.
