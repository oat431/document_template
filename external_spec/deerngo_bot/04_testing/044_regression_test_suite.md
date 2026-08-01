---
document_type: Regression Test Suite
version: "0.2"
status: Draft
author: "QA Engineer / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [regression-test-suite, smoke-test, ci-cd, swebok, members, privacy, easydonate]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Regression Test Suite

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** Active regression is now based on explicit members and TC-M001–TC-M062. Original subscriber/polling cases are historical only.

---

## 1. Purpose

Define smoke, core, full, and manual release suites for the member-based Phase 1 MVP. Every active acceptance criterion is mapped to one active test case in `042_test_cases.md`.

## 2. Suite Structure

| Suite | Purpose | Trigger | Target |
|-------|---------|---------|:------:|
| **Smoke** | Critical member/points/scoreboard sanity | PR/review or deployment | 12 cases |
| **Core Regression** | All 43 🔴 Must Have criteria | Sprint release | 43 cases |
| **Full Regression** | All 62 active criteria | Phase release | 62 cases |
| **Manual Checklist** | streamer.bot, provider, UI, live behavior | Stream/release | Selected manual cases |

> Counts refer to test cases, not unique runtime tests. Provider-contract and security gates may block release even if mapped functional tests pass.

## 3. Smoke Suite

### 3.1 Selection

| # | Test Case | Why |
|---|-----------|-----|
| 1 | TC-M001 | New member creation/zero start |
| 2 | TC-M002 | Same-handle idempotency |
| 3 | TC-M003 | Changed-handle old/new state |
| 4 | TC-M004 | Active-handle conflict |
| 5 | TC-M013 | Normalization |
| 6 | TC-M031 | Valid webhook ingestion |
| 7 | TC-M032 | Duplicate provider reference |
| 8 | TC-M038 | Exact normalized match |
| 9 | TC-M041 | Pre-registration cutoff |
| 10 | TC-M044 | Exactly-once points |
| 11 | TC-M057 | Scoreboard public projection |
| 12 | TC-M062 | Public data allowlist |

### 3.2 Execution

```bash
docker compose -f docker-compose.test.yml up -d
go test -tags=smoke -race -v -timeout=5m ./...
docker compose -f docker-compose.test.yml down
```

Any smoke failure blocks the deployment gate.

## 4. Core Regression — Must Have

The core suite contains the 43 🔴 active cases. QA should generate the exact command/tag list from the traceability table rather than maintaining a second hand-edited ID list.

**Pass rule:** all 43 🔴 criteria/test cases verified; no open critical defect; provider/security gates resolved or explicitly blocked from release.

## 5. Full Regression — Phase Release

| Epic | Test Case Range | Count |
|------|-----------------|:-----:|
| E-01 Member Registration | TC-M001–TC-M013 | 13 |
| E-02 Bot Commands | TC-M014–TC-M030 | 17 |
| E-03 Points Engine | TC-M031–TC-M050 | 20 |
| E-04 Scoreboard | TC-M051–TC-M062 | 12 |
| **Total** | **TC-M001–TC-M062** | **62** |

### Execution Order

| Order | Module | Reason |
|:-----:|--------|--------|
| 1 | E-01 Member Registration | Foundation and identity state |
| 2 | E-03 Points Engine | Depends on active member data |
| 3 | E-02 Bot Commands | Depends on member/points APIs |
| 4 | E-04 Scoreboard | Depends on points and visibility |
| 5 | Provider/security gates | Prevent production data-integrity/privacy failure |

**Go-live rule:** all active criteria pass or are formally accepted by PO; no 🔴 defect remains open; EasyDonate contract and privacy gates are satisfied.

## 6. Manual Checklist

### 6.1 Streamer.bot

- [ ] TC-M005 — typed handle ignored; actual user identity used.
- [ ] TC-M006 — registration notice wording.
- [ ] TC-M007 — backend unavailable fallback.
- [ ] TC-M014/M015 — donate command online/offline behavior.
- [ ] TC-M020/M021 — private exact band/zero band.
- [ ] TC-M024–M030 — registration/visibility/point/donate actions and safe logs.

### 6.2 Provider/Deployment

- [ ] TC-M031 — verified provider payload reaches webhook.
- [ ] TC-M037 — invalid path/payload rejected.
- [ ] TECH-001 / Issue #18 — provider auth/signing contract confirmed.
- [ ] Tunnel exposes only approved scoreboard/webhook routes.
- [ ] Secrets/path tokens are absent from logs.

### 6.3 UI

- [ ] TC-M051 — ranked handle/points page.
- [ ] TC-M054 — empty state.
- [ ] TC-M056 — error state.
- [ ] TC-M060/M061 — pagination.
- [ ] TC-M062 — API allowlist and no display-name/donor fields.
- [ ] Keyboard/mobile/accessibility checks.

## 7. Test Data Reset

```sql
TRUNCATE point_adjustment_notes, donations, members CASCADE;
```

Use synthetic IDs, handles, amounts, and messages. Never use real client donor details in CI or fixtures.

## 8. CI/CD Integration

```yaml
stages:
  - name: unit
    command: go test -race -cover ./...
  - name: integration
    command: go test -tags=integration -timeout=10m ./...
  - name: smoke
    command: go test -tags=smoke -timeout=5m ./...
  - name: full-active-regression
    command: go test -tags=regression-full -timeout=60m ./...
```

Provider test events and live streamer.bot tests remain manual and are not simulated with production credentials in CI.

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Test strategy |
| [[042_test_cases]] | Detailed active cases |
| [[043_defect_report]] | Active blockers/gaps |
| [[045_coverage_report]] | Coverage summary |
| [[061_security_test_report]] | Security release gates |
| `https://github.com/oat431/deerngo-bot/issues/18` | Provider contract gate |
| `https://github.com/oat431/deerngo-bot/issues/19` | Manual correction procedure |

---

> **Template Standard:** Based on SWEBOK v4 and ISO/IEC/IEEE 29119
> **Usage:** Release gate for the revised member-based MVP. Historical subscriber tests do not satisfy active coverage.
---
