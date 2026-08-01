---
document_type: Commit Messages / Changelog
version: "0.2"
status: Draft
author: "SA / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "SHARED"
classification: "Internal"
tags: [commit-messages, changelog, conventional-commits, shared, members, privacy]
standard_ref:
  - SWEBOK v4 — Construction
  - Conventional Commits v1.0
parent_project: "Deerngo Bot — VRM"
---

# Commit Messages / Changelog — Deerngo Bot (Shared)

> **Project:** Deerngo Bot — VRM | **Repo:** Both (`deerngo-bot` + `deerngo-web`)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> Commit guidance reflects the explicit member-registration MVP. Historical subscriber/poller entries are retained only in the historical section.

---

## Commit Message Format

```text
<type>(<scope>): <description>

[optional body — explains WHY, not WHAT]

[optional footer(s)]
```

## Types

| Type | Description | Example |
|------|-----------|---------|
| `feat` | New feature | `feat(members): add explicit live-chat registration` |
| `fix` | Bug fix | `fix(points): prevent duplicate point application` |
| `docs` | Documentation | `docs(easydonate): record verified webhook contract` |
| `style` | Formatting only | `style: fix indentation` |
| `refactor` | Restructure | `refactor(points): extract transaction service` |
| `test` | Add/update tests | `test(members): cover changed-handle registration` |
| `chore` | Build/tooling/deps | `chore(deps): bump Fiber` |
| `perf` | Performance | `perf(scoreboard): add active-public index` |
| `ci` | CI/CD | `ci: add Docker build workflow` |
| `build` | Build system | `build: configure Dockerfile` |
| `revert` | Revert commit | `revert: feat(scoreboard): add realtime` |

## Scopes — Backend (`deerngo-bot`)

| Scope | Applies To | Example |
|-------|-----------|---------|
| `members` | Explicit registration and status | `feat(members): add changed-handle transaction` |
| `points` | Point application/query | `fix(points): enforce registration cutoff` |
| `scoreboard` | Scoreboard API | `feat(scoreboard): add active-public filter` |
| `matcher` | Exact donation matching | `feat(matcher): normalize donor names` |
| `donations` | Donation ingestion | `fix(donations): handle reference idempotency` |
| `webhook` | EasyDonate webhook | `docs(webhook): record provider auth contract` |
| `bot` | streamer.bot integration | `feat(bot): add private command response` |
| `docker` | Container config | `chore(docker): add webhook path token` |
| `db` | Migrations/schema | `chore(db): add members table` |
| `privacy` | Public-data minimization | `test(privacy): reject donor fields from scoreboard` |

## Scopes — Frontend (`deerngo-web`)

| Scope | Applies To | Example |
|-------|-----------|---------|
| `scoreboard` | Scoreboard page | `feat(scoreboard): implement member ranking table` |
| `components` | React components | `fix(components): responsive mobile layout` |
| `styles` | CSS/Tailwind/DaisyUI | `style(styles): apply deerngo theme` |
| `api` | API client/SWR | `feat(api): add privacy-safe scoreboard types` |
| `privacy` | Data allowlist | `test(privacy): never render display names` |
| `docker` | Container config | `chore(docker): multi-stage Dockerfile` |

## Commit Guidelines

| Rule | Rationale |
|------|----------|
| Imperative mood | “add” not “added” |
| First line ≤72 chars | Readable in `git log --oneline` |
| Body explains WHY | Code shows what; commit explains why |
| One logical change | Atomic and revertible |
| No WIP on main | Squash before merge |
| Reference stable story/issue ID | Keep docs/backlog traceable |
| Never include secrets/real donor data | Protect credentials and privacy |

## Current Changelog Baseline

```markdown
# Changelog

## [0.2.0] - 2026-08-02

### Product Scope
- **members:** replace incomplete YouTube subscriber capture with explicit `:deer: register`
- **members:** add active/inactive changed-handle behavior with manual point correction later
- **points:** use normalized exact matching and `donation_time >= registered_at`
- **privacy:** remove YouTube display names from the stored/public member model
- **scoreboard:** show only active, public, points>0 normalized handles and totals

### Integrations
- **donations:** keep EasyDonate webhook primary + API fallback
- **webhook:** require provider-contract verification; do not assume HMAC
- **bot:** add `:deer: public` and `:deer: private` behavior

### Removed from Active MVP
- YouTube API subscriber polling/OAuth dependency
- `subscribers` as the membership source
- `viewer_points` as the point summary table
- pg_trgm fuzzy attribution
- automatic historical point credit
```

## Historical Baseline

```markdown
## [0.1.0] - 2026-07-30

### Historical / Superseded
- subscribers: hybrid YouTube API + streamer.bot capture was implemented as an experiment
- webhook: HMAC-SHA256 was assumed before provider confirmation
- matcher: pg_trgm fuzzy matching was specified before exact matching was selected
- scoreboard: initial public page and pagination design
```

The historical baseline must not be used as the active implementation contract.

## Related Documents

| Document | Path |
|----------|------|
| Backend README | `03_construction/031_BE_README.md` |
| Frontend README | `03_construction/031_FE_README.md` |
| API Specification | `02_design/022_API_specification.md` |
| Scope Decision | `07_pm/072_MM07_po-to-dev-qa-member-registration-mvp_20260802.md` |

---

> **Template Standard:** Based on SWEBOK v4 and Conventional Commits v1.0
> **Usage:** Shared commit and changelog contract for the revised member-based MVP.
---
