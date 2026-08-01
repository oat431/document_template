---
document_type: Coverage Report
version: "0.2"
status: Draft
author: "QA Engineer / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [coverage-report, requirements-coverage, code-coverage, swebok, vrm, deerngo-bot, members]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
  - ISO/IEC 25010 — Software Quality
---

# Coverage Report

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** This baseline now covers the explicit member-registration MVP. The former subscriber-polling tests remain historical and are not active regression scope.

---

## 1. Purpose

Track requirements coverage, projected test coverage, and code coverage readiness for the current Phase 1 member-based MVP.

## 2. Requirements Coverage

### 2.1 Active AC Summary

| Metric | Count | Percentage |
|--------|:-----:|:----------:|
| Total active acceptance criteria | 62 | 100% |
| Active ACs with test cases | 62 | 100% |
| Active ACs without test cases | 0 | 0% |
| 🔴 Must Have ACs covered | 43/43 | 100% |
| 🟡 Should Have ACs covered | 19/19 | 100% |

### 2.2 Coverage by Epic

| Epic | ACs | Covered | Coverage | Status |
|------|:---:|:-------:|:--------:|--------|
| E-01 Member Registration | 13 | 13 | 100% | 🟢 |
| E-02 Bot Commands | 17 | 17 | 100% | 🟢 |
| E-03 Points Engine | 20 | 20 | 100% | 🟢 |
| E-04 Scoreboard | 12 | 12 | 100% | 🟢 |
| **Total** | **62** | **62** | **100%** | 🟢 |

### 2.3 Coverage by User Story

| User Story | ACs | 🔴 | 🟡 | Test Cases | Coverage |
|------------|:---:|:---:|:---:|-----------|:--------:|
| US-001 Register Viewer as Member | 7 | 7 | 0 | TC-M001–M007 | 100% |
| US-002 Member Registration API | 6 | 5 | 1 | TC-M008–M013 | 100% |
| US-010 Donate Command | 4 | 2 | 2 | TC-M014–M017 | 100% |
| US-011 Point Command | 6 | 6 | 0 | TC-M018–M023 | 100% |
| US-012 Streamer.bot Actions | 7 | 6 | 1 | TC-M024–M030 | 100% |
| US-020 Donation Ingestion | 7 | 5 | 2 | TC-M031–M037 | 100% |
| US-021 Exact Donation Matching | 7 | 6 | 1 | TC-M038–M044 | 100% |
| US-022 Member Points API | 6 | 6 | 0 | TC-M045–M050 | 100% |
| US-030 Scoreboard Page | 6 | 0 | 6 | TC-M051–M056 | 100% |
| US-031 Scoreboard API | 6 | 0 | 6 | TC-M057–M062 | 100% |
| **Total** | **62** | **43** | **19** | **TC-M001–M062** | **100%** |

### 2.4 Superseded Coverage

| Former Scope | Historical Coverage | Active Status |
|--------------|--------------------|---------------|
| US-003 YouTube API polling | TC-012–TC-018 | Superseded; excluded from active regression |
| Subscriber table/viewer_points | Original QA baseline | Superseded by members schema |
| HMAC-specific tests | Original TC-059 | Replace with provider-contract/path-token tests until signing is confirmed |

## 3. Test Case Coverage

### 3.1 Test Case Summary

| Metric | Count |
|--------|:-----:|
| Active test cases | 62 |
| Mapped to active ACs | 62 |
| Additional active cross-cutting cases | 0 planned outside AC mapping |
| Automated target | 43 |
| Manual target | 19 |

### 3.2 Test Type Distribution (Target)

| Type | Count | Percentage | Automation |
|------|:-----:|:----------:|-----------:|
| Unit | 18 | 29% | 100% |
| Integration | 27 | 44% | 100% |
| System | 7 | 11% | 86% |
| Manual | 10 | 16% | 0% |
| **Total** | **62** | **100%** | **84% target** |

### 3.3 Priority Distribution

| Priority | Count | Percentage |
|----------|:-----:|:----------:|
| 🔴 Must Have | 43 | 69% |
| 🟡 Should Have | 19 | 31% |
| **Total** | **62** | **100%** |

## 4. Code Coverage

> **Status:** Not yet measured — code implementation is pending or not connected to this revised baseline.

### 4.1 Target Coverage

| Layer | Target | Actual | Status |
|-------|:------:|:------:|--------|
| Member/points service | ≥80% | — | ⬜ Pending |
| Handler/API layer | ≥60% | — | ⬜ Pending |
| Repository/transaction layer | ≥70% | — | ⬜ Pending |
| Overall | ≥70% | — | ⬜ Pending |

### 4.2 Measurement

```bash
go test -race -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
go tool cover -html=coverage.out
```

### 4.3 Projected Module Coverage

| Module | Key Focus | Expected |
|--------|-----------|:--------:|
| Member service | normalization, same-handle idempotency, handle change, conflict | ≥85% |
| Visibility service | active lookup, public/private update, no auto-create | ≥80% |
| Donation ingestion | payload validation, path token, reference idempotency, 429 | ≥80% |
| Matching/points | exact match, cutoff, inactive/unmatched, transaction guard | ≥90% |
| Scoreboard | filter, ranking, pagination, response allowlist | ≥80% |

## 5. API Endpoint Coverage

| Endpoint | Method | Active Test Cases | Covered |
|----------|:------:|-------------------|:-------:|
| `/api/v1/members/register` | POST | TC-M008–M013 | ✅ |
| `/api/v1/members/{youtube_user_id}/visibility` | PUT | TC-M025–M026 | ✅ |
| `/api/v1/members/{youtube_user_id}/points` | GET | TC-M045–M050 | ✅ |
| `/api/v1/scoreboard` | GET | TC-M057–M062 | ✅ |
| `/api/v1/webhooks/easydonate/{path_token}` | POST | TC-M031–M037 | ✅ |
| EasyDonate fallback sync | Internal | TC-M033–M036 | ✅ |

## 6. Database Coverage

| Table | Constraints/Rules | Active Test Cases |
|-------|-------------------|-------------------|
| `members` | active user uniqueness, active handle uniqueness, status, points, normalization | TC-M001–M013, TC-M038–M050 |
| `donations` | reference uniqueness, amount, status/source, member FK, points marker | TC-M031–M044 |
| `point_adjustment_notes` | member FK and correction metadata | Release/manual procedure test |

| Transaction/Rule | Tested Via |
|------------------|------------|
| Changed-handle transaction | TC-M003, TC-M010 |
| Exactly-once point application | TC-M032, TC-M044 |
| Donation cutoff | TC-M041 |
| Public projection allowlist | TC-M056, TC-M062 |

## 7. Business Objective Coverage

| Objective | KPI | Test Cases | Coverage |
|-----------|-----|-----------|:--------:|
| OBJ-01 Member registration | Registration success and identity correctness | TC-M001–M013 | ✅ |
| OBJ-02 Donate command | Response and latency | TC-M014–M017, TC-M027 | ✅ |
| OBJ-03 Member points | Exact, cutoff, once-only point accuracy | TC-M018–M050 | ✅ |
| OBJ-04 Privacy-aware scoreboard | Filter, freshness, public field allowlist | TC-M051–M062 | ✅ |

## 8. Coverage Gaps & Risks

| Gap | Risk | Mitigation |
|-----|:----:|------------|
| Code coverage not measured | 🟡 | Populate after Dev implementation |
| streamer.bot integration not automatable | 🟡 | Manual live-stream checklist |
| EasyDonate provider payload/security not fully verified | 🟠 | Block production webhook release until dashboard/test event confirms contract |
| Manual point correction not surfaced in admin UI | 🟡 | Transactional DB procedure + correction note; admin console Phase 2 |
| Public handle/score personal-data handling | 🟠 | Privacy notice, visibility command, allowlist, owner/legal review |

## 9. Coverage Trend

| Date | Active AC Coverage | Code Coverage | Tests Passed | Tests Failed |
|------|:------------------:|:------------:|:------------:|:------------:|
| 2026-08-02 | 100% requirements mapped | — | — | — |
| TBD Sprint 1 | — | — | — | — |
| TBD Sprint 2 | — | — | — | — |
| TBD Sprint 3 | — | — | — | — |

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Test strategy and targets |
| [[042_test_cases]] | Detailed cases; rewrite required for active baseline |
| [[043_defect_report]] | Historical defects and current gaps |
| [[013_acceptance_criteria]] | Active criteria covered |
| [[011_business_objective]] | Objectives verified |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119, ISO/IEC 25010
> **Usage:** This is a baseline coverage contract, not evidence that tests have passed. Actual execution status remains pending.
