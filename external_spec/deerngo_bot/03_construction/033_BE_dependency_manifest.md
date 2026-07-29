---
document_type: Dependency Manifest (Backend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [dependencies, go-modules, go, backend, sbom]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Top 10 (A06:2021 — Vulnerable Dependencies)
parent_project: "Deerngo Bot — VRM"
---

# Dependency Manifest — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## go.mod

```go
module github.com/deerngo/deerngo-bot

go 1.24

require (
    github.com/gofiber/fiber/v3 v3.0.0
    github.com/jmoiron/sqlx v1.4.0
    github.com/lib/pq v1.10.9
    github.com/golang-migrate/migrate/v4 v4.18.0
    golang.org/x/oauth2 v0.25.0
    github.com/google/uuid v1.6.0
)
```

---

## Direct Dependencies

| Module | Version | License | Purpose |
|--------|---------|---------|---------|
| `github.com/gofiber/fiber/v3` | v3.0.0 | MIT | HTTP web framework (fasthttp) |
| `github.com/jmoiron/sqlx` | v1.4.0 | MIT | DB access (extends database/sql) |
| `github.com/lib/pq` | v1.10.9 | MIT | PostgreSQL driver |
| `github.com/golang-migrate/migrate/v4` | v4.18.0 | MIT | Database migrations |
| `golang.org/x/oauth2` | v0.25.0 | BSD-3 | YouTube API OAuth2 |
| `github.com/google/uuid` | v1.6.0 | BSD-3 | UUID generation |

---

## Dev/Test Dependencies

| Module | Version | License | Purpose |
|--------|---------|---------|---------|
| `github.com/stretchr/testify` | v1.10.0 | MIT | Test assertions |
| `github.com/golangci/golangci-lint` | latest | GPL-3 | Linter |

---

## Dependency Notes

| Dependency | Notes |
|-----------|-------|
| `fiber/v3` | Built on fasthttp. NOT compatible with net/http middleware. |
| `sqlx` | Thin layer — no ORM, no code generation. Write SQL directly. |
| `lib/pq` | Pure Go, no CGO required. |
| `migrate/v4` | File-based: `001_initial_schema.up.sql`, `001_initial_schema.down.sql` |
| `oauth2` | Access tokens generated at runtime from refresh token in DB. |

---

## Dependency Policy

| Rule | Tool |
|------|------|
| No Critical/High vulnerabilities | `govulncheck` |
| Lock file committed | `go.sum` |
| Monthly audit | `go list -m all` |

---

## Related Documents

| Document | Path |
|----------|------|
| Build Scripts | `03_construction/033_BE_build_scripts.md` |
| ADR | `02_design/021_SHARED_architecture_decision_records.md` |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Top 10
