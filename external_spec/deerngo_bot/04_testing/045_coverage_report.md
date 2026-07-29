---
document_type: Coverage Report
version: "0.1"
status: Draft
author: "QA Engineer"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [coverage-report, requirements-coverage, code-coverage, swebok, vrm, deerngo-bot]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
  - ISO/IEC 25010 — Software Quality
---

# Coverage Report

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

Tracks testing coverage across three dimensions: requirements coverage (ACs → test cases), code coverage (when tests are runnable), and test type coverage. This is the baseline report before code exists.

---

## 2. Requirements Coverage

### 2.1 AC Coverage Summary

| Metric | Count | Percentage |
|--------|:-----:|:----------:|
| Total Acceptance Criteria | 54 | 100% |
| ACs with test cases | 54 | **100%** |
| ACs without test cases | 0 | 0% |
| 🔴 Must Have ACs covered | 31/31 | **100%** |
| 🟡 Should Have ACs covered | 23/23 | **100%** |

### 2.2 Coverage by Epic

| Epic | ACs | Covered | Coverage | Status |
|------|:---:|:-------:|:--------:|--------|
| E-01 Subscriber Capture | 18 | 18 | 100% | 🟢 |
| E-02 Bot Commands | 12 | 12 | 100% | 🟢 |
| E-03 Points Engine | 16 | 16 | 100% | 🟢 |
| E-04 Scoreboard | 10 | 10 | 100% | 🟢 |
| Cross-Cutting (Rate Limiting + HMAC) | — | 3 TCs | — | 🟢 |

### 2.3 Coverage by User Story

| User Story | ACs | 🔴 | 🟡 | Test Cases | Coverage |
|------------|:---:|:---:|:---:|:----------:|:--------:|
| US-001 Capture Subscriber | 6 | 4 | 2 | TC-001 → TC-006 | 100% |
| US-002 Subscriber API | 5 | 3 | 2 | TC-007 → TC-011 | 100% |
| US-003 YouTube API Polling | 7 | 5 | 2 | TC-012 → TC-018 | 100% |
| US-010 Donate Command | 4 | 2 | 2 | TC-019 → TC-022 | 100% |
| US-011 Point Command | 4 | 3 | 1 | TC-023 → TC-026 | 100% |
| US-012 SB Action Config | 4 | 3 | 1 | TC-027 → TC-030 | 100% |
| US-020 Donation Sync | 5 | 3 | 2 | TC-031 → TC-035 | 100% |
| US-021 Name Matching | 6 | 4 | 2 | TC-036 → TC-041 | 100% |
| US-022 Point Query API | 5 | 4 | 1 | TC-042 → TC-046 | 100% |
| US-030 Scoreboard Page | 5 | 0 | 5 | TC-047 → TC-051 | 100% |
| US-031 Scoreboard API | 5 | 0 | 5 | TC-052 → TC-056 | 100% |

### 2.4 Gap Analysis

| Gap | Description | Status |
|-----|-------------|--------|
| AC-001f (display_name fallback) | Contradicted API spec. PO decided: remove/rewrite AC. TC-006 updated. | ✅ Resolved (DEF-S001) |
| Rate limiting | No ACs existed. PO decided: add ACs. TC-057, TC-058 added. | ✅ Resolved (DEF-S004) |
| HMAC key rotation | No ACs existed. PO decided: document now. TC-059 added. | ✅ Resolved (DEF-S005) |

> **Result:** 0 requirements coverage gaps remaining.

---

## 3. Test Case Coverage

### 3.1 Test Case Summary

| Metric | Count |
|--------|:-----:|
| Total test cases | 59 |
| Mapped to ACs | 56 |
| Additional (rate limiting + HMAC) | 3 |
| Automated (unit + integration) | 40 |
| Manual (streamer.bot + UI) | 19 |

### 3.2 Test Type Distribution

| Type | Count | Percentage | Automation |
|------|:-----:|:----------:|:----------:|
| Unit | 14 | 24% | 100% |
| Integration | 26 | 44% | 100% |
| System | 7 | 12% | 86% |
| Manual | 12 | 20% | 0% |
| **Total** | **59** | **100%** | **68%** |

### 3.3 Priority Distribution

| Priority | Count | Percentage |
|----------|:-----:|:----------:|
| 🔴 Critical | 31 | 53% |
| 🟡 High | 28 | 47% |
| **Total** | **59** | **100%** |

---

## 4. Code Coverage

> **Status:** Not yet measured — code does not exist. Will be populated after Dev implements and tests are runnable.

### 4.1 Target Coverage

| Layer | Target | Actual | Status |
|-------|:------:|:------:|--------|
| Service layer (Go) | ≥ 80% | — | ⬜ Pending |
| Handler layer (Fiber) | ≥ 60% | — | ⬜ Pending |
| Repository layer (sqlx) | ≥ 70% | — | ⬜ Pending |
| Overall | ≥ 70% | — | ⬜ Pending |

### 4.2 Measurement Method

```bash
# Go coverage — run after implementation
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out    # Per-function summary
go tool cover -html=coverage.out    # HTML report
```

### 4.3 Coverage by Module (Projected)

| Module | Key Files | Test Focus | Expected Coverage |
|--------|-----------|-----------|:-----------------:|
| E-01 Subscriber | `handler/subscriber.go`, `service/subscriber.go`, `repo/subscriber.go` | Upsert logic, validation | ≥ 80% |
| E-02 Bot Commands | `handler/points.go` | Point query, error handling | ≥ 70% |
| E-03 Points Engine | `service/matcher.go`, `service/sync.go`, `service/points.go` | Matching algorithm, sync logic | ≥ 85% |
| E-04 Scoreboard | `handler/scoreboard.go`, `service/scoreboard.go` | Ranking, pagination | ≥ 70% |

---

## 5. API Endpoint Coverage

| Endpoint | Method | Test Cases | Covered |
|----------|:------:|-----------|:-------:|
| `/api/v1/subscribers` | POST | TC-007, TC-008, TC-009, TC-010, TC-011 | ✅ |
| `/api/v1/points/{handle}` | GET | TC-042, TC-043, TC-044, TC-045, TC-046 | ✅ |
| `/api/v1/scoreboard` | GET | TC-052, TC-053, TC-054, TC-055, TC-056 | ✅ |
| `/api/v1/webhooks/easydonate` | POST | TC-059 (HMAC) | ✅ |
| Rate Limiting (all endpoints) | — | TC-057, TC-058 | ✅ |

> **Result:** All 4 endpoints + 1 webhook + rate limiting covered.

---

## 6. Database Coverage

| Table | Constraints Tested | Test Cases |
|-------|-------------------|-----------|
| `subscribers` | UNIQUE(youtube_handle), CHECK(source), CHECK(handle length) | TC-003, TC-004, TC-007, TC-008, TC-009 |
| `donations` | UNIQUE(easydonate_id), CHECK(match_status), CHECK(amount > 0), FK → subscribers | TC-032, TC-033, TC-036, TC-037, TC-038, TC-039 |
| `viewer_points` | UNIQUE(youtube_handle), CHECK(total >= 0), CHECK(count >= 0), FK → subscribers | TC-042, TC-043, TC-045 |
| `oauth_tokens` | UNIQUE(provider, channel_id) | TC-017 (401 token expired) |

| Trigger | Tested Via | Test Cases |
|---------|-----------|-----------|
| `trg_*_updated` (updated_at) | Implicit — all update tests | All TCs that modify records |
| `trg_donations_sync_points` | Points calculation after match | TC-042, TC-045 (DEF-S003 flag for double-count) |

---

## 7. Business Objective Coverage

| Objective | KPI | Test Cases | Coverage |
|-----------|-----|-----------|:--------:|
| OBJ-01: Subscriber capture | 100% capture rate | TC-001 → TC-018 | ✅ |
| OBJ-02: Donate command | 100% response, <2s latency | TC-019 → TC-022, TC-027 | ✅ |
| OBJ-03: Point system | 100% accuracy, <5s sync | TC-023 → TC-046 | ✅ |
| OBJ-04: Scoreboard | <2s load, <60s freshness | TC-047 → TC-056 | ✅ |

---

## 8. Coverage Gaps & Risks

| Gap | Risk Level | Mitigation |
|-----|:----------:|-----------|
| Code coverage not yet measured | 🟡 | Populate after Dev implements |
| streamer.bot integration not automatable | 🟡 | Manual checklist per stream |
| EasyDonate API behavior not verified | 🟡 | Mock tests + manual verification |
| YouTube API quota monitoring | 🟢 | Mock tests cover error scenarios |
| DEF-S003 trigger double-count | 🟡 | Dev to add guard; retest after fix |
| DEF-S002 pagination limit enforcement | 🟢 | Dev to add validation; TC-055/056 verify |

---

## 9. Coverage Trend

> To be updated after each test cycle.

| Date | AC Coverage | Code Coverage | Tests Passed | Tests Failed |
|------|:-----------:|:------------:|:------------:|:------------:|
| 2026-07-30 | 100% (baseline) | — (no code) | — | — |
| TBD (Sprint 1) | — | — | — | — |
| TBD (Sprint 2) | — | — | — | — |
| TBD (Sprint 3) | — | — | — | — |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Coverage targets defined in plan |
| [[042_test_cases]] | Test cases that achieve coverage |
| [[043_defect_report]] | Gaps and defects tracked |
| [[013_acceptance_criteria]] | ACs that are covered |
| [[011_business_objective]] | Objectives verified by coverage |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119, ISO/IEC 25010
> **Usage:** Coverage is a *metric*, not a goal. 80% coverage with meaningful tests beats 95% with trivial assertions. Track what matters — business logic, edge cases, integration contracts.
