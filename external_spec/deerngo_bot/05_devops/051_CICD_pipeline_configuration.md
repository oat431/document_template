---
document_type: CI/CD Pipeline Configuration
version: "0.2"
status: Draft
author: "DevOps / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [ci-cd, pipeline, github-actions, devops, docker, go, nextjs, members, privacy]
standard_ref:
  - SWEBOK v4 — Operations
  - 12-Factor App (Build, release, run)
parent_project: "Deerngo Bot — VRM"
---

# CI/CD Pipeline Configuration

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Phase 1 deployment decision:** CI/CD is a future/optional path. Phase 1 uses reviewed manual Docker Compose deployment. These workflows may build/test images, but must not silently deploy production without the approved deployment gate.

---

## 1. Purpose

Define optional GitHub Actions checks/builds for the two repositories while preserving the Phase 1 manual deployment decision. The pipeline must validate the revised member-based MVP and never expose secrets or real donor data.

## 2. Pipeline Overview

```mermaid
flowchart LR
    PR[Pull Request] --> LINT[Lint / Format]
    LINT --> TEST[Unit + Integration Tests]
    TEST --> SCAN[Dependency/Container Scan]
    SCAN --> BUILD[Build Docker Image]
    BUILD --> ARTIFACT[Push Immutable GHCR Artifact]
    ARTIFACT -.-> MANUAL[Manual approved deployment]
    MANUAL --> MIGRATE[Backup + migrate]
    MIGRATE --> SMOKE[Health + privacy smoke test]

    style PR fill:#1565C0,color:#fff
    style SCAN fill:#f44336,color:#fff
    style ARTIFACT fill:#9C27B0,color:#fff
    style MANUAL fill:#FF9800,color:#fff
    style SMOKE fill:#4CAF50,color:#fff
```

## 3. Pipeline Strategy

| Aspect | Phase 1 Choice | Rationale |
|--------|----------------|-----------|
| CI Platform | GitHub Actions | PR checks and artifact builds |
| Registry | GHCR | Existing repository ecosystem |
| Production deploy | Manual approved Compose run | Prevent unreviewed schema/provider/privacy changes from deploying |
| Branches | `main`, `develop`, feature branches | Project plan workflow |
| Image tags | Immutable SHA + release tag | Rollback and traceability |
| Database | Service PostgreSQL in CI | Migration/transaction tests |
| Provider | Mock EasyDonate in CI; real provider test manually | Protect credentials and avoid accidental donations |
| YouTube | No API/OAuth tests in active MVP | Subscriber polling removed |

## 4. Backend CI Checks

Required checks for `oat431/deerngo-bot`:

```bash
gofmt -l . | tee /tmp/fmt-check
test ! -s /tmp/fmt-check
go vet ./...
golangci-lint run
go test -race -cover ./...
govulncheck ./...
go build ./cmd/server
# Run migration up/down against isolated PostgreSQL
git diff --check
```

Test focus:

- member normalization and active uniqueness
- same/changed-handle registration transaction
- visibility and identity-based point API
- EasyDonate payload validation/path-token fallback
- reference idempotency and exactly-once points
- cutoff and exact matching
- public response allowlist
- log redaction

No real API key, provider secret, donor name, or production URL is required in CI.

## 5. Frontend CI Checks

Required checks for `oat431/deerngo-web`:

```bash
npm ci
npm run lint
npm run type-check
npm test -- --run
npm run build
```

Test focus:

- allowlisted scoreboard fields only
- no display-name rendering
- private/inactive/zero-point data not rendered
- loading/empty/error/retry states
- pagination and refresh
- responsive/accessibility behavior

## 6. Example PR Workflow

```yaml
name: PR Checks

on:
  pull_request:
    branches: [develop, main]

jobs:
  backend:
    if: contains(github.repository, 'deerngo-bot')
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:18
        env:
          POSTGRES_DB: deerngo_test
          POSTGRES_PASSWORD: test
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
      - run: gofmt -l . | tee /tmp/fmt && test ! -s /tmp/fmt
      - run: go vet ./...
      - run: go test -race -cover ./...
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/deerngo_test?sslmode=disable
      - run: go build ./cmd/server

  frontend:
    if: contains(github.repository, 'deerngo-web')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm test -- --run
      - run: npm run build
```

> In the real repository, split jobs/workflows as appropriate. Do not copy this example blindly without verifying the current scaffold.

## 7. GHCR Build Artifact

Build/push may occur after protected-branch checks:

- backend: `ghcr.io/oat431/deerngo-bot:<sha>`
- frontend: `ghcr.io/oat431/deerngo-web:<sha>`

Use least-privilege `GITHUB_TOKEN` permissions (`contents: read`, `packages: write` only for artifact job). Do not place EasyDonate or database secrets in GitHub Actions unless a later approved deployment design requires them.

## 8. Manual Production Deployment Gate

Before production:

1. Review/merge PR to approved branch.
2. Build or pull immutable images.
3. Back up PostgreSQL.
4. Confirm active migration and provider contract.
5. Apply migration.
6. Deploy backend/frontend manually.
7. Run `/healthz`, API, public-data-allowlist, and synthetic smoke tests.
8. Verify streamer.bot integration during an approved live/test session.
9. Record outcome in release notes/meeting minute.

No automatic deploy on push to `main` is required for Phase 1.

## 9. Secrets

| Secret | Where | Rule |
|--------|-------|------|
| `DATABASE_URL` | Homelab secret store | Never CI logs/Git |
| `EASYDONATE_API_KEY` | Homelab backend secret | Never browser/CI output |
| `EASYDONATE_WEBHOOK_PATH_TOKEN` | Homelab/backend + provider dashboard | Never issue/docs/logs |
| Provider signature secret | Only if confirmed | Store as secret; exact scheme only |
| SSH deploy key | Future CI/CD only | Not needed for Phase 1 manual path |

## 10. Rollback

- Prefer rolling back immutable application images without rolling back database migrations.
- If migration/data integrity is implicated, stop writes, preserve logs/backups, and follow the runbook recovery procedure.
- Do not auto-run destructive migration down.

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[052_deployment_plan]] | Current manual deployment decision |
| [[054_operations_manual_runbook]] | Operations/rollback/backup |
| [[061_security_test_report]] | Security gates |
| [[035_BE_coding_standards]] | Code quality/security rules |
| `external_plan/phase1-deerngo-bot-mvp.md` | Sprint and deployment plan |

---

> **Template Standard:** Based on SWEBOK v4 and 12-Factor App
> **Usage:** Optional CI/build source of truth; production deployment remains manually gated during Phase 1.
---

