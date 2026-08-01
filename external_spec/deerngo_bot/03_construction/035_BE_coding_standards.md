---
document_type: Coding Standards (Backend)
version: "0.2"
status: Draft
author: "SA / Dev"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "BE"
classification: "Internal"
tags: [coding-standards, go, backend, fiber, sqlx, swebok, members, points]
standard_ref:
  - SWEBOK v4 — Construction
  - Effective Go
  - Go Code Review Comments
parent_project: "Deerngo Bot — VRM"
---

# Coding Standards — Deerngo Bot Backend (Go)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> Standards are enforced — if it does not pass formatting, lint, tests, and security checks, it does not merge.

## 1. Language Standards

| Aspect | Standard | Tool |
|--------|---------|------|
| Go Version | 1.25+ or repository-approved version | `go.mod` |
| Style | Effective Go | `gofmt`, `golangci-lint` |
| Imports | stdlib / external / internal groups | `goimports` |
| Errors | Wrap with context; sentinel/domain errors for expected API mapping | `errors`, `fmt.Errorf` |

## 2. Naming and Layout

| Element | Convention | Example |
|---------|-----------|---------|
| Packages | lowercase single word | `service`, `repository` |
| Exported | PascalCase | `MemberService` |
| Unexported | camelCase | `normalizeHandle` |
| Files | snake_case | `member_service.go` |
| Tests | `_test.go` | `member_service_test.go` |
| Test functions | `TestXxx` | `TestNormalizeHandle` |

```text
deerngo-bot/
├── cmd/
│   ├── server/main.go
│   └── migrate/main.go
├── internal/
│   ├── config/                # environment and provider config
│   ├── handler/               # member/visibility/points/scoreboard/webhook
│   ├── service/               # registration, donation, matcher, points
│   ├── repository/            # sqlx queries and transactions
│   ├── scheduler/             # EasyDonate fallback sync only
│   ├── client/                # EasyDonate API client
│   └── middleware/             # CORS, rate limit, body limit, logging
├── migrations/
├── go.mod
├── go.sum
├── Makefile
└── Dockerfile
```

## 3. Domain Rules

- `members.member_id` is the record primary key.
- `youtube_user_id` is the actual streamer.bot identity and unique among active members.
- `youtube_handle` is normalized and unique among active members.
- New member: active, public, total points 0.
- Same-handle re-registration: idempotent.
- Changed handle: old inactive/points preserved; new active/points 0; transaction required.
- Inactive members are excluded from normal points/matching/scoreboard behavior.
- No display-name field.
- Exact normalized matching only.

### Normalization

```go
func normalizeHandle(value string) string {
    value = strings.TrimSpace(value)
    value = strings.TrimPrefix(value, "@")
    return strings.ToLower(strings.TrimSpace(value))
}
```

Do not remove punctuation or use fuzzy similarity in Phase 1.

## 4. Fiber Handler Pattern

```go
func (h *MemberHandler) Register(c fiber.Ctx) error {
    var req RegisterMemberRequest
    if err := c.Bind().JSON(&req); err != nil {
        return writeError(c, fiber.StatusBadRequest, "VALIDATION_ERROR", "Invalid request body")
    }
    result, err := h.service.Register(c.Context(), req)
    if err != nil {
        return writeServiceError(c, err)
    }
    return c.Status(result.HTTPStatus).JSON(result.Response)
}
```

Handlers validate and map errors. Business rules belong in services; SQL belongs in repositories.

## 5. Service and Transaction Rules

### Registration

- Normalize before lookup.
- Check active user and active handle conflicts.
- Use one transaction for old-row inactivation/new-row creation.
- Never transfer points automatically.

### Donation/Points

- Store donations by unique provider `reference_no`.
- Normalize donor and handle identically.
- Require active member and `donation_time >= registered_at`.
- Lock/check `points_applied_at` before incrementing `members.total_points`.
- Update member summary and donation marker in the same transaction.

### Public Projection

Only query/serialize:

```text
rank, youtube_handle, total_points, safe pagination metadata
```

Never accidentally serialize domain structs containing private fields.

## 6. Repository Pattern

Use parameterized queries:

```go
const findActiveMember = `
    SELECT member_id, youtube_user_id, youtube_handle, status,
           public_visibility, registered_at, total_points
    FROM members
    WHERE youtube_user_id = $1 AND status = 'active'`
```

Use `QueryRowxContext`/`GetContext`, close rows, propagate context, and use explicit transactions for multi-row changes.

## 7. EasyDonate Provider Rules

- Verify the actual provider payload/auth contract before final implementation.
- Do not assume HMAC or `X-EasyDonate-Signature`.
- If no provider signing/custom auth exists, protect with path token, strict validation, body limit, rate limit, and reference idempotency.
- API key is backend-only.
- Raw donor names/messages are private and redacted from logs.

## 8. Error Mapping

| Domain Error | HTTP | Chat Mapping |
|--------------|-----:|--------------|
| invalid request | 400 | Friendly invalid/unavailable message |
| active handle conflict | 409 | Handle already in use |
| no active member | 404 | Ask viewer to register |
| invalid webhook | 401/403 | Provider retry/error; no internal details |
| duplicate reference | 200/duplicate result | No duplicate points |
| provider unavailable | 5xx/internal | Retry later |
| rate limited | 429 | Backoff; no sensitive detail |

## 9. Testing Standards

- Table-driven unit tests for normalization/bands.
- Integration tests with isolated PostgreSQL.
- Concurrency tests for registration and points.
- Webhook/provider tests using synthetic payloads.
- Public response allowlist tests.
- `go test -race`, `go vet`, `golangci-lint`, `govulncheck` before merge.
- No real credentials or donor data in fixtures.

## 10. Observability

Log request ID, route, status, duration, safe domain event, and error class. Redact:

```text
API keys, path tokens, DB URLs, display names, donor names/messages, provider payloads, OAuth tokens
```

## Related Documents

| Document | Path |
|----------|------|
| Security standards | `06_security/062_coding_standards_security.md` |
| Backend README | `03_construction/031_BE_README.md` |
| API Specification | `02_design/022_API_specification.md` |
| Database Schema | `02_design/023_database_schema_DDL.md` |
| Test Plan | `04_testing/041_test_plan.md` |

---

> **Template Standard:** Based on SWEBOK v4 and Effective Go
> **Usage:** Mandatory backend construction standard for the revised member-based MVP.
---
