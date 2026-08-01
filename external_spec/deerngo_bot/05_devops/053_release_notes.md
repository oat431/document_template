---
document_type: Release Notes
version: "0.2"
status: Draft
author: "DevOps / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [release-notes, changelog, swebok, conventional-commits, members, privacy]
standard_ref:
  - SWEBOK v4 — Operations
  - Conventional Commits v1.0
parent_project: "Deerngo Bot — VRM"
---

# Release Notes

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> This release-note baseline reflects the revised explicit-member MVP. It is a planning document; no release is claimed until Dev/QA provide execution evidence.

---

## 1. Purpose

Communicate product/runtime changes per release. Notes are generated from commits/issues, then reviewed by PO before publication. Never include secrets, raw donor data, or unverified provider claims.

## 2. Release Template

### vX.Y.Z — YYYY-MM-DD

**Release Type:** Major / Minor / Patch
**Deployed:** YYYY-MM-DD HH:MM (UTC+7)

#### New Features

| # | Feature | Repo | Description | Related |
|---|---------|------|-------------|---------|
| 1 | | | | |

#### Improvements

| # | Improvement | Before | After |
|---|-------------|--------|-------|
| 1 | | | |

#### Bug Fixes

| # | Fix | Issue | Impact |
|---|-----|-------|--------|
| 1 | | | |

#### Known Issues / Gates

| # | Issue | Workaround | Fix Planned |
|---|-------|-----------|-------------|
| 1 | EasyDonate provider webhook contract not verified | Keep production webhook disabled or use approved provider-confirmed path | Issue #18 |
| 2 | Manual point correction required after handle change | Controlled DB transaction + adjustment note | Issue #19; admin console Phase 2 |
| 3 | Privacy notice/removal gate | Do not release public scoreboard until owner approves | DEF-S009 |

#### Infrastructure Changes

| # | Change | Description |
|---|--------|-------------|
| 1 | Database | New `members`/revised `donations` migrations; no automatic historical migration |
| 2 | Webhook | Provider-confirmed auth or unpredictable path-token fallback |
| 3 | Public API | Allowlist active/public/points>0 handle+points only |

#### Migration Notes

| # | Action Required | Who | When |
|---|----------------|-----|------|
| 1 | Back up database before migration | DevOps | Before deploy |
| 2 | Verify EasyDonate payload/security | Dev/DevOps | Before webhook production |
| 3 | Verify streamer.bot action payloads | Dev/QA | Before live test |
| 4 | Confirm privacy notice/visibility behavior | PO/Owner | Before scoreboard release |

---

## 3. Current Planned Release — v0.1.0 Phase 1 MVP

**Release Type:** Major (initial active product)
**Target:** Phase 1 MVP completion

### Planned Features

| # | Feature | Repo | Description | User Story |
|---|---------|------|-------------|-----------|
| 1 | Explicit Member Registration | deerngo-bot | `:deer: register` uses actual streamer.bot identity; starts 0 points | US-001, US-002 |
| 2 | Donate Chat Command | deerngo-bot/integration | `:deer: donate` posts EasyDonate link | US-010 |
| 3 | Visibility Commands | deerngo-bot/integration | `:deer: public` / `:deer: private` | US-012 |
| 4 | Visibility-Aware Point Command | deerngo-bot | Exact public points or 100-point private band | US-011, US-022 |
| 5 | EasyDonate Webhook | deerngo-bot | Primary donation capture; contract must be verified | US-020 |
| 6 | EasyDonate API Sync | deerngo-bot | Fallback reconciliation every configured interval | US-020 |
| 7 | Normalized Exact Matcher | deerngo-bot | Active member handle match; 1 THB = 1 point; cutoff/once-only | US-021 |
| 8 | Scoreboard API | deerngo-bot | Paginated active/public/points>0 handle+points projection | US-031 |
| 9 | Public Scoreboard | deerngo-web | Public responsive page with no login and no display names | US-030 |

### Removed from Active Release

- YouTube subscriber polling/OAuth
- Automatic subscriber registration
- Subscriber table as points eligibility source
- `viewer_points` summary table
- pg_trgm fuzzy matching
- Automatic historical donation credit
- Admin console
- Automatic CI/CD production deployment

### Infrastructure

| # | Change | Description |
|---|--------|-------------|
| 1 | Docker Compose | `deerngo-bot` + `deerngo-web` on `db-network` |
| 2 | PostgreSQL | `members`, `donations`, optional `point_adjustment_notes` |
| 3 | Cloudflare Tunnel | Public scoreboard + intentionally configured webhook route |
| 4 | Manual deployment | Backup → migrate → compose → health/privacy/live smoke |

## 4. Release History

| Version | Date | Type | Features | Fixes | Status |
|---------|------|------|---------|-------|--------|
| v0.1.0 | TBD | Major | 9 planned feature groups | 0 | 🔜 Planned |

## 5. Conventional Commit Mapping

| Commit Type | Release Section |
|-------------|----------------|
| `feat` | New Features |
| `fix` | Bug Fixes |
| `perf` / `refactor` | Improvements |
| `docs` | Usually omitted; link if contract changed |
| `chore` / `ci` / `build` | Infrastructure |
| `test` / `style` | Usually omitted |

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[052_deployment_plan]] | Deployment procedure and gates |
| [[054_operations_manual_runbook]] | Operations and correction procedure |
| [[034_SHARED_commit_messages_changelog]] | Commit-level source |
| [[041_test_plan]] | Test scope |
| [[043_defect_report]] | Known blockers |
| `external_plan/phase1-deerngo-bot-mvp.md` | Phase plan |

---

> **Template Standard:** Based on SWEBOK v4 and Conventional Commits v1.0
> **Usage:** Release notes are published only from verified deployment/test evidence.
---

