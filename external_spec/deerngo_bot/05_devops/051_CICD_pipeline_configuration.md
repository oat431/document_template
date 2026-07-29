---
document_type: CI/CD Pipeline Configuration
version: "0.1"
status: Draft
author: "DevOps"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [ci-cd, pipeline, github-actions, devops, docker, go, nextjs]
standard_ref:
  - SWEBOK v4 — Operations
  - 12-Factor App (Build, release, run)
parent_project: "Deerngo Bot — VRM"
---

# CI/CD Pipeline Configuration

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | DevOps |
| Repositories | `deerngo-bot` (Go backend), `deerngo-web` (Next.js frontend) |
| CI/CD Platform | GitHub Actions |
| Container Registry | GHCR (`ghcr.io`) |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-30 | DevOps | Initial CI/CD pipeline — dual-repo, GHCR, homelab deploy |

---

## 1. Purpose

> Defines the CI/CD pipeline for Deerngo Bot — automated build, test, container image creation, and deployment to the homelab server. Two separate pipelines (one per repo) that produce Docker images pushed to GHCR, then pulled by the homelab server.

---

## 2. Pipeline Overview

```mermaid
flowchart LR
    subgraph PR["Pull Request"]
        PUSH_PR[Push] --> LINT[Lint]
        LINT --> TYPE[Type Check]
        TYPE --> TEST[Unit Tests]
    end

    subgraph MAIN["Merge to main"]
        PUSH_MAIN[Push to main] --> LINT2[Lint]
        LINT2 --> TEST2[Tests]
        TEST2 --> BUILD[Build Docker Image]
        BUILD --> SCAN[Security Scan]
        SCAN --> PUSH_REG[Push to GHCR]
        PUSH_REG --> DEPLOY[Deploy to Homelab]
        DEPLOY --> SMOKE[Smoke Test]
    end

    style PR fill:#1565C0,color:#fff
    style MAIN fill:#2E7D32,color:#fff
    style BUILD fill:#9C27B0,color:#fff
    style SCAN fill:#f44336,color:#fff
    style DEPLOY fill:#4CAF50,color:#fff
```

---

## 3. Pipeline Strategy

| Aspect | Choice | Rationale |
|--------|--------|-----------|
| CI Platform | GitHub Actions | Free for public repos, tight GitHub integration |
| Container Registry | GHCR (`ghcr.io`) | No extra auth for GitHub repos, same ecosystem |
| Deploy Method | SSH + `docker compose pull && up` | Simple, no orchestrator needed for 2 containers |
| Branch Strategy | `main` = production, PRs = CI only | Single developer, no staging environment |
| Image Tagging | SHA-based (`sha-abc1234`) + `latest` | Immutable tags for rollback, `latest` for convenience |

---

## 4. Pipeline — Backend (`deerngo-bot`)

### 4.1 Workflow File

```yaml
# .github/workflows/ci-cd.yml  (in deerngo-bot repo)
name: CI/CD — Backend

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint-and-test:
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
          go-version: '1.24'

      - name: Install linting tools
        run: go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

      - name: Lint
        run: golangci-lint run

      - name: Format check
        run: |
          gofmt -l . | tee /tmp/fmt-check
          test ! -s /tmp/fmt-check

      - name: Run tests
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/deerngo_test?sslmode=disable
        run: go test -v -cover ./...

      - name: Build binary
        run: go build -o /dev/null ./cmd/server

  build-and-push:
    needs: lint-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Generate image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=raw,value=latest

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Deploy to homelab via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.HOMELAB_HOST }}
          username: ${{ secrets.HOMELAB_USER }}
          key: ${{ secrets.HOMELAB_SSH_KEY }}
          script: |
            cd ~/platform
            docker compose -f docker-compose.deerngo.yml pull deerngo-bot
            docker compose -f docker-compose.deerngo.yml up -d deerngo-bot
            sleep 5
            curl -sf http://localhost:8008/api/v1/health || exit 1
            echo "Backend deployed and healthy"
```

### 4.2 Pipeline Stages — Backend

| Stage | Purpose | Duration | Failure Action |
|-------|---------|---------|---------------|
| Lint | `golangci-lint` + `gofmt` check | < 1 min | Block merge |
| Test | Unit + integration tests (PostgreSQL service) | < 3 min | Block merge |
| Build Binary | `go build` sanity check | < 1 min | Block merge |
| Docker Build | Multi-stage image → GHCR | < 3 min | Alert |
| Deploy | SSH → `docker compose pull && up` | < 2 min | Alert + manual check |
| Smoke Test | `curl /api/v1/health` | < 10s | Alert + rollback |

---

## 5. Pipeline — Frontend (`deerngo-web`)

### 5.1 Workflow File

```yaml
# .github/workflows/ci-cd.yml  (in deerngo-web repo)
name: CI/CD — Frontend

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Unit tests
        run: npm test -- --run

      - name: Build
        run: npm run build

  build-and-push:
    needs: lint-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Generate image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=raw,value=latest

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Deploy to homelab via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.HOMELAB_HOST }}
          username: ${{ secrets.HOMELAB_USER }}
          key: ${{ secrets.HOMELAB_SSH_KEY }}
          script: |
            cd ~/platform
            docker compose -f docker-compose.deerngo.yml pull deerngo-web
            docker compose -f docker-compose.deerngo.yml up -d deerngo-web
            sleep 5
            curl -sf http://localhost:3008 || exit 1
            echo "Frontend deployed and healthy"
```

### 5.2 Pipeline Stages — Frontend

| Stage | Purpose | Duration | Failure Action |
|-------|---------|---------|---------------|
| Lint | ESLint | < 1 min | Block merge |
| Type Check | `tsc --noEmit` | < 1 min | Block merge |
| Test | Vitest unit tests | < 2 min | Block merge |
| Build | `next build` | < 3 min | Block merge |
| Docker Build | Multi-stage image → GHCR | < 3 min | Alert |
| Deploy | SSH → `docker compose pull && up` | < 2 min | Alert |
| Smoke Test | `curl http://localhost:3008` | < 10s | Alert |

---

## 6. GitHub Secrets Required

| Secret | Repository | Description | Rotation |
|--------|-----------|-------------|----------|
| `HOMELAB_HOST` | Both | Homelab server IP / Tailscale hostname | On network change |
| `HOMELAB_USER` | Both | SSH username (`flowero`) | — |
| `HOMELAB_SSH_KEY` | Both | SSH private key for deploy | Quarterly |
| `GITHUB_TOKEN` | Both | Auto-provided by GitHub Actions | Automatic |

> **Note:** `GITHUB_TOKEN` is automatically available — no manual setup needed. The other 3 secrets must be added to each repo's Settings → Secrets → Actions.

---

## 7. Environment Configuration

| Environment | Branch | Auto-Deploy | Approval | URL |
|------------|--------|------------|---------|-----|
| Local Dev | feature branches | No | — | `localhost:8008` / `localhost:3008` |
| Production | `main` | Yes (on push) | No (single dev) | `deerngo-viewer-score.panomete.com` |

> **Note:** Phase 1 has no staging environment. The homelab IS production. Local Docker Compose serves as the dev/test environment. A staging environment can be added in Phase 2 if needed.

---

## 8. Docker Compose — Production (Homelab)

```yaml
# docker-compose.deerngo.yml  (on homelab server at ~/platform/)
services:
  deerngo-bot:
    image: ghcr.io/deerngo/deerngo-bot:latest
    container_name: deerngo-bot
    restart: unless-stopped
    ports:
      - "127.0.0.1:8008:8008"
    environment:
      - DATABASE_URL=${DERNBOT_DATABASE_URL}
      - PORT=8008
      - EASYDONATE_WEBHOOK_SECRET=${EASYDONATE_WEBHOOK_SECRET}
      - YOUTUBE_CHANNEL_ID=${YOUTUBE_CHANNEL_ID}
      - SCOREBOARD_ORIGIN=https://deerngo-viewer-score.panomete.com
    networks:
      - db-network
    depends_on:
      local-postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8008/api/v1/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s

  deerngo-web:
    image: ghcr.io/deerngo/deerngo-web:latest
    container_name: deerngo-web
    restart: unless-stopped
    ports:
      - "127.0.0.1:3008:3008"
    environment:
      - NEXT_PUBLIC_API_URL=http://deerngo-bot:8008
    networks:
      - db-network
    depends_on:
      - deerngo-bot
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3008"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s

networks:
  db-network:
    external: true
```

---

## 9. Nginx Configuration (Host-Level)

```nginx
# /etc/nginx/sites-available/deerngo-scoreboard
server {
    listen 80;
    server_name deerngo-viewer-score.panomete.com;

    location / {
        proxy_pass http://127.0.0.1:3008;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

> The Go backend (`:8008`) is NOT exposed via Nginx — it's internal-only (LAN + Docker network). Only the frontend scoreboard is publicly accessible.

---

## 10. Rollback Procedure

```bash
# On homelab server — rollback to previous image
cd ~/platform

# Check available image tags
docker images ghcr.io/deerngo/deerngo-bot --format "{{.Tag}} {{.CreatedAt}}"

# Rollback: re-deploy with specific SHA tag
# Edit docker-compose.deerngo.yml to pin the known-good tag:
#   image: ghcr.io/deerngo/deerngo-bot:sha-abc1234
docker compose -f docker-compose.deerngo.yml up -d

# Verify
curl http://localhost:8008/api/v1/health
curl http://localhost:3008
```

> **Key principle:** Every deploy is reversible. The SHA-based image tag is the rollback target. Never delete old images from GHCR — let retention policy handle cleanup.

---

## 11. Pipeline Health

| Metric | Target | Measurement |
|--------|--------|------------|
| Pipeline success rate | > 95% | GitHub Actions insights |
| Build duration (total) | < 10 min | GitHub Actions timing |
| Deploy duration | < 5 min | SSH action timing |
| Time from commit to production | < 15 min | Push timestamp → deploy log |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[052_deployment_plan]] | Manual deployment procedures (fallback) |
| [[053_release_notes]] | Release details per version |
| [[032_BE_build_scripts]] | Backend Makefile + Dockerfile |
| [[032_FE_build_scripts]] | Frontend npm scripts + Dockerfile |
| [[034_SHARED_commit_messages_changelog]] | Conventional commits (input for release notes) |

---

> **Template Standard:** Based on SWEBOK v4, 12-Factor App
> **Usage:** The pipeline is *the single path to production*. If it's not in the pipeline, it doesn't ship.
