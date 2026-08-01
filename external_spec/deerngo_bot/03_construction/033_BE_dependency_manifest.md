---
document_type: Dependency Manifest (Backend)
version: "0.2"
status: Draft
author: "SA / Dev"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [dependencies, go-modules, go, backend, sbom, members, easydonate]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Top 10 (A06:2021 — Vulnerable Dependencies)
parent_project: "Deerngo Bot — VRM"
---

# Dependency Manifest — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> The active MVP does not require YouTube OAuth or YouTube Data API polling dependencies.

---

## go.mod Baseline

```go
module github.com/oat431/deerngo-bot

go 1.25

require (
    github.com/gofiber/fiber/v3 v3.0.0
    github.com/jmoiron/sqlx v1.4.0
    github.com/lib/pq v1.10.9
    github.com/golang-migrate/migrate/v4 v4.18.0
    github.com/google/uuid v1.6.0
)
```

> Versions are a design baseline. Dev must verify compatible current versions in the repository before implementation.

## Direct Dependencies

| Module | Purpose |
|--------|---------|
| `github.com/gofiber/fiber/v3` | HTTP API, middleware |
| `github.com/jmoiron/sqlx` | PostgreSQL access and transactions |
| `github.com/lib/pq` | PostgreSQL driver |
| `github.com/golang-migrate/migrate/v4` | Versioned schema migrations |
| `github.com/google/uuid` | UUID generation |
| Go standard library `crypto/hmac`/`crypto/sha256` | Only if EasyDonate confirms HMAC; do not add provider-specific auth without contract |
| Go standard library `net/http`/`encoding/json` | EasyDonate API client/webhook payloads |

## Removed from Active MVP

| Dependency | Reason |
|------------|--------|
| `golang.org/x/oauth2` | No YouTube OAuth/polling in active Phase 1 |
| YouTube Data API client | No subscriber polling |
| `pg_trgm` database extension | Exact normalized matching replaces fuzzy matching |

## Dev/Test Dependencies

| Module | Purpose |
|--------|---------|
| `github.com/stretchr/testify` | Assertions and test helpers |
| `golangci-lint` | Static analysis tool, installed in CI/toolchain |

## Dependency Policy

- No critical/high vulnerabilities: `govulncheck ./...`.
- Commit `go.mod` and `go.sum`.
- Run monthly dependency review.
- Pin or review container base images.
- Never add a provider SDK that sends data outside the documented EasyDonate contract without PO/Dev review.

## Related Documents

| Document | Path |
|----------|------|
| Backend README | `03_construction/031_BE_README.md` |
| Security standards | `06_security/062_coding_standards_security.md` |
| ADRs | `02_design/021_architecture_decision_records.md` |

---

> **Template Standard:** Based on SWEBOK v4 and OWASP Top 10
> **Usage:** Active member-based MVP dependency baseline.
---
