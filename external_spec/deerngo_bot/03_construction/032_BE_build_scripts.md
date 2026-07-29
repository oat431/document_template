---
document_type: Build Scripts (Backend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [build-scripts, makefile, docker, go, backend]
standard_ref:
  - SWEBOK v4 — Construction
  - 12-Factor App (Build, release, run)
parent_project: "Deerngo Bot — VRM"
---

# Build Scripts — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Makefile

```makefile
.PHONY: build test lint run clean docker migrate

build:
	go build -o bin/deerngo-bot ./cmd/server

test:
	go test ./...

test-verbose:
	go test -v ./...

test-coverage:
	go test -cover ./... -coverprofile=coverage.out
	go tool cover -html=coverage.out -o coverage.html

lint:
	golangci-lint run

fmt:
	gofmt -w .
	goimports -w .

run: build
	./bin/deerngo-bot

clean:
	rm -rf bin/ coverage.out coverage.html

migrate-up:
	go run cmd/migrate/main.go up

migrate-down:
	go run cmd/migrate/main.go down

migrate-create:
	go run cmd/migrate/main.go create $(name)

docker-build:
	docker build -t deerngo-bot:latest .

docker-run:
	docker run -p 8008:8008 --env-file ../.env deerngo-bot:latest

deps:
	go mod download
	go mod tidy

check: fmt lint test build
```

---

## 2. Dockerfile (Multi-Stage)

```dockerfile
# ---- Stage 1: Build ----
FROM golang:1.24-alpine AS build
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o bin/deerngo-bot ./cmd/server

# ---- Stage 2: Runtime ----
FROM alpine:3.20
WORKDIR /app

RUN apk --no-cache add ca-certificates tzdata
RUN addgroup -S app && adduser -S app -G app

COPY --from=build /app/bin/deerngo-bot .
COPY --from=build /app/migrations ./migrations

EXPOSE 8008
USER app

ENTRYPOINT ["./deerngo-bot"]
```

---

## 3. Build Commands Summary

| Command | Purpose |
|---------|---------|
| `make build` | Compile → `bin/deerngo-bot` |
| `make test` | Run all tests |
| `make test-coverage` | Coverage → `coverage.html` |
| `make lint` | golangci-lint |
| `make fmt` | Auto-format |
| `make run` | Build + run |
| `make migrate-up` | Apply migrations |
| `make migrate-down` | Rollback migration |
| `make docker-build` | Build Docker image |
| `make docker-run` | Run container |
| `make check` | fmt + lint + test + build |

---

## Related Documents

| Document | Path |
|----------|------|
| README | `03_construction/031_BE_README.md` |
| Dependency Manifest | `03_construction/035_BE_dependency_manifest.md` |
| DB Schema | `02_design/023_BE_database_schema_DDL.md` |

---

> **Template Standard:** Based on SWEBOK v4, 12-Factor App
