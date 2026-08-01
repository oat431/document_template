---
document_type: README (Backend)
version: "0.2"
status: Draft
author: "SA / Dev"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [readme, developer-guide, onboarding, go, fiber, sqlx, members, easydonate]
standard_ref:
  - SWEBOK v4 — Construction
  - 12-Factor App Methodology
parent_project: "Deerngo Bot — VRM"
---

# README — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Repo:** `deerngo-bot` (backend)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> Phase 1 uses explicit member registration from streamer.bot. The former YouTube subscriber poller is not part of the active MVP.

---

## Project Header

# Deerngo Bot — Backend 🦌

> Go backend for member registration, EasyDonate ingestion, normalized exact matching, points, visibility, and the public scoreboard API.

| Aspect | Detail |
|--------|--------|
| **Language** | Go 1.25+ |
| **Framework** | Fiber v3 |
| **DB Access** | sqlx |
| **Database** | PostgreSQL 18 (`deerngo`) |
| **Port** | 8008 |
| **Deployment** | Docker on homelab (`db-network`) |

---

## Quick Start

### Prerequisites

- Go 1.25+
- PostgreSQL 18 / isolated test database
- Docker (optional for local integration)
- streamer.bot only for live integration testing

### Build & Run

```bash
go mod download
go test ./...
go vet ./...
go build -o bin/deerngo-bot ./cmd/server
go run ./cmd/server
```

### Docker

```bash
docker network inspect db-network
docker compose --env-file .env up -d --build
docker compose ps
curl http://localhost:8008/healthz
docker compose down
```

---

## Configuration

| Variable | Required | Description |
|----------|:--------:|-------------|
| `DATABASE_URL` | ✅ | PostgreSQL connection string |
| `PORT` | — | Default `8008` |
| `EASYDONATE_API_KEY` | For fallback sync | Backend-only EasyDonate personal API key with donation-read scope |
| `EASYDONATE_WEBHOOK_PATH_TOKEN` | For webhook | Long random path token if provider does not offer signed webhooks |
| `EASYDONATE_WEBHOOK_SECRET` | ❌ | Use only if EasyDonate explicitly confirms a signing secret |
| `EASYDONATE_WEBHOOK_SIGNATURE_HEADER` | ❌ | Use only if provider confirms a signature header |
| `SCOREBOARD_ORIGIN` | — | Default `http://localhost:3008` |
| `EASYDONATE_POLL_INTERVAL_MINUTES` | — | Fallback sync interval, default 5 |

Do not configure YouTube OAuth or `YOUTUBE_CHANNEL_ID` for the active Phase 1 member-registration MVP.

Never commit secrets, `.env`, API keys, path tokens, or provider payloads containing personal data.

---

## API Reference

See `02_design/022_API_specification.md`.

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/healthz` | GET | Health check |
| `/api/v1/members/register` | POST | Register/identify member from streamer.bot identity |
| `/api/v1/members/{youtube_user_id}/visibility` | PUT | `:deer: public` / `:deer: private` |
| `/api/v1/members/{youtube_user_id}/points` | GET | Exact or 100-point-band result |
| `/api/v1/scoreboard` | GET | Public active/public/points>0 projection |
| `/api/v1/webhooks/easydonate/{path_token}` | POST | Primary donation ingestion |

---

## Member Rules

- Use the actual `youtube_user_id` and current handle supplied by streamer.bot.
- Ignore any handle typed in `:deer: register` text.
- New members start at 0 points and public visibility enabled.
- Same active identity/handle re-registration is idempotent.
- New handle for the same user creates a new active 0-point record and inactivates the old record.
- Active handle conflicts return `409 HANDLE_IN_USE`.
- Inactive members are excluded from matching, point queries, and scoreboard.
- No YouTube display name is stored.

## Donation and Points Rules

- EasyDonate webhook is primary; API-key polling is fallback.
- Confirm actual EasyDonate payload and security behavior before production.
- Use `referenceNo` as the idempotency key.
- Normalize donor name and member handle: trim, remove leading `@`, lowercase.
- Exact match only; no `pg_trgm` fuzzy matching in the active MVP.
- Donation qualifies only when `donation_time >= member.registered_at`.
- Apply `amount_thb` once to `members.total_points`.
- Keep raw donor data private; never return it from public APIs.

---

## Testing

```bash
go test -race -count=1 ./...
go vet ./...
go build ./cmd/server
git diff --check
```

Test separately:

- member registration and re-registration
- active-handle conflicts
- visibility commands
- public/private point responses
- normalized exact donation matching
- pre-registration cutoff
- duplicate `referenceNo`
- webhook path/payload validation
- API fallback and 429 backoff
- public data minimization

---

## Project Structure

```text
deerngo-bot/
├── cmd/
│   └── server/main.go
├── internal/
│   ├── config/                 # Environment configuration
│   ├── handler/                # Member, visibility, points, scoreboard, webhook
│   ├── service/                # Member, donation, matching, points services
│   ├── repository/             # sqlx queries and transactions
│   ├── scheduler/              # EasyDonate fallback sync only
│   ├── client/                 # EasyDonate client
│   └── middleware/             # CORS, rate limit, body limit, logger
├── migrations/                 # Versioned members/donations schema
├── go.mod
├── go.sum
├── Dockerfile
├── Makefile
└── README.md
```

---

## Related Documents

| Document | Path |
|----------|------|
| User Stories | `01_requirement/012_user_stories.md` |
| Acceptance Criteria | `01_requirement/013_acceptance_criteria.md` |
| API Specification | `02_design/022_API_specification.md` |
| Database Schema | `02_design/023_database_schema_DDL.md` |
| ERD | `02_design/024_ERD.md` |
| Security Standards | `06_security/062_coding_standards_security.md` |
| MM06 Scope Decision | `07_pm/072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801.md` |

---

> **Template Standard:** Based on SWEBOK v4 and 12-Factor App
> **Usage:** This README is the construction starting point for the revised member-based MVP.
