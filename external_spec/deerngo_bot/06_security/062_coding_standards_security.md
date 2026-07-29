---
document_type: Coding Standards — Security
version: "0.1"
status: Draft
author: "QA Engineer"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [coding-standards, security, go, backend, owasp, swebok, vrm, deerngo-bot]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Secure Coding Practices
  - OWASP Go Security Cheat Sheet
  - Effective Go
parent_project: "Deerngo Bot — VRM"
---

# Coding Standards — Security

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

Security-specific coding standards for the Deerngo Bot Go backend. These rules **complement** the general coding standards (`035_BE_coding_standards.md`) — they focus exclusively on security patterns. Dev agents must follow both.

---

## 2. Input Validation

### 2.1 Rule: Validate All Input at the Handler Layer

```go
// ✅ Good — validate before passing to service
func (h *SubscriberHandler) Create(c fiber.Ctx) error {
    var req CreateSubscriberRequest
    if err := c.Bind().JSON(&req); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "Invalid request body",
        })
    }

    // Validate required fields
    if req.YouTubeHandle == "" {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "youtube_handle is required",
            Details: []FieldError{{Field: "youtube_handle", Message: "This field is required"}},
        })
    }

    // Validate field lengths
    if len(req.YouTubeHandle) > 100 {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "youtube_handle must be 100 characters or less",
        })
    }

    // Validate format
    if req.Source != "streamer_bot" {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "source must be 'streamer_bot'",
        })
    }

    return h.service.Create(c.Context(), req)
}

// ❌ Bad — no validation, pass raw input to service/DB
func (h *SubscriberHandler) Create(c fiber.Ctx) error {
    body := c.Body()
    h.repo.Insert(body)
    return c.SendStatus(fiber.StatusCreated)
}
```

### 2.2 Rule: Validate Path Parameters

```go
// ✅ Good — validate path parameter
func (h *PointsHandler) GetByHandle(c fiber.Ctx) error {
    handle := c.Params("handle")
    if handle == "" {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "handle is required",
        })
    }

    // Normalize: strip @, lowercase, trim
    normalized := normalizeHandle(handle)
    if normalized == "" {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "VALIDATION_ERROR",
            Message: "invalid handle",
        })
    }

    return h.service.GetPoints(c.Context(), normalized)
}
```

### 2.3 Rule: Validate Query Parameters

```go
// ✅ Good — validate and clamp pagination parameters
func (h *ScoreboardHandler) List(c fiber.Ctx) error {
    page := c.QueryInt("page", 1)
    limit := c.QueryInt("limit", 50)

    if page < 1 {
        page = 1
    }
    if limit < 1 {
        limit = 1
    }
    if limit > 100 {
        limit = 100  // Clamp to max per API spec
    }

    return h.service.GetScoreboard(c.Context(), page, limit)
}
```

---

## 3. SQL Injection Prevention

### 3.1 Rule: Always Use Named Parameters with sqlx

```go
// ✅ Good — named parameter, no string concatenation
func (r *subscriberRepo) GetByHandle(ctx context.Context, handle string) (*Subscriber, error) {
    query := `SELECT * FROM subscribers WHERE youtube_handle = :handle`
    rows, err := r.db.NamedQueryContext(ctx, query, map[string]interface{}{"handle": handle})
    // ...
}

// ❌ BAD — string concatenation = SQL injection
func (r *subscriberRepo) GetByHandle(handle string) *Subscriber {
    query := fmt.Sprintf("SELECT * FROM subscribers WHERE youtube_handle = '%s'", handle)
    rows := r.db.Query(query)
    // ...
}
```

### 3.2 Rule: Never Use fmt.Sprintf for SQL Queries

```go
// ❌ NEVER DO THIS
query := fmt.Sprintf("SELECT * FROM subscribers WHERE youtube_handle = '%s'", handle)

// ✅ ALWAYS DO THIS
query := `SELECT * FROM subscribers WHERE youtube_handle = :handle`
```

### 3.3 Rule: Use Parameterized Queries for pg_trgm

```go
// ✅ Good — parameterized similarity query
func (r *donationRepo) FindMatches(ctx context.Context, donorName string, threshold float64) ([]MatchResult, error) {
    query := `
        SELECT youtube_handle, similarity(youtube_handle, $1) AS score
        FROM subscribers
        WHERE similarity(youtube_handle, $1) > $2
        ORDER BY score DESC
        LIMIT 2`
    rows, err := r.db.QueryxContext(ctx, query, donorName, threshold)
    // ...
}

// ❌ Bad — interpolated threshold
query := fmt.Sprintf("... WHERE similarity(youtube_handle, '%s') > %f", donorName, threshold)
```

---

## 4. Webhook Security (HMAC-SHA256)

### 4.1 Rule: Verify HMAC Signature on Every Webhook Request

```go
// ✅ Good — HMAC-SHA256 verification with constant-time comparison
import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
)

func (h *WebhookHandler) VerifySignature(c fiber.Ctx) error {
    // 1. Get signature from header
    signature := c.Get("X-EasyDonate-Signature")
    if signature == "" {
        return c.Status(fiber.StatusUnauthorized).JSON(ErrorResponse{
            Code:    "WEBHOOK_INVALID",
            Message: "Missing webhook signature",
        })
    }

    // 2. Compute expected signature
    secret := os.Getenv("EASYDONATE_WEBHOOK_SECRET")
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(c.Body())
    expected := hex.EncodeToString(mac.Sum(nil))

    // 3. Constant-time comparison (prevents timing attacks)
    if !hmac.Equal([]byte(signature), []byte(expected)) {
        return c.Status(fiber.StatusUnauthorized).JSON(ErrorResponse{
            Code:    "WEBHOOK_INVALID",
            Message: "Invalid webhook signature",
        })
    }

    return h.processWebhook(c)
}
```

### 4.2 Rule: Use Constant-Time Comparison

```go
// ✅ Good — hmac.Equal is constant-time
if !hmac.Equal([]byte(signature), []byte(expected)) { ... }

// ❌ BAD — == operator is NOT constant-time, vulnerable to timing attacks
if signature != expected { ... }
```

### 4.3 Rule: Never Log the Secret Key

```go
// ✅ Good — log that verification happened, not the key
log.Info("Webhook signature verified", "easydonate_id", donation.ID)

// ❌ BAD — logging the secret
log.Info("Webhook secret", "secret", os.Getenv("EASYDONATE_WEBHOOK_SECRET"))
log.Debug("HMAC key", "key", secret)
```

---

## 5. Secret Management

### 5.1 Rule: Never Hardcode Secrets

```go
// ✅ Good — read from environment variable
secret := os.Getenv("EASYDONATE_WEBHOOK_SECRET")
if secret == "" {
    log.Fatal("EASYDONATE_WEBHOOK_SECRET not set")
}

// ❌ BAD — hardcoded secret
secret := "my-super-secret-key-123"
```

### 5.2 Rule: Validate Secrets at Startup

```go
// ✅ Good — fail fast if secrets missing
func LoadConfig() (*Config, error) {
    cfg := &Config{
        DBURL:      os.Getenv("DATABASE_URL"),
        WebhookSecret: os.Getenv("EASYDONATE_WEBHOOK_SECRET"),
        YouTubeOAuthPath: os.Getenv("YOUTUBE_OAUTH_PATH"),
    }

    if cfg.DBURL == "" {
        return nil, errors.New("DATABASE_URL is required")
    }
    if cfg.WebhookSecret == "" {
        return nil, errors.New("EASYDONATE_WEBHOOK_SECRET is required")
    }

    return cfg, nil
}
```

### 5.3 Rule: Never Log Secrets or Tokens

```go
// ✅ Good — log redacted
log.Info("YouTube OAuth token refreshed", "channel_id", channelID)

// ❌ BAD — logging tokens
log.Info("YouTube OAuth token", "token", accessToken)
log.Debug("DB connection", "url", cfg.DBURL)
```

---

## 6. Error Handling (Security)

### 6.1 Rule: Never Expose Internal Details in Error Responses

```go
// ✅ Good — generic error message for clients
func handleServiceError(c fiber.Ctx, err error) error {
    switch {
    case errors.Is(err, ErrSubscriberNotFound):
        return c.Status(fiber.StatusNotFound).JSON(ErrorResponse{
            Code:    "NOT_FOUND",
            Message: "Subscriber not found",
        })
    default:
        // Log full error internally
        log.Error("Internal error", "error", err)

        // Return generic message to client
        return c.Status(fiber.StatusInternalServerError).JSON(ErrorResponse{
            Code:    "INTERNAL_ERROR",
            Message: "An internal error occurred",
        })
    }
}

// ❌ BAD — exposing stack trace or DB error to client
return c.Status(500).JSON(fiber.Map{
    "error": err.Error(),  // May contain DB connection string, table names, etc.
})
```

### 6.2 Rule: Log Errors Internally, Return Generic Messages

| Log Level | What to Log | What to Return to Client |
|-----------|-------------|------------------------|
| `log.Error` | Full error with stack trace, request ID | `"An internal error occurred"` |
| `log.Warn` | Validation failures, rate limit hits | Specific validation message |
| `log.Info` | Business events (subscriber created, donation matched) | Success response with data |

---

## 7. Rate Limiting

### 7.1 Rule: Apply Rate Limiting to All Endpoints

```go
// ✅ Good — Fiber rate limiter middleware
app := fiber.New()

// Global rate limit: 100 req/min per IP
app.Use(limiter.New(limiter.Config{
    Max:        100,
    Expiration: 1 * time.Minute,
    KeyGenerator: func(c fiber.Ctx) string {
        return c.IP()
    },
    LimitReached: func(c fiber.Ctx) error {
        return c.Status(fiber.StatusTooManyRequests).JSON(ErrorResponse{
            Code:    "RATE_LIMITED",
            Message: "Too many requests",
        })
    },
}))
```

### 7.2 Rule: Separate Rate Limit for Webhook

```go
// ✅ Good — webhook has higher limit (200/min)
webhook := app.Group("/api/v1/webhooks")
webhook.Use(limiter.New(limiter.Config{
    Max:        200,
    Expiration: 1 * time.Minute,
}))
webhook.Post("/easydonate", h.webhookHandler.Handle)
```

---

## 8. CORS Configuration

### 8.1 Rule: Restrict CORS to Known Origins

```go
// ✅ Good — explicit allowed origins
app.Use(cors.New(cors.Config{
    AllowOrigins: "https://deerngo-viewer-score.panomete.com",
    AllowMethods: "GET",
    AllowHeaders: "Content-Type",
    MaxAge:       3600,
}))

// ❌ BAD — allow all origins
app.Use(cors.New(cors.Config{
    AllowOrigins: "*",
}))
```

### 8.2 Rule: Restrict Methods per Endpoint

| Endpoint | Allowed Methods | CORS Origin |
|----------|:--------------:|-------------|
| `/api/v1/scoreboard` | `GET` | Scoreboard frontend URL |
| `/api/v1/subscribers` | `POST` | None (LAN only) |
| `/api/v1/points/*` | `GET` | None (LAN only) |
| `/api/v1/webhooks/*` | `POST` | EasyDonate (if applicable) |

---

## 9. Security Headers

### 9.1 Rule: Set Security Headers via Middleware

```go
// ✅ Good — security headers middleware
func SecurityHeaders() fiber.Handler {
    return func(c fiber.Ctx) error {
        c.Set("X-Content-Type-Options", "nosniff")
        c.Set("X-Frame-Options", "DENY")
        c.Set("X-XSS-Protection", "1; mode=block")
        c.Set("Referrer-Policy", "strict-origin-when-cross-origin")
        c.Set("Content-Security-Policy", "default-src 'self'")
        return c.Next()
    }
}

app.Use(SecurityHeaders())
```

### 9.2 Required Headers

| Header | Value | Purpose |
|--------|-------|---------|
| `X-Content-Type-Options` | `nosniff` | Prevent MIME type sniffing |
| `X-Frame-Options` | `DENY` | Prevent clickjacking |
| `X-XSS-Protection` | `1; mode=block` | XSS filter (legacy browsers) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer leakage |
| `Content-Security-Policy` | `default-src 'self'` | Prevent inline script injection |

---

## 10. Database Security

### 10.1 Rule: Use Least-Privilege Database User

```sql
-- ✅ Good — dedicated user with minimal permissions
CREATE USER deerngo_bot WITH PASSWORD '...';
GRANT CONNECT ON DATABASE deerngo TO deerngo_bot;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO deerngo_bot;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO deerngo_bot;
-- No DELETE, no DROP, no ALTER

-- ❌ BAD — superuser access
GRANT ALL PRIVILEGES ON DATABASE deerngo TO deerngo_bot;
```

### 10.2 Rule: Never Expose Database Errors to Clients

```go
// ✅ Good — wrap DB errors
if err != nil {
    return nil, fmt.Errorf("querying subscriber %s: %w", handle, err)
    // Client sees: "An internal error occurred"
    // Log sees: "querying subscriber @viewer1: connection refused"
}

// ❌ BAD — raw DB error to client
if err != nil {
    return c.Status(500).JSON(fiber.Map{"error": err.Error()})
    // Client sees: "pq: relation "subscribers" does not exist"
}
```

---

## 11. Dependency Security

### 11.1 Rule: Run govulncheck in CI

```yaml
# ✅ Add to CI pipeline
- name: Security scan
  run: govulncheck ./...
```

### 11.2 Rule: Pin Dependency Versions

```
# ✅ Good — pinned version in go.mod
require (
    github.com/gofiber/fiber/v3 v3.0.0-beta.2
    github.com/jmoiron/sqlx v1.4.0
)

# ❌ BAD — unpinned
require (
    github.com/gofiber/fiber/v3 latest
)
```

---

## 12. Security Checklist (PR Review)

Every PR touching security-sensitive code must verify:

| # | Check | File Pattern |
|---|-------|-------------|
| 1 | No hardcoded secrets | `*.go` |
| 2 | No `fmt.Sprintf` in SQL queries | `internal/repository/*.go` |
| 3 | All input validated at handler layer | `internal/handler/*.go` |
| 4 | HMAC verification on webhook | `internal/handler/webhook.go` |
| 5 | `hmac.Equal` used (not `==`) | `internal/handler/webhook.go` |
| 6 | Error responses don't expose internals | `internal/handler/*.go` |
| 7 | No secrets in log statements | `*.go` |
| 8 | Rate limiting configured | `internal/middleware/ratelimit.go` |
| 9 | CORS restricted to known origins | `internal/middleware/cors.go` |
| 10 | Security headers set | `internal/middleware/*.go` |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[035_BE_coding_standards]] | General Go coding standards (complement) |
| [[061_security_test_report]] | Security findings these rules address |
| [[022_API_specification]] | API contracts these rules implement |
| [[021_architecture_decision_records]] | ADR-012 (HMAC), ADR-009 (Fiber) |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Secure Coding Practices, OWASP Go Security Cheat Sheet
> **Usage:** Security standards are *enforced* in code review. If a PR violates these rules, it doesn't merge.
