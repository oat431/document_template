---
document_type: Risk Register
version: "0.1"
status: Active
author: "PO"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
pm_owner: "PO"
classification: "External"
tags: [risk-register, risk-tracking, pmbok, iso-31000, deerngo-bot, vrm]
standard_ref:
  - PMBOK v8 — Planning (Risk Management)
  - ISO 31000 — Risk Management
---

# Risk Register

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Active
> **Last Updated:** 2026-07-30

---

## 1. Purpose

> Living document for tracking all identified risks throughout the Deerngo Bot project lifecycle. Risks are reviewed at sprint retrospectives and updated as the project evolves.

---

## 2. Risk Register

| Risk ID | Category | Risk Description | Probability | Impact | Score | Level | Response Strategy | Response Actions | Owner | Trigger | Status | Due Date | Last Reviewed |
|---------|----------|-----------------|:-----------:|:------:|:-----:|:------:|------------------|-----------------|:-----:|---------|:------:|:--------:|:--------------:|
| R-001 | Technical | YouTube API quota exhaustion (>10K units/day) | 2 — Unlikely | 4 — Major | 8 | 🟡 Medium | Mitigate | Poll every 15 min (96 calls/day, well within limit). Monitor quota usage via Google Cloud Console. | Dev | Quota >80% used | ⬜ Open | Ongoing | 2026-07-30 |
| R-002 | Technical | EasyDonate webhook unreliable or unavailable | 3 — Possible | 3 — Moderate | 9 | 🟡 Medium | Mitigate | Fallback to polling scheduler (every 5 min). Cache last known donations. Alert on consecutive failures. | Dev | 3 consecutive webhook failures | ⬜ Open | Ongoing | 2026-07-30 |
| R-003 | Technical | Fuzzy name matching false positives | 3 — Possible | 4 — Major | 12 | 🟠 High | Mitigate | Log all matches with confidence score. Manual review queue for low-confidence matches. Configurable match threshold. | Dev | >5% false positive rate | ⬜ Open | Ongoing | 2026-07-30 |
| R-004 | Technical | streamer.bot LAN connection unstable | 2 — Unlikely | 3 — Moderate | 6 | 🟡 Medium | Mitigate | Retry logic with exponential backoff. Health check endpoint. Log connection failures. | Dev | >10 connection failures/hour | ⬜ Open | Ongoing | 2026-07-30 |
| R-005 | Infrastructure | Cloudflare Tunnel downtime | 2 — Unlikely | 2 — Minor | 4 | 🟢 Low | Accept | Uptime Kuma monitoring. Scoreboard is non-critical (read-only). Accept brief downtime. | DevOps | Tunnel offline >5 min | ⬜ Open | Ongoing | 2026-07-30 |
| R-006 | Technical | `sync_viewer_points()` trigger double-count risk (DEF-S003) | 3 — Possible | 4 — Major | 12 | 🟠 High | Mitigate | Add guard clause: only add points when `OLD.match_status != 'matched' AND NEW.match_status = 'matched'`. Test with edge cases. | Dev | Points mismatch >1% | ⬜ Open | Sprint 2 | 2026-07-30 |
| R-007 | | | | | | | | | | | | | |
| R-008 | | | | | | | | | | | | | |
| R-009 | | | | | | | | | | | | | |
| R-010 | | | | | | | | | | | | | |

> **Empty rows (R-007 → R-010)** reserved for future risks. Add as identified.

---

## 3. Risk Heat Map

| Impact \ Probability | 1 — Rare | 2 — Unlikely | 3 — Possible | 4 — Likely | 5 — Almost Certain |
|---------------------|:--------:|:------------:|:------------:|:----------:|:------------------:|
| **5 — Critical** | 🟢 | 🟡 | 🟠 | 🔴 | 🔴 |
| **4 — Major** | 🟢 | 🟡 R-001 | 🟠 R-003, R-006 | 🟠 | 🔴 |
| **3 — Moderate** | 🟢 | 🟡 R-004 | 🟡 R-002 | 🟡 | 🟠 |
| **2 — Minor** | 🟢 | 🟢 R-005 | 🟡 | 🟡 | 🟡 |
| **1 — Insignificant** | 🟢 | 🟢 | 🟢 | 🟢 | 🟡 |

> **Legend:** 🔴 Critical — Immediate action required | 🟠 High — Mitigation plan required | 🟡 Medium — Monitor and manage | 🟢 Low — Accept and monitor

---

## 4. Risk Summary

| Level | Count | Risks |
|-------|:-----:|-------|
| 🔴 Critical | 0 | — |
| 🟠 High | 2 | R-003, R-006 |
| 🟡 Medium | 3 | R-001, R-002, R-004 |
| 🟢 Low | 1 | R-005 |
| **Total** | **6** | |

---

## 5. Risk Response Status

| Risk ID | Response | Planned Actions | Completed Actions | Status |
|---------|----------|----------------|------------------|:------:|
| R-001 | Mitigate | Poll every 15 min, monitor quota | — | ⬜ Not Started |
| R-002 | Mitigate | Fallback polling, caching, alerting | — | ⬜ Not Started |
| R-003 | Mitigate | Logging, manual review, configurable threshold | — | ⬜ Not Started |
| R-004 | Mitigate | Retry logic, health check, logging | — | ⬜ Not Started |
| R-005 | Accept | Uptime monitoring | — | ⬜ Not Started |
| R-006 | Mitigate | Guard clause in trigger, edge case tests | — | ⬜ Not Started |

---

## 6. Retired / Closed Risks

| Risk ID | Description | Close Date | Reason | Outcome |
|---------|-------------|-----------|--------|---------|
| — | — | — | — | — |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[phase1-deerngo-bot-mvp]] | Risks identified in Phase 1 plan |
| [[061_security_test_report]] | Security risks (SEC-001 → SEC-005) |
| [[043_defect_report]] | Spec gaps (DEF-S001 → DEF-S006) |

---

> **Template Standard:** Based on PMBOK v8, ISO 31000
> **Usage:** Living document — update at every sprint retrospective. Every 🟠/🔴 risk must have a mitigation plan, owner, and trigger.
