---
document_type: Meeting Minutes
version: "1.0"
status: Final
author: "PO Persona"
created: "2026-08-02"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
meeting_type: "PO Decision and Dev/QA Handoff — Member Registration MVP"
participants: ["PO Persona", "Dev Persona", "QA Persona"]
classification: "Internal"
tags: [meeting-minutes, decision, member-registration, points, easydonate, privacy, vrm]
standard_ref:
  - SWEBOK v4 — Requirements
  - ISO/IEC/IEEE 29148 — Requirements Engineering
  - ISO/IEC/IEEE 12207 — Software Life Cycle Processes
---

# Meeting Minutes — PO → Dev/QA: Member Registration MVP

> **Date:** 2026-08-02
> **Type:** Product decision and implementation handoff
> **From:** PO Persona
> **To:** Dev Persona, QA Persona
> **Status:** ✅ Final — requirements/design baseline updated

---

## 1. Purpose

Record the approved change from incomplete YouTube subscriber capture to explicit viewer membership, and hand off the revised requirements, API, database, architecture, test, risk, and phase-plan baseline to Dev and QA.

## 2. Trigger

MM06 documented a live YouTube API result of 172 visible subscribers out of a channel total of 1,310. The API cannot provide a complete historical subscriber list, so automatic subscriber capture is not a reliable membership source for the VRM MVP.

## 3. Decisions Made

| Decision ID | Decision | Rationale | Authority | Impact |
|-------------|----------|-----------|----------|--------|
| DEC-MM07-001 | Replace automatic subscriber capture with explicit `:deer: register` | Viewer intentionally joins the points program; avoids incomplete external list | PO | US-003/poller removed from active MVP |
| DEC-MM07-002 | Use actual streamer.bot chat identity | Prevent arbitrary handle self-claiming | PO | Register API accepts actual user ID + current handle; typed handle ignored |
| DEC-MM07-003 | New member starts with 0 points and public visibility enabled | Clear cutover and simple MVP | PO | New `members` table and registration criteria |
| DEC-MM07-004 | Same-handle repeat is idempotent; changed handle creates inactive old + active new zero-point row | Avoid alias/admin-console complexity; manual correction later | PO | Old points preserved; no automatic transfer |
| DEC-MM07-005 | Match only normalized exact active handles | Prevent fuzzy false-positive point attribution | PO | Trim, remove leading `@`, lowercase; `pg_trgm` removed from active path |
| DEC-MM07-006 | Only donations at/after `registered_at` can earn points | Prevent retroactive attribution | PO | Donation-time cutoff in matching transaction |
| DEC-MM07-007 | Private members continue earning but receive 100-point bands in public chat | Reduce exact score exposure while retaining engagement | PO | Visibility API and banded point response |
| DEC-MM07-008 | Scoreboard returns active/public/points>0 handle + points only | Minimize public personal data | PO | No display name, user ID, donor name, or message |
| DEC-MM07-009 | EasyDonate webhook primary + API fallback, without assumed HMAC | Public documentation does not confirm HMAC/signature header | PO | Verify dashboard/test payload before production; path-token fallback |
| DEC-MM07-010 | No admin console in Phase 1 | Reduce MVP complexity | PO | Owner/back office may correct points via controlled DB procedure; console Phase 2 |

## 4. Current Active Scope

| Area | Active Behavior |
|------|-----------------|
| Member registration | `POST /api/v1/members/register` from streamer.bot |
| Visibility | `PUT /api/v1/members/{youtube_user_id}/visibility` |
| Point query | `GET /api/v1/members/{youtube_user_id}/points` |
| Donation link | `:deer: donate` → `https://easydonate.app/deerngo0` |
| Donation ingestion | EasyDonate webhook primary; API polling fallback |
| Matching | Normalized exact donor name to active handle |
| Points | 1 THB = 1 point; transactionally applied once |
| Scoreboard | Active + public + total_points > 0 |
| Removed | YouTube polling, OAuth, subscriber table as eligibility source, fuzzy matching, admin console |

## 5. Documents Updated

| Document | Version | Change |
|----------|:------:|--------|
| `01_requirement/011_business_objective.md` | 0.2 | Explicit membership, revised KPIs/dependencies/risks |
| `01_requirement/012_user_stories.md` | 0.2 | 10 active stories; US-003 superseded |
| `01_requirement/013_acceptance_criteria.md` | 0.2 | 62 active ACs: 43 Must, 19 Should |
| `02_design/021_architecture_decision_records.md` | 0.2 | Member/exact/privacy decisions; HMAC assumption superseded |
| `02_design/022_API_specification.md` | 0.2 | Member/visibility/points routes and provider caveats |
| `02_design/023_database_schema_DDL.md` | 0.2 | `members`, revised `donations`, correction notes |
| `02_design/024_ERD.md` | 0.2 | Member-based logical model |
| `02_design/025_software_architecture_document.md` | 0.2 | Revised components and data flows |
| `02_design/029_architecture_overview.md` | 0.2 | Revised architecture map and data boundaries |
| `03_construction/031_BE_README.md` | 0.2 | Revised backend setup/configuration/API |
| `04_testing/041_test_plan.md` | 0.2 | Active member-based test strategy |
| `04_testing/042_test_cases.md` | 0.2 | TC-M001–TC-M062 active test set |
| `04_testing/045_coverage_report.md` | 0.2 | Active coverage baseline |
| `07_pm/071_risk_register.md` | 0.2 | Member, privacy, provider, and manual-correction risks |
| `external_overview/Deer Ngo Bot.md` | 0.2 | Revised product overview |
| `external_plan/phase1-deerngo-bot-mvp.md` | 0.2 | Revised 3-sprint implementation plan |

## 6. Handoff to Dev

- Implement the `members` schema before feature code.
- Use a transaction for changed-handle registration.
- Enforce active user/handle uniqueness in PostgreSQL.
- Do not store YouTube display names.
- Do not implement YouTube polling/OAuth for the active MVP.
- Do not implement fuzzy matching.
- Verify EasyDonate's actual payload/security settings before production webhook configuration.
- Do not log API keys, webhook path tokens, donor messages, or full provider payloads.
- Provide a controlled DB correction procedure before live points migration/correction.

## 7. Handoff to QA

- Test only active TC-M001–TC-M062 in the current regression baseline.
- Keep original subscriber/polling tests as historical evidence, not active pass criteria.
- Verify exact normalization, cutoff, inactive-member exclusion, handle conflict, and exactly-once point application.
- Inspect public JSON for data-minimization violations.
- Record real test execution results; specification review is not a test pass.

## 8. Open Items / Phase Gates

| Item | Owner | Gate |
|------|-------|------|
| Confirm EasyDonate payload field names from a real test event | Dev/DevOps | Before webhook production deployment |
| Confirm provider webhook signing/custom-auth support | Dev/DevOps | Before choosing HMAC/header implementation |
| Define retention period for raw donation data | PO/Owner | Before production launch |
| Confirm privacy notice and hide/removal wording | PO/Owner | Before public scoreboard launch |
| Use controlled DB procedure for manual point transfer | Owner/DevOps | Before first handle-change correction |

## Related Documents

| Document | Relationship |
|----------|-------------|
| `07_pm/072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801.md` | Source finding and scope pivot |
| `01_requirement/012_user_stories.md` | Current stories |
| `01_requirement/013_acceptance_criteria.md` | Current AC baseline |
| `external_plan/phase1-deerngo-bot-mvp.md` | Current implementation plan |

---

> **Handoff status:** Requirements and design baseline are updated. Dev may begin the revised member foundation work. QA may prepare execution against TC-M001–TC-M062.
