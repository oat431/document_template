---
document_type: Risk Register
version: "0.3"
status: Active
author: "PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
pm_owner: "PO"
classification: "External"
tags: [risk-register, risk-tracking, pmbok, iso-31000, deerngo-bot, vrm, members, privacy]
standard_ref:
  - PMBOK v8 — Planning (Risk Management)
  - ISO 31000 — Risk Management
---

# Risk Register

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.3 | **Status:** Active
> **Last Updated:** 2026-08-02
>
> **Scope change:** YouTube subscriber polling and automatic subscriber collection were removed from the active Phase 1 MVP. Risks below reflect explicit member registration and member-based points.

---

## 1. Purpose

Living document for tracking all identified risks throughout the Deerngo Bot project lifecycle. Risks are reviewed at sprint retrospectives and phase gates.

---

## 2. Risk Register

| Risk ID | Category | Risk Description | Probability | Impact | Score | Level | Response Strategy | Response Actions | Owner | Trigger | Status | Due Date | Last Reviewed |
|---------|----------|-----------------|:-----------:|:------:|:-----:|:------:|------------------|-----------------|:-----:|---------|:------:|:--------:|:--------------:|
| R-001 | Technical | EasyDonate API rate limit or provider contract changes | 3 — Possible | 3 — Moderate | 9 | 🟡 Medium | Mitigate | Use API polling only as fallback; respect provider limits; confirm current OpenAPI contract; back off on 429. | Dev | 429 or payload mismatch | ⬜ Open | Ongoing | 2026-08-02 |
| R-002 | Integration | EasyDonate webhook has no confirmed signing/authentication mechanism | 3 — Possible | 4 — Major | 12 | 🟠 High | Mitigate | Use provider-confirmed auth if available; otherwise unpredictable path token, strict validation, body limit, rate limit, and reference idempotency. | Dev/DevOps | Provider dashboard exposes no signing option | ⬜ Open | Before webhook deployment | 2026-08-02 |
| R-003 | Technical | Donor name does not match registered member handle | 4 — Likely | 3 — Moderate | 12 | 🟠 High | Mitigate | Channel owner publishes exact naming rule; normalize harmless formatting only; retain unmatched records privately without awarding points. | Owner/Dev | High unmatched-donation rate | ⬜ Open | Ongoing | 2026-08-02 |
| R-004 | Data Integrity | Same normalized handle is claimed by different active YouTube user IDs | 2 — Unlikely | 4 — Major | 8 | 🟡 Medium | Mitigate | Partial unique index for active handles; reject conflicting registration; owner resolves manually. | Dev | Registration returns HANDLE_IN_USE | ⬜ Open | Sprint 1 | 2026-08-02 |
| R-005 | Data Integrity | Re-registration creates separate old/new records and points require manual transfer | 3 — Possible | 4 — Major | 12 | 🟠 High | Mitigate | Old record becomes inactive; points remain; new record starts at 0; use transaction and point-adjustment note for manual correction. | Dev/Owner | Handle-change registration | ⬜ Open | Sprint 1 | 2026-08-02 |
| R-006 | Privacy / PDPA | Public handle and score may identify a natural person or reveal donation participation | 3 — Possible | 4 — Major | 12 | 🟠 High | Mitigate | Apply the owner checklist: privacy notice, purpose/lawful-basis decision, optional public-display decision, retention/deletion, rights/removal route, controller/processor instructions, breach process, and Thai legal review where needed. | Owner/PO/Dev | Member asks to hide/remove data or owner checklist incomplete | ⬜ Open | Before scoreboard release | 2026-08-02 |
| R-007 | Availability | streamer.bot offline prevents registration and chat responses | 3 — Possible | 3 — Moderate | 9 | 🟡 Medium | Accept/Mitigate | No false success; viewer retries next live stream; backend errors return friendly retry message when streamer.bot is online. | Dev/Owner | No chat events received | ⬜ Open | Ongoing | 2026-08-02 |
| R-008 | Data Integrity | Manual database point correction introduces an incorrect total | 3 — Possible | 4 — Major | 12 | 🟠 High | Mitigate | Transactional correction procedure; backup; record before/after/reason in `point_adjustment_notes`; admin console later. | Owner/DevOps | Point mismatch or transfer request | ⬜ Open | Ongoing | 2026-08-02 |
| R-009 | Infrastructure | Cloudflare Tunnel downtime affects scoreboard/webhook reachability | 2 — Unlikely | 2 — Minor | 4 | 🟢 Low | Accept | Monitor tunnel; API fallback reconciles missed donations; accept short read-only outage. | DevOps | Tunnel offline >5 min | ⬜ Open | Ongoing | 2026-08-02 |
| R-010 | Security | Webhook path token is leaked or abused | 2 — Unlikely | 4 — Major | 8 | 🟡 Medium | Mitigate | Store token as deployment secret; never publish it; rotate path on suspected leak; validate payload and idempotency. | DevOps | Unexpected webhook traffic | ⬜ Open | Ongoing | 2026-08-02 |

---

## 3. Risk Heat Map

| Impact \ Probability | 1 — Rare | 2 — Unlikely | 3 — Possible | 4 — Likely | 5 — Almost Certain |
|---------------------|:--------:|:------------:|:------------:|:----------:|:------------------:|
| **5 — Critical** | 🟢 | 🟡 | 🟠 | 🔴 | 🔴 |
| **4 — Major** | 🟢 | 🟡 R-004, R-010 | 🟠 R-002, R-005, R-006, R-008 | 🟠 | 🔴 |
| **3 — Moderate** | 🟢 | 🟡 R-007 | 🟡 R-001 | 🟡 R-003 | 🟠 |
| **2 — Minor** | 🟢 | 🟢 R-009 | 🟡 | 🟡 | 🟡 |
| **1 — Insignificant** | 🟢 | 🟢 | 🟢 | 🟢 | 🟡 |

> **Legend:** 🔴 Critical — Immediate action required | 🟠 High — Mitigation plan required | 🟡 Medium — Monitor and manage | 🟢 Low — Accept and monitor

---

## 4. Risk Summary

| Level | Count | Risks |
|-------|:-----:|-------|
| 🔴 Critical | 0 | — |
| 🟠 High | 5 | R-002, R-003, R-005, R-006, R-008 |
| 🟡 Medium | 4 | R-001, R-004, R-007, R-010 |
| 🟢 Low | 1 | R-009 |
| **Total** | **10** | |

---

## 5. Risk Response Status

| Risk ID | Response | Planned Actions | Completed Actions | Status |
|---------|----------|----------------|------------------|:------:|
| R-001 | Mitigate | Verify provider contract, fallback polling/backoff | — | ⬜ Not Started |
| R-002 | Mitigate | Confirm webhook security; implement path-token fallback | — | ⬜ Not Started |
| R-003 | Mitigate | Publish naming rule; exact normalization; private unmatched data | — | ⬜ Not Started |
| R-004 | Mitigate | Active-handle unique constraint and conflict response | — | ⬜ Not Started |
| R-005 | Mitigate | Re-registration transaction and manual correction note | — | ⬜ Not Started |
| R-006 | Mitigate | Notice, visibility commands, private/public API filtering | — | ⬜ Not Started |
| R-007 | Accept/Mitigate | Retry messaging and live-stream operational guidance | — | ⬜ Not Started |
| R-008 | Mitigate | Backup, transaction, adjustment-note procedure | — | ⬜ Not Started |
| R-009 | Accept | Tunnel monitoring and donation API reconciliation | — | ⬜ Not Started |
| R-010 | Mitigate | Secret storage and rotation runbook | — | ⬜ Not Started |

---

## 6. Retired / Closed Risks

| Risk ID | Description | Close Date | Reason | Outcome |
|---------|-------------|-----------|--------|---------|
| R-OLD-001 | YouTube subscriber API quota/exposure completeness | 2026-08-02 | Subscriber polling removed from active MVP | Retained as MM06 rationale; no longer a Phase 1 runtime risk |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[phase1-deerngo-bot-mvp]] | Phase plan risks |
| [[061_security_test_report]] | Security and privacy findings |
| [[043_defect_report]] | Original pre-code spec gaps |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Scope-change decision |

---

> **Template Standard:** Based on PMBOK v8 and ISO 31000
> **Usage:** Living document — update at every sprint retrospective. Every 🟠/🔴 risk must have a mitigation plan, owner, and trigger.
