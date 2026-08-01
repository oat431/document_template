---
document_type: Coding Standards — Security
version: "0.2"
status: Draft
author: "QA Engineer / Dev"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [coding-standards, security, go, backend, owasp, members, privacy, easydonate]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Secure Coding Practices
  - OWASP Go Security Cheat Sheet
  - Effective Go
parent_project: "Deerngo Bot — VRM"
---

# Coding Standards — Security

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-bot` (backend)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> These rules complement the general backend coding standards. They define the security baseline for explicit member registration, donation ingestion, points, and the public scoreboard.

---

## 1. Input Validation

### 1.1 Validate at the Handler Layer

Validate JSON shape, required fields, length, encoding, and allowed values before service/repository calls.

For member registration:

- `youtube_user_id`: required, non-empty, maximum configured length.
- `youtube_handle`: required, non-empty after normalization, maximum 100 characters.
- Do not accept or persist a YouTube display name.
- Do not trust a handle typed in the chat message; only streamer.bot metadata is accepted.

For donations:

- `referenceNo`: required and bounded.
- donor name: required for matching; private.
- amount: positive, bounded decimal in expected currency.
- time: valid timezone-aware timestamp.
- reject unknown/invalid payloads according to verified provider contract.

### 1.2 Normalize Handles Consistently

```go
func normalizeHandle(value string) string {
    value = strings.TrimSpace(value)
    value = strings.TrimPrefix(value, "@")
    return strings.ToLower(strings.TrimSpace(value))
}
```

Apply the same function to the member handle and EasyDonate donor name. Do not add fuzzy matching, punctuation removal, or similarity thresholds in Phase 1.

### 1.3 Validate Path and Query Parameters

- `youtube_user_id` path values must be non-empty and bounded.
- `page >= 1`.
- `1 <= limit <= 100`; choose and document reject-vs-clamp behavior.
- Webhook path token is compared securely and never returned/logged.

---

## 2. SQL Injection and Database Integrity

### 2.1 Parameterized Queries Only

```go
query := `SELECT member_id, youtube_handle, total_points
          FROM members
          WHERE youtube_user_id = $1 AND status = 'active'`
row := db.QueryRowContext(ctx, query, userID)
```

Never interpolate IDs, handles, donor names, page, limit, or timestamps into SQL strings.

### 2.2 Enforce Active Uniqueness in PostgreSQL

Use partial unique indexes:

```sql
CREATE UNIQUE INDEX uk_members_active_user
    ON members(youtube_user_id) WHERE status = 'active';

CREATE UNIQUE INDEX uk_members_active_handle
    ON members(youtube_handle) WHERE status = 'active';
```

Treat uniqueness violations as a controlled `HANDLE_IN_USE`/conflict response, not as a generic success.

### 2.3 Transactional Changed-Handle Registration

When a same user registers a different handle:

1. Begin transaction.
2. Lock/find active row for `youtube_user_id`.
3. Check active-handle conflict.
4. Mark old row inactive without changing its points.
5. Insert new active row with 0 points.
6. Commit atomically.

If any step fails, rollback all steps. Do not leave two active rows.

### 2.4 Exactly-Once Point Application

Lock the donation row and apply points only when `points_applied_at IS NULL`.

Required conditions:

```sql
member.status = 'active'
AND member.youtube_handle = normalized_donor_name
AND donation_time >= member.registered_at
```

Update member total and donation marker in one transaction. A repeated webhook, API poll, worker retry, or concurrent matcher must not add points twice.

---

## 3. Webhook Security

### 3.1 Provider Contract First

Do not assume EasyDonate supports HMAC, `X-EasyDonate-Signature`, custom headers, or a particular payload field name. Verify the current official contract/dashboard/test event before implementing provider-specific verification.

### 3.2 Fallback if No Provider Signing/Auth Exists

Use all of:

- long unpredictable path token, stored as a deployment secret
- strict JSON/payload validation
- request body-size limit appropriate for the provider payload
- per-IP/provider rate limiting
- unique `referenceNo` idempotency
- safe error responses without payload/secret reflection

### 3.3 If Provider Signing Is Confirmed Later

Implement the exact provider scheme using:

- raw request body verification before parsing
- constant-time comparison (`hmac.Equal` where HMAC is actually specified)
- replay/timestamp checks if provider supports timestamps
- secret rotation and dual-key transition procedure
- tests for valid, invalid, missing, and replayed signatures

Do not add HMAC code merely because a previous document assumed it.

---

## 4. Secrets and Sensitive Data

### 4.1 Never Hardcode or Log Secrets

Sensitive values include:

- `DATABASE_URL` credentials
- `EASYDONATE_API_KEY`
- `EASYDONATE_WEBHOOK_PATH_TOKEN`
- Any provider signing secret if later confirmed
- OAuth/client secrets from historical YouTube work

Use environment/deployment secret storage. Never place secrets in source, examples, GitHub issues, meeting minutes, test fixtures, or ordinary chat.

### 4.2 Redact Logs

Safe to log:

```text
registration outcome, member_id, non-sensitive status, timestamp, route, error class
```

Do not log:

```text
API keys, path tokens, raw provider payloads, donor messages, display names, access/refresh tokens, database URLs
```

Use synthetic data for tests.

---

## 5. Public API Data Minimization

The scoreboard response allowlist is:

```text
rank
youtube_handle
total_points
pagination metadata
```

Never return:

- `youtube_user_id`
- YouTube display name
- raw donor name
- donation message/reference
- registration timestamp
- inactive/private/zero-point member data

The backend is authoritative for:

```sql
status = 'active'
AND public_visibility = TRUE
AND total_points > 0
```

The frontend must not request or reconstruct private fields.

---

## 6. CORS and HTTP Hardening

- Allow the configured scoreboard origin only on the public scoreboard route.
- Do not use wildcard CORS for internal member/visibility/points routes.
- Add security headers where applicable: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, and appropriate CSP.
- Disable debug/error stack traces in production.
- Set request timeouts and body limits.
- Rate-limit public and webhook routes.

---

## 7. Error Handling

- Return stable error codes (`VALIDATION_ERROR`, `MEMBER_NOT_REGISTERED`, `HANDLE_IN_USE`, `WEBHOOK_INVALID`, `RATE_LIMITED`, `INTERNAL_ERROR`).
- Do not include SQL errors, stack traces, provider secrets, internal addresses, or raw request bodies in client responses.
- Log internal context with redaction and correlation/request ID.
- Map backend failures to friendly streamer.bot messages.

---

## 8. Manual Point Correction

Until the admin console exists:

- Do not allow arbitrary public correction endpoints.
- Require owner/back-office database access with least privilege.
- Take/verify a backup before correction.
- Use a transaction.
- Record before/after totals, reason, operator, and time in `point_adjustment_notes`.
- Preserve the old inactive member and donation history; do not fabricate an automatic alias/match.
- Verify the active member's point query and scoreboard behavior after correction.

---

## 9. Testing Requirements

Every implementation must include tests for:

- normalization and maximum lengths
- missing/invalid member fields
- same-handle idempotency
- changed-handle transaction rollback
- active user/handle conflicts
- inactive-member exclusion
- exact donor matching and punctuation mismatch
- donation-time cutoff
- duplicate/concurrent point application
- invalid webhook path/payload
- provider 429/backoff
- public response allowlist
- secret and raw donor-data redaction
- SQL injection payloads
- CORS and rate limits

Run:

```bash
go test -race -coverprofile=coverage.out ./...
go vet ./...
govulncheck ./...
npm audit
```

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[061_security_test_report]] | Threat model and security tests |
| [[022_API_specification]] | Current API/security contract |
| [[023_database_schema_DDL]] | Data constraints and transactions |
| [[021_architecture_decision_records]] | Member/privacy/provider decisions |
| [[071_risk_register]] | Runtime risks |
| `https://github.com/oat431/deerngo-bot/issues/18` | Provider contract/security gate |
| `https://github.com/oat431/deerngo-bot/issues/19` | Manual correction procedure |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Secure Coding Practices, and Effective Go
> **Usage:** Mandatory security baseline for active Phase 1 code.
---
