---
document_type: Coding Standards (Backend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [coding-standards, go, backend, fiber, sqlx, swebok]
standard_ref:
  - SWEBOK v4 — Construction
  - Effective Go
  - Go Code Review Comments
parent_project: "Deerngo Bot — VRM"
---

# Coding Standards — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

> Coding standards for the Deerngo Bot Go backend. Standards are **enforced** — if it doesn't pass `golangci-lint` and `go test`, it doesn't merge. Dev agents should follow these rules to produce consistent, readable code.

---

## 2. Language Standards

| Aspect | Standard | Tool |
|--------|---------|------|
| Go Version | 1.24+ | `go.mod` |
| Style Guide | [Effective Go](https://go.dev/doc/effective_go) | `gofmt`, `golangci-lint` |
| Formatting | `gofmt` default (tabs, no semicolons) | `gofmt -w .` |
| Import Grouping | stdlib / external / internal (blank line between) | `goimports` |

---

## 3. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Packages | lowercase, single word | `package service`, `package handler` |
| Exported | PascalCase | `func NewSubscriberService()`, `type DonationRepository` |
| Unexported | camelCase | `func normalizeHandle()`, `var maxRetries` |
| Acronyms | All caps or all lower | `HTTPClient` or `httpClient` (be consistent) |
| Interfaces | -er suffix when possible | `Reader`, `SubscriberFinder` |
| Files | snake_case | `subscriber_service.go`, `youtube_poller.go` |
| Test Files | `_test.go` suffix | `subscriber_service_test.go` |
| Test Functions | `TestXxx` | `TestNormalizeHandle`, `TestUpsertSubscriber` |

---

## 4. Project Layout

```
deerngo-bot/
├── cmd/
│   ├── server/main.go          # Entry point — Fiber app, scheduler startup
│   └── migrate/main.go         # Migration runner
├── internal/
│   ├── config/
│   │   └── config.go           # env var loading, validation
│   ├── handler/
│   │   ├── subscriber.go       # POST /api/v1/subscribers
│   │   ├── points.go           # GET /api/v1/points/{handle}
│   │   ├── scoreboard.go       # GET /api/v1/scoreboard
│   │   ├── webhook.go          # POST /api/v1/webhooks/easydonate
│   │   └── health.go           # GET /api/v1/health
│   ├── service/
│   │   ├── subscriber.go       # Subscriber business logic
│   │   ├── donation.go         # Donation business logic
│   │   ├── points.go           # Points calculation
│   │   └── matcher.go          # Name matching engine
│   ├── repository/
│   │   ├── subscriber.go       # Subscriber SQL queries (sqlx)
│   │   ├── donation.go         # Donation SQL queries
│   │   ├── points.go           # Points SQL queries
│   │   └── oauth.go            # OAuth token queries
│   ├── scheduler/
│   │   ├── youtube.go          # YouTube API poller (15 min)
│   │   ├── easydonate.go       # EasyDonate sync (5 min)
│   │   └── matcher.go          # Name matcher (2 min)
│   ├── client/
│   │   ├── youtube.go          # YouTube Data API client
│   │   └── easydonate.go       # EasyDonate API client
│   └── middleware/
│       ├── cors.go             # CORS config
│       ├── ratelimit.go        # Rate limiting
│       └── logger.go           # Request logging
├── migrations/
│   ├── 001_initial_schema.up.sql
│   └── 001_initial_schema.down.sql
├── go.mod
├── go.sum
├── Makefile
└── Dockerfile
```

---

## 5. Code Patterns

### 5.1 Handler Pattern (Fiber v3)

```go
// ✅ Good — clean handler, delegates to service
func (h *SubscriberHandler) Create(c fiber.Ctx) error {
    var req CreateSubscriberRequest
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "Invalid request body",
        })
    }

    sub, err := h.service.Create(c.Context(), req)
    if err != nil {
        return handleServiceError(c, err)
    }

    status := fiber.StatusCreated
    if sub.Upserted {
        status = fiber.StatusOK
    }
    return c.Status(status).JSON(SuccessResponse{Data: sub})
}

// ❌ Bad — business logic in handler, no error handling
func Create(c fiber.Ctx) error {
    body := c.Body()
    db.Exec("INSERT INTO subscribers ...", body)
    return c.SendString("ok")
}
```

### 5.2 Service Pattern

```go
// ✅ Good — service layer with dependency injection
type SubscriberService struct {
    repo repository.SubscriberRepository
}

func NewSubscriberService(repo repository.SubscriberRepository) *SubscriberService {
    return &SubscriberService{repo: repo}
}

func (s *SubscriberService) Create(ctx context.Context, req CreateSubscriberRequest) (*Subscriber, error) {
    handle := normalizeHandle(req.YouTubeHandle)
    if handle == "" {
        return nil, ErrInvalidHandle
    }

    sub := &Subscriber{
        YouTubeHandle: handle,
        DisplayName:   req.DisplayName,
        SubscribedAt:  req.SubscribedAt,
        Source:        req.Source,
    }

    result, err := s.repo.Upsert(ctx, sub)
    if err != nil {
        return nil, fmt.Errorf("upserting subscriber %s: %w", handle, err)
    }
    return result, nil
}
```

### 5.3 Repository Pattern (sqlx)

```go
// ✅ Good — sqlx with named parameters, struct scanning
func (r *subscriberRepo) Upsert(ctx context.Context, sub *Subscriber) (*Subscriber, error) {
    query := `
        INSERT INTO subscribers (youtube_handle, display_name, subscribed_at, source)
        VALUES (:youtube_handle, :display_name, :subscribed_at, :source)
        ON CONFLICT (youtube_handle) DO UPDATE SET
            display_name = EXCLUDED.display_name,
            source = EXCLUDED.source
        RETURNING id, created_at, updated_at`

    result := &Subscriber{}
    rows, err := r.db.NamedQueryContext(ctx, query, sub)
    if err != nil {
        return nil, fmt.Errorf("upserting subscriber: %w", err)
    }
    defer rows.Close()

    if rows.Next() {
        if err := rows.StructScan(result); err != nil {
            return nil, fmt.Errorf("scanning subscriber: %w", err)
        }
    }
    return result, nil
}
```

### 5.4 Error Handling

```go
// ✅ Good — wrap errors with context, sentinel errors for known cases
var (
    ErrSubscriberNotFound = errors.New("subscriber not found")
    ErrInvalidHandle      = errors.New("invalid youtube handle")
    ErrDuplicateDonation  = errors.New("donation already processed")
)

func (s *SubscriberService) GetByHandle(ctx context.Context, handle string) (*Subscriber, error) {
    sub, err := s.repo.GetByHandle(ctx, normalizeHandle(handle))
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrSubscriberNotFound
        }
        return nil, fmt.Errorf("getting subscriber %s: %w", handle, err)
    }
    return sub, nil
}

// ❌ Bad — swallowed errors, no context
func Get(handle string) *Subscriber {
    sub, _ := repo.Get(handle)
    return sub
}
```

### 5.5 Scheduler Pattern

```go
// ✅ Good — clean scheduler with context cancellation
func (s *YouTubePoller) Start(ctx context.Context) {
    ticker := time.NewTicker(15 * time.Minute)
    defer ticker.Stop()

    // Run immediately on start
    s.poll(ctx)

    for {
        select {
        case <-ctx.Done():
            log.Info("YouTube poller stopped")
            return
        case <-ticker.C:
            s.poll(ctx)
        }
    }
}

func (s *YouTubePoller) poll(ctx context.Context) {
    if err := s.pollOnce(ctx); err != nil {
        log.Error("YouTube poll failed", "error", err)
    }
}
```

---

## 6. Testing Standards

| Rule | Standard |
|------|----------|
| File naming | `*_test.go` next to the file it tests |
| Function naming | `TestXxx(t *testing.T)` |
| Table-driven tests | Use for multiple input/output cases |
| Assertions | `testify/assert` or `testify/require` |
| Mocking | Interface-based mocks (mockgen or hand-written) |
| Coverage | ≥ 80% on service layer |

```go
// ✅ Good — table-driven test
func TestNormalizeHandle(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
    }{
        {"strip @", "@viewer1", "viewer1"},
        {"lowercase", "Viewer1", "viewer1"},
        {"trim spaces", " viewer1 ", "viewer1"},
        {"empty", "", ""},
        {"anonymous", "anonymous", ""},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            assert.Equal(t, tt.expected, normalizeHandle(tt.input))
        })
    }
}
```

---

## 7. Fiber v3 Specific Rules

| Rule | Rationale |
|------|----------|
| Use `fiber.Ctx` handlers | Standard pattern for this project |
| Use `c.Bind().JSON(&req)` for request binding | Built-in validation |
| Use `c.JSON()` for responses | Consistent JSON format |
| Middleware in `internal/middleware/` | Separated from handlers |
| Use `fiber.Map{}` for quick responses | Avoids creating response structs for simple cases |

---

## 8. sqlx Specific Rules

| Rule | Rationale |
|------|----------|
| Named parameters (`:field`) | Readable, less error-prone than positional |
| `StructScan` for row mapping | Automatic field mapping |
| `NamedQueryContext` for INSERT/UPDATE | Supports named params |
| `QueryxContext` for SELECT | Returns `*sqlx.Rows` |
| Always `defer rows.Close()` | Prevent connection leaks |
| Use `context.Context` everywhere | Cancellation, timeouts |

---

## 9. Universal Rules

| Rule | Rationale |
|------|----------|
| No hardcoded secrets | Env vars only |
| No commented-out code | Delete it — git history preserves |
| No magic numbers | Extract to named constants |
| Functions do one thing | If you need "and" in the name, split it |
| Early returns | Flatten nested ifs |
| Log at the right level | INFO for business events, ERROR for failures |
| Meaningful names | `createSubscriber()` not `cs()` |

---

## 10. Linting & CI

| Tool | Command | Config |
|------|---------|--------|
| `gofmt` | `gofmt -w .` | Default (tabs) |
| `goimports` | `goimports -w .` | Group: stdlib / external / internal |
| `golangci-lint` | `golangci-lint run` | `.golangci.yml` |
| `go test` | `go test ./...` | — |
| `govulncheck` | `govulncheck ./...` | Vulnerability scan |

All linting **must pass** before merge. Configure as pre-commit hook.

---

## Related Documents

| Document | Path |
|----------|------|
| README | `03_construction/031_BE_README.md` |
| Build Scripts | `03_construction/032_BE_build_scripts.md` |
| Dependency Manifest | `03_construction/033_BE_dependency_manifest.md` |
| API Specification | `02_design/022_SHARED_API_specification.md` |
| DB Schema | `02_design/023_BE_database_schema_DDL.md` |

---

> **Template Standard:** Based on SWEBOK v4, Effective Go
> **Usage:** Dev agents should follow these patterns exactly. Standards are enforced in CI.
