---
document_type: Security Test Report
version: "0.3"
status: Draft
author: "QA Engineer / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [security-testing, vulnerability, owasp, swebok, vrm, members, privacy, easydonate]
standard_ref:
  - SWEBOK v4 — Testing
  - OWASP Testing Guide v4
  - OWASP Top 10 (2021)
  - ISO/IEC 25010 — Security
---

# Security Test Report

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.3 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** This pre-code security assessment covers explicit member registration, private donation ingestion, exact point attribution, visibility, and the privacy-safe public scoreboard. The former subscriber/OAuth/poller path is not active MVP scope.

---

## 1. Purpose

Assess the proposed Phase 1 attack surface and define security verification before code exists. Findings are design-level controls, not execution evidence.

## 2. Security Test Summary

| Field | Detail |
|-------|--------|
| Test Date | 2026-08-02 (spec-based assessment) |
| Test Type | Spec review + threat modeling (pre-code) |
| Tools | Manual document review; code scan pending implementation |
| Tester | QA Engineer |
| Scope | Go member API, EasyDonate webhook/API, database, public scoreboard, streamer.bot integration |
| Overall Risk | 🟠 High until provider webhook contract, privacy notice, and point-correction procedure are verified |

---

## 3. Threat Model

### 3.1 Attack Surface

| Surface | Exposure | Entry Point | Risk |
|---------|----------|-------------|:----:|
| EasyDonate webhook | Public via Cloudflare route | `POST /api/v1/webhooks/easydonate/{path_token}` | 🟠 High |
| Scoreboard page | Public | Next.js | 🟡 Medium |
| Scoreboard API | Public read-only | `GET /api/v1/scoreboard` | 🟡 Medium |
| Registration/visibility/points API | LAN from streamer.bot | Member routes | 🟡 Medium |
| PostgreSQL | Docker network only | `members`, `donations` | 🟢 Low |
| Direct DB correction | Operator-only | SQL transaction | 🟠 High |

### 3.2 Trust Boundaries

```text
Internet → Cloudflare Tunnel → Next.js / intentionally configured webhook route → Go Backend → PostgreSQL
                                      ↑                           ↑
LAN → streamer.bot (Windows PC) ──────┘                           │
EasyDonate provider → webhook/API ────────────────────────────────┘
```

| Boundary | Trust Level | Required Control |
|----------|-------------|------------------|
| Internet → Cloudflare | Untrusted | TLS, DDoS/WAF, route minimization |
| Provider → webhook | Untrusted/semi-trusted | Provider-confirmed auth or path token, strict validation, idempotency |
| streamer.bot → member API | LAN trusted but not authenticated | Actual identity fields, input validation, rate limit; add API key/mTLS later |
| Go → PostgreSQL | Application trust boundary | Parameterized queries and transactions |
| Public API → browser | Public | Allowlisted fields and filters |
| Operator → manual corrections | Privileged | Backup, transaction, least privilege, before/after/reason audit note |

---

## 4. OWASP Top 10 Assessment

| # | Category | Status | Notes |
|---|----------|:------:|-------|
| A01 | Broken Access Control | 🟡 Partial | Internal routes have no Phase 1 auth; LAN restriction and actual streamer.bot identity are controls. Public API is allowlisted. |
| A02 | Cryptographic Failures | 🟡 Partial | TLS via Cloudflare; provider signature is unverified. Use secret path only as documented fallback. |
| A03 | Injection | ✅ Design Pass | Use parameterized sqlx queries; validate user ID, handle, path token, JSON, page/limit. |
| A04 | Insecure Design | 🟡 Partial | Add body limits, rate limiting, exactly-once point transaction, cutoff, conflict constraints. |
| A05 | Security Misconfiguration | 🟡 Partial | Explicit CORS, security headers, debug off, secrets outside Git, public route minimization required. |
| A06 | Vulnerable Components | ⬜ Pending | Run `govulncheck`, npm audit, and container scans after implementation. |
| A07 | Auth Failures | 🟡 Partial | No user login by design; provider webhook auth and future internal API auth remain controls to verify. |
| A08 | Data Integrity Failures | 🟡 Partial | Unique `reference_no`, transaction lock, timestamp cutoff, and active uniqueness are required. |
| A09 | Logging Failures | 🟡 Partial | Redact donor messages, API keys, path tokens, and provider payloads; retain safe action audit. |
| A10 | SSRF | ✅ Design Pass | Outbound clients use fixed provider URLs/configuration; no arbitrary user URL fetch. |

### Summary

| Status | Count |
|--------|:-----:|
| ✅ Design Pass | 2 |
| 🟡 Partial | 7 |
| ⬜ Pending | 1 |
| 🔴 Fail | 0 |

---

## 5. Security Findings

### SEC-001: Internal APIs lack authentication

| Field | Detail |
|-------|--------|
| Severity | 🟡 Medium |
| Component | Member registration, visibility, points routes |
| Risk | A compromised LAN device could inject/update/query member data |
| Phase 1 Response | Accept temporarily; restrict network reachability, validate actual identity payload, rate-limit |
| Phase 2 | Add service API key or mTLS |
| Status | ⬜ Open / accepted for MVP |

### SEC-002: EasyDonate webhook authenticity is unverified

| Field | Detail |
|-------|--------|
| Severity | 🟠 High |
| Component | EasyDonate webhook route |
| Risk | Forged events could create false donation records/points |
| Phase 1 Response | Verify dashboard/provider contract. If no signing/custom auth exists, use a long random path token, strict payload validation, body-size limit, rate limit, and `referenceNo` idempotency. |
| Status | ⬜ Blocker before production webhook release |

### SEC-003: Webhook replay/duplicate risk

| Field | Detail |
|-------|--------|
| Severity | 🟡 Medium |
| Component | Donation ingestion and point application |
| Risk | Repeated provider events could double-count points |
| Control | Unique `reference_no`, row lock, `points_applied_at IS NULL` guard, transactional update |
| Status | ⬜ Test required |

### SEC-004: Manual point correction risk

| Field | Detail |
|-------|--------|
| Severity | 🟠 High |
| Component | Direct owner/back-office database correction |
| Risk | Human error can assign wrong balance or corrupt history |
| Control | Backup, transaction, least privilege, before/after/reason note, verify active/inactive rows; admin console later |
| Status | ⬜ Procedure issue #19 |

### SEC-005: Public handle/score personal-data exposure

| Field | Detail |
|-------|--------|
| Severity | 🟠 High |
| Component | Scoreboard API/page and public chat replies |
| Risk | Handle and score may identify a person or reveal contribution |
| Control | Do not store display names; public only for active/public/points>0; `:deer: private`; 100-point band for private chat replies; privacy notice and removal path; legal review by owner |
| Status | ⬜ Gate before scoreboard release |

### SEC-006: No request-size limit / permissive CORS risk

| Field | Detail |
|-------|--------|
| Severity | 🟡 Medium |
| Component | Webhook/API middleware |
| Risk | Resource exhaustion or unintended cross-origin access |
| Control | Small webhook body limit, rate limiting, explicit scoreboard origin CORS, no CORS for internal routes, security headers |
| Status | ⬜ Open |

### SEC-007: Secrets/logs/data minimization

| Field | Detail |
|-------|--------|
| Severity | 🟠 High |
| Component | Configuration, logs, public DTOs, fixtures |
| Risk | API key/path token/raw donor data leaks |
| Control | Environment/deployment secrets, redaction, synthetic test data, public response allowlist |
| Status | ⬜ Open |

---

## 6. Security Test Plan

| Test | Target | Method | Priority |
|------|--------|--------|:--------:|
| Invalid webhook path | Webhook | Wrong/missing path token | 🔴 |
| Invalid provider payload | Webhook | Missing reference, negative amount, invalid timestamp, oversized body | 🔴 |
| Webhook duplicate/replay | Donation/points | Same reference concurrently and sequentially | 🔴 |
| SQL injection | Member/donation/query inputs | Fuzz IDs, handles, donor names, page/limit | 🔴 |
| Point race | Points transaction | Concurrent matcher invocations | 🔴 |
| Active uniqueness | Registration | Same user/handle concurrent registrations | 🔴 |
| Visibility enforcement | Points/scoreboard | Toggle private/inactive and inspect responses | 🔴 |
| Public response allowlist | Scoreboard | Inspect all JSON keys/values | 🔴 |
| Rate limiting | All routes | Exceed configured thresholds | 🟡 |
| CORS | Public/internal routes | Authorized and unauthorized origins | 🟡 |
| Error leakage | All routes | Trigger 4xx/5xx and inspect body/logs | 🟡 |
| Secret redaction | Logs/config | Search output for key/token/raw messages | 🔴 |
| Dependencies | Go/Node/container | `govulncheck`, npm audit, image scanner | 🔴 |

## 7. Security Recommendations

| # | Recommendation | Priority | Owner | Status |
|---|---------------|:--------:|-------|:------:|
| 1 | Confirm EasyDonate webhook auth/payload from dashboard/test event | 🔴 | Dev/DevOps | ⬜ |
| 2 | Implement path-token fallback only if provider signing is unavailable | 🔴 | Dev | ⬜ |
| 3 | Add body-size limit and rate limit | 🔴 | Dev | ⬜ |
| 4 | Add exactly-once donation point transaction | 🔴 | Dev | ⬜ |
| 5 | Use explicit public response allowlist | 🔴 | Dev/QA | ⬜ |
| 6 | Document manual correction procedure | 🔴 | PO/DevOps | ⬜ |
| 7 | Add explicit CORS and security headers | 🟡 | Dev | ⬜ |
| 8 | Keep secrets out of Git/logs/issues | 🔴 | All | ⬜ |
| 9 | Run dependency/container security scans | 🔴 | Dev/DevOps | ⬜ |
| 10 | Add internal API authentication in Phase 2 | 🟢 | Dev | Deferred |

## 8. Thai PDPA Owner Readiness Gate

A public YouTube handle, donor name, donation event, and contribution score may be personal data under Thai PDPA depending on context. Public visibility is a separate disclosure purpose; public availability is not a blanket exemption. This document is not legal advice or a compliance certification.

Before public release, the channel owner must review `06_security/063_thai_pdpa_owner_checklist.md` and confirm:

- [ ] Controller/contact and developer/host processing roles are documented.
- [ ] Separate purposes and lawful-basis decisions are recorded for registration, matching, points, publication, logs, and backups.
- [ ] Privacy notice is approved and accessible.
- [ ] Public-display choice, `:deer: private`, correction, access, removal/erasure, and external contact routes are usable.
- [ ] Retention/deletion rules cover members, donor data, logs, caches, exports, and backups.
- [ ] Developer/host instructions, no-reuse rule, security, subprocessors, incident escalation, and deletion obligations are recorded.
- [ ] Breach escalation and Thai counsel review (where needed) are complete.

The scoreboard release gate remains blocked until the owner completes this review.

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[022_API_specification]] | Current API/security contract |
| [[021_architecture_decision_records]] | ADR-005, ADR-006, ADR-007, ADR-012, ADR-016 |
| [[023_database_schema_DDL]] | Transaction/data constraints |
| [[041_test_plan]] | Active testing plan |
| [[071_risk_register]] | Runtime risk register |
| [[062_coding_standards_security]] | Secure implementation rules |
| [[063_thai_pdpa_owner_checklist]] | Owner privacy-readiness checklist |
| `https://github.com/oat431/deerngo-bot/issues/18` | EasyDonate contract gate |
| `https://github.com/oat431/deerngo-bot/issues/19` | Manual point-correction procedure |

---

> **Template Standard:** Based on SWEBOK v4 and OWASP Testing Guide v4
> **Usage:** Pre-code security assessment. Do not report controls as passed until executed and evidenced.
---

