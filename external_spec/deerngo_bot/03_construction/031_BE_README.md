---
document_type: README (Backend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [readme, developer-guide, onboarding, go, backend, fiber, sqlx]
standard_ref:
  - SWEBOK v4 — Construction
  - 12-Factor App Methodology
parent_project: "Deerngo Bot — VRM"
---

# README — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Repo:** `deerngo-bot` (backend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## Project Header

# Deerngo Bot — Backend 🦌

> Go backend for the Deerngo Bot VRM system — handles subscriber capture, donation processing, name matching, points calculation, and serves the REST API.

| Aspect | Detail |
|--------|--------|
| **Language** | Go 1.24+ |
| **Framework** | Fiber v3 (fasthttp) |
| **DB Access** | sqlx (extends database/sql) |
| **Database** | PostgreSQL 18 (`deerngo` database) |
| **Port** | 8008 |
| **Deployment** | Docker on homelab (`db-network`) |

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│              Go Backend (:8008)                  │
│                                                  │
│  ┌──────────────┐  ┌──────────────┐             │
│  │ API Layer    │  │ Schedulers   │             │
│  │ (Fiber v3)   │  │ (goroutines) │             │
│  ├──────────────┤  ├──────────────┤             │
│  │ Service Layer│  │ YouTube Poll │             │
│  │ (business)   │  │ EasyDonate   │             │
│  ├──────────────┤  │ Name Matcher │             │
│  │ Repository   │  └──────────────┘             │
│  │ (sqlx)       │                               │
│  └──────┬───────┘                               │
│         │                                       │
│         ▼                                       │
│  ┌──────────────┐                               │
│  │ PostgreSQL   │                               │
│  │ :5432        │                               │
│  └──────────────┘                               │
└─────────────────────────────────────────────────┘
```

---

## Quick Start

### Prerequisites

- **Go** 1.24+
- **PostgreSQL 18** running on homelab (`db-network`)

### Build & Run

```bash
# Download dependencies
go mod download

# Run database migrations
make migrate-up

# Build
make build

# Run
make run
# → API server starts on :8008
```

### Docker

```bash
# Build image
make docker-build

# Run container
make docker-run
```

### Verify

```bash
curl http://localhost:8008/api/v1/health
# Expected: {"status":"ok"}
```

---

## Configuration

| Variable | Required | Default | Description |
|----------|:--------:|---------|------------|
| `DATABASE_URL` | ✅ | — | PostgreSQL connection string |
| `PORT` | — | `8008` | Listen port |
| `EASYDONATE_WEBHOOK_SECRET` | ✅ | — | HMAC-SHA256 secret |
| `YOUTUBE_CHANNEL_ID` | ✅ | — | YouTube channel ID |
| `SCOREBOARD_ORIGIN` | — | `http://localhost:3008` | CORS origin |

---

## API Reference

See [[022_SHARED_API_specification]].

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/health` | GET | Health check |
| `/api/v1/subscribers` | POST | Register subscriber |
| `/api/v1/points/{handle}` | GET | Query viewer points |
| `/api/v1/scoreboard` | GET | Public scoreboard |
| `/api/v1/webhooks/easydonate` | POST | Donation webhook |

---

## Project Structure

```
deerngo-bot/
├── cmd/
│   ├── server/main.go          # API server entry point
│   └── migrate/main.go         # Database migration runner
├── internal/
│   ├── config/config.go        # Environment variable loading
│   ├── handler/                # HTTP handlers (Fiber)
│   ├── service/                # Business logic
│   ├── repository/             # SQL queries (sqlx)
│   ├── scheduler/              # Background goroutines
│   ├── client/                 # External API clients
│   └── middleware/             # CORS, rate limit, logger
├── migrations/                 # SQL migration files
├── go.mod
├── go.sum
├── Dockerfile
├── Makefile
└── README.md
```

---

## Testing

```bash
make test              # Run all tests
make test-verbose      # Detailed output
make test-coverage     # Coverage report
make check             # fmt + lint + test + build
```

---

## Related Documents

| Document | Path | Purpose |
|----------|------|---------|
| API Specification | `02_design/022_SHARED_API_specification.md` | Endpoint contracts |
| Database Schema | `02_design/023_BE_database_schema_DDL.md` | DDL + triggers |
| ERD | `02_design/024_BE_ERD.md` | Data model |
| Build Scripts | `03_construction/033_BE_build_scripts.md` | Build pipeline |
| Dependency Manifest | `03_construction/035_BE_dependency_manifest.md` | Go dependencies |

---

> **Template Standard:** Based on SWEBOK v4, 12-Factor App
