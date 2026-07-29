---
document_type: Release Notes
version: "0.1"
status: Draft
author: "DevOps"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [release-notes, changelog, swebok, conventional-commits]
standard_ref:
  - SWEBOK v4 — Operations
  - Conventional Commits v1.0
parent_project: "Deerngo Bot — VRM"
---

# Release Notes

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | DevOps / PO |
| Format | Conventional Commits → auto-compiled |
| Repositories | `deerngo-bot` (Go), `deerngo-web` (Next.js) |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-30 | DevOps | Initial release notes template |

---

## 1. Purpose

> Communicates what changed in each release — new features, improvements, bug fixes, and known issues. Auto-generated from conventional commit messages, reviewed by PO before publishing.

---

## 2. How to Generate Release Notes

### From Git Log (Manual)

```bash
# Backend — changes since last tag
cd deerngo-bot
git log $(git describe --tags --abbrev=0)..HEAD --oneline --no-merges

# Frontend — changes since last tag
cd deerngo-web
git log $(git describe --tags --abbrev=0)..HEAD --oneline --no-merges
```

### From GitHub (Auto)

GitHub auto-generates release notes from PRs when creating a release:
1. Go to repo → Releases → Draft new release
2. Choose tag (e.g., `v0.1.0`)
3. Click "Generate release notes"
4. Review and edit

---

## 3. Release Template

Copy this template for each new release:

---

### vX.Y.Z — YYYY-MM-DD

**Release Type:** Major / Minor / Patch
**Deployed:** YYYY-MM-DD HH:MM (UTC+7)

#### 🚀 New Features

| # | Feature | Repo | Description | Related |
|---|---------|------|-------------|---------|
| 1 | | deerngo-bot / deerngo-web | | |

#### ✨ Improvements

| # | Improvement | Before | After |
|---|-----------|--------|-------|
| 1 | | | |

#### 🐛 Bug Fixes

| # | Fix | Issue | Impact |
|---|-----|-------|--------|
| 1 | | | |

#### ⚠️ Known Issues

| # | Issue | Workaround | Fix Planned |
|---|-------|-----------|------------|
| 1 | | | |

#### 📦 Infrastructure Changes

| # | Change | Description |
|---|--------|-------------|
| 1 | | |

#### 🔄 Migration Notes

| # | Action Required | Who | When |
|---|----------------|-----|------|
| 1 | | Dev / DevOps | Before / After deploy |

#### 📊 Release Metrics

| Metric | Value |
|--------|-------|
| Total commits | |
| Features | |
| Bug fixes | |
| Files changed | |

---

## 4. Planned Releases

### v0.1.0 — Phase 1 MVP (Planned)

**Release Type:** Major (initial release)
**Target:** Phase 1 MVP completion

#### 🚀 New Features (Planned)

| # | Feature | Repo | Description | User Story |
|---|---------|------|-------------|-----------|
| 1 | Subscriber Registration (Hybrid) | deerngo-bot | YouTube API polling (24/7) + streamer.bot real-time (during live) | US-001, US-002, US-003 |
| 2 | Donate Chat Command | deerngo-bot | `:deer: donate` → posts EasyDonate link | US-010 |
| 3 | Point Chat Command | deerngo-bot | `:deer: point` → shows viewer's points | US-011 |
| 4 | EasyDonate Webhook | deerngo-bot | Real-time donation capture via webhook | US-020 |
| 5 | EasyDonate API Sync | deerngo-bot | Polling fallback every 5 min | US-020 |
| 6 | Name Matching Engine | deerngo-bot | Fuzzy matching via `pg_trgm` (threshold 0.7) | US-021 |
| 7 | Point Query API | deerngo-bot | `GET /api/v1/points/{handle}` | US-022 |
| 8 | Scoreboard API | deerngo-bot | `GET /api/v1/scoreboard` with pagination | US-030 |
| 9 | Public Scoreboard | deerngo-web | Next.js page with viewer rankings | US-031 |
| 10 | CI/CD Pipeline | infra | GitHub Actions → GHCR → Homelab deploy | — |

#### 📦 Infrastructure

| # | Change | Description |
|---|--------|-------------|
| 1 | Docker Compose | `deerngo-bot` + `deerngo-web` on `db-network` |
| 2 | GitHub Actions | Lint → Test → Build → Push → Deploy |
| 3 | GHCR Registry | `ghcr.io/deerngo/deerngo-bot`, `ghcr.io/deerngo/deerngo-web` |
| 4 | Nginx Config | Host-level proxy for scoreboard (`:3008` → public) |
| 5 | Cloudflare Tunnel | Scoreboard exposed at `deerngo-viewer-score.panomete.com` |

---

## 5. Release History

| Version | Date | Type | Features | Fixes | Status |
|---------|------|------|---------|-------|--------|
| v0.1.0 | TBD | Major | 10 | 0 | 🔜 Planned |

---

## 6. Conventional Commit → Release Note Mapping

| Commit Type | Release Section |
|-------------|----------------|
| `feat` | 🚀 New Features |
| `fix` | 🐛 Bug Fixes |
| `perf` | ✨ Improvements |
| `refactor` | ✨ Improvements |
| `docs` | (omitted from release notes) |
| `chore` | 📦 Infrastructure Changes |
| `ci` | 📦 Infrastructure Changes |
| `build` | 📦 Infrastructure Changes |
| `test` | (omitted from release notes) |
| `style` | (omitted from release notes) |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[052_deployment_plan]] | How this release was deployed |
| [[034_SHARED_commit_messages_changelog]] | Commit-level changes (source data) |
| [[041_test_plan]] | What was tested |
| [[043_defect_report]] | Known defects |

---

> **Template Standard:** Based on SWEBOK v4, Conventional Commits v1.0
> **Usage:** Release notes are for *everyone* — developers, PO, users. Write them for humans, not machines.
