---
document_type: Commit Messages / Changelog
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "SHARED"
classification: "Internal"
tags: [commit-messages, changelog, conventional-commits, shared]
standard_ref:
  - SWEBOK v4 — Construction
  - Conventional Commits v1.0
parent_project: "Deerngo Bot — VRM"
---

# Commit Messages / Changelog — Deerngo Bot (Shared)

> **Project:** Deerngo Bot — VRM | **Repo:** Both (`deerngo-bot` + `deerngo-web`)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## Commit Message Format

```
<type>(<scope>): <description>

[optional body — explains WHY, not WHAT]

[optional footer(s)]
```

---

## Types

| Type | Description | Example |
|------|-----------|---------|
| `feat` | New feature | `feat(subscribers): add YouTube API polling` |
| `fix` | Bug fix | `fix(matcher): handle anonymous donations` |
| `docs` | Documentation | `docs(readme): add Docker setup` |
| `style` | Formatting (no logic change) | `style: fix indentation` |
| `refactor` | Restructure (no feature/fix) | `refactor(points): extract to repository` |
| `test` | Add/update tests | `test(matcher): add fuzzy match cases` |
| `chore` | Build, tooling, deps | `chore(deps): bump Fiber to v3.0.0` |
| `perf` | Performance | `perf(scoreboard): add index on total_points` |
| `ci` | CI/CD | `ci: add Docker build workflow` |
| `build` | Build system | `build: configure Dockerfile` |
| `revert` | Revert commit | `revert: feat(scoreboard): add realtime` |

---

## Scopes — Backend (`deerngo-bot`)

| Scope | Applies To | Example |
|-------|-----------|---------|
| `subscribers` | Subscriber capture | `feat(subscribers): add YouTube API polling` |
| `points` | Points query | `fix(points): case-insensitive handle lookup` |
| `scoreboard` | Scoreboard API | `feat(scoreboard): add pagination` |
| `matcher` | Name matching | `feat(matcher): pg_trgm fuzzy matching` |
| `donations` | Donation sync | `fix(donations): handle 429 rate limit` |
| `webhook` | EasyDonate webhook | `feat(webhook): HMAC-SHA256 verification` |
| `bot` | streamer.bot integration | `feat(bot): donate command response` |
| `youtube` | YouTube API client | `fix(youtube): handle 401 token expired` |
| `docker` | Container config | `chore(docker): add healthcheck` |
| `db` | Migrations/schema | `chore(db): initial schema migration` |

## Scopes — Frontend (`deerngo-web`)

| Scope | Applies To | Example |
|-------|-----------|---------|
| `scoreboard` | Scoreboard page | `feat(scoreboard): implement ranking table` |
| `components` | React components | `fix(components): responsive mobile layout` |
| `styles` | CSS/Tailwind/DaisyUI | `style(styles): apply deerngo theme` |
| `api` | API client / SWR | `feat(api): add scoreboard data fetching` |
| `docker` | Container config | `chore(docker): multi-stage Dockerfile` |

---

## Commit Guidelines

| Rule | Rationale |
|------|----------|
| Imperative mood | "add" not "added" |
| First line ≤ 72 chars | Readable in `git log --oneline` |
| Body explains WHY | Code shows what; commit explains why |
| One logical change | Atomic → easy to revert |
| No WIP on main | Squash before merge |

---

## Changelog

```markdown
# Changelog

## [0.1.0] - 2026-07-30

### Features
- **subscribers:** add hybrid capture via YouTube API + streamer.bot
- **webhook:** add HMAC-SHA256 verification
- **matcher:** implement pg_trgm fuzzy matching
- **scoreboard:** add pagination with 0-point exclusion
- **scoreboard:** implement public page with DaisyUI

### Bug Fixes
- **matcher:** handle anonymous donations
- **youtube:** handle 401 token expired gracefully

### Chores
- **docker:** multi-stage Dockerfiles for backend + frontend
- **db:** initial schema migration with pg_trgm
```

---

## Related Documents

| Document | Path |
|----------|------|
| README (BE) | `03_construction/031_BE_README.md` |
| README (FE) | `03_construction/032_FE_README.md` |

---

> **Template Standard:** Based on SWEBOK v4, Conventional Commits v1.0
