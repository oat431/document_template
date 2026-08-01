---
document_type: API Specification
version: "0.2"
status: Draft
author: "PO / SA"
created: "2026-07-29"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
tech_lead: "SA / Dev"
classification: "Internal"
tags: [api-specification, openapi, rest, members, vrm, fiber, easydonate, privacy]
standard_ref:
  - SWEBOK v4 — Design
  - OpenAPI Specification 3.0
parent_project: "Deerngo Bot — VRM"
---

# API Specification

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** The YouTube subscriber-polling API and OAuth flow are removed from the active Phase 1 MVP. Members are created explicitly from streamer.bot live-chat identity.

---

## 1. Purpose

This document defines the Go backend API contract for member registration, visibility controls, points queries, EasyDonate ingestion, and the public scoreboard.

## 2. API Overview

| Field | Detail |
|-------|--------|
| LAN Base URL | `http://192.168.1.121:8008` — streamer.bot calls this address |
| Docker Internal URL | `http://deerngo-bot:8008` — frontend-to-backend communication |
| Public URL | Cloudflare Tunnel — scoreboard and configured EasyDonate webhook route only |
| Protocol | HTTP internally; HTTPS through Cloudflare |
| Format | JSON |
| Authentication | No general auth in Phase 1; streamer.bot identity is supplied by the integration |
| Versioning | `/api/v1/` |
| Default Rate Limit | 100 requests/minute per IP |
| Webhook Rate Limit | 200 requests/minute if provider traffic requires a separate tier |

## 3. Data and Identity Rules

- `youtube_user_id` comes from streamer.bot's actual chat author identity. Never trust a handle typed in the chat message.
- `youtube_handle` is normalized before storage and matching: trim whitespace, remove one leading `@`, lowercase.
- `members.member_id` is the primary key.
- Only one active member may exist for a YouTube user ID.
- Only one active member may own a normalized handle.
- Re-registering with the same active identity is idempotent.
- Registering a changed handle deactivates the old member and creates a new active member with 0 points.
- No YouTube display name is stored or returned.
- `public_visibility` defaults to `true`; private members remain eligible for points but are excluded from public responses.

## 4. Common Response Formats

### Success

```json
{
  "data": { "...": "..." },
  "meta": {
    "timestamp": "2026-08-02T10:00:00Z"
  }
}
```

### Error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "youtube_handle is required",
    "details": [
      { "field": "youtube_handle", "message": "This field is required" }
    ]
  }
}
```

### Error Codes

| Code | HTTP | Meaning |
|------|-----:|---------|
| `VALIDATION_ERROR` | 400 | Invalid or missing input |
| `MEMBER_NOT_REGISTERED` | 404 | No active member for the supplied identity |
| `HANDLE_IN_USE` | 409 | Handle belongs to another active user |
| `CONFLICT` | 409 | State conflict |
| `RATE_LIMITED` | 429 | Too many requests |
| `WEBHOOK_INVALID` | 401/403 | Invalid path token or provider-confirmed auth failure |
| `INTERNAL_ERROR` | 500 | Unexpected backend error |

---

## 5. API Endpoints

### 5.1 POST `/api/v1/members/register` — Register a Viewer

Called by streamer.bot for `:deer: register`.

| Field | Detail |
|-------|--------|
| Auth | Internal LAN integration; no typed handle trusted |
| Called By | streamer.bot HTTP Request action |
| Rate Limit | 100/min |

**Request**

```json
{
  "youtube_user_id": "UC-viewer-123",
  "youtube_handle": "@Viewer123"
}
```

**Validation**

| Field | Rule |
|-------|------|
| `youtube_user_id` | Required, non-empty, provider user/channel identifier |
| `youtube_handle` | Required, 1–100 characters before normalization |
| display name | Not accepted and not stored |
| command text handle | Ignored; only streamer.bot identity is accepted |

**Response — New Member (201)**

```json
{
  "data": {
    "member_id": "uuid",
    "youtube_user_id": "UC-viewer-123",
    "youtube_handle": "viewer123",
    "status": "active",
    "public_visibility": true,
    "total_points": 0,
    "registered_at": "2026-08-02T10:00:00Z"
  },
  "meta": {
    "created": true
  }
}
```

**Response — Same Active Member (200)**

```json
{
  "data": {
    "member_id": "uuid",
    "youtube_user_id": "UC-viewer-123",
    "youtube_handle": "viewer123",
    "status": "active",
    "public_visibility": true,
    "total_points": 563
  },
  "meta": {
    "already_registered": true
  }
}
```

**Response — Changed Handle (201)**

The backend must perform this transaction:

1. Mark the old active member for the same `youtube_user_id` as `inactive`.
2. Preserve the old member's points.
3. Create a new active member with the new normalized handle and 0 points.
4. Do not transfer points automatically.

**Response — Active Handle Conflict (409)**

```json
{
  "error": {
    "code": "HANDLE_IN_USE",
    "message": "This handle is already registered by another active member"
  }
}
```

**Response — Backend Unavailable**

streamer.bot maps network/API failure to:

```text
🦌 Registration is temporarily unavailable. Please try again later.
```

---

### 5.2 PUT `/api/v1/members/{youtube_user_id}/visibility` — Change Visibility

Called by `:deer: public` and `:deer: private`.

**Request**

```json
{
  "public_visibility": false
}
```

**Response (200)**

```json
{
  "data": {
    "youtube_user_id": "UC-viewer-123",
    "public_visibility": false
  }
}
```

Rules:

- No member creation.
- No point changes.
- No handle changes.
- If no active member exists, return `MEMBER_NOT_REGISTERED`.

---

### 5.3 GET `/api/v1/members/{youtube_user_id}/points` — Visibility-Aware Points

Called by streamer.bot for `:deer: point` using the actual chat author identity.

**Public Member with Exact Points**

```json
{
  "data": {
    "youtube_handle": "viewer123",
    "public_visibility": true,
    "point_display": "exact",
    "total_points": 563
  }
}
```

**Private Member with Banded Points**

```json
{
  "data": {
    "youtube_handle": "viewer123",
    "public_visibility": false,
    "point_display": "range",
    "lower": 500,
    "upper": 600
  }
}
```

Banded calculation:

```text
lower = floor(total_points / 100) * 100
upper = lower + 100
```

Examples:

```text
563  → 500–600
600  → 600–700
1249 → 1200–1300
0    → 0–100
```

**Unregistered Member**

```json
{
  "error": {
    "code": "MEMBER_NOT_REGISTERED",
    "message": "Viewer is not registered"
  }
}
```

streamer.bot maps this to:

```text
🦌 You are not registered yet. Use :deer: register first.
```

---

### 5.4 GET `/api/v1/scoreboard` — Public Scoreboard

Public read-only endpoint consumed by `deerngo-web`.

**Query Parameters**

| Parameter | Type | Default | Rule |
|-----------|------|---------|------|
| `page` | integer | 1 | 1-based; must be >= 1 |
| `limit` | integer | 50 | 1–100; behavior above 100 must be documented by Dev |

Only records satisfying all conditions are returned:

```sql
status = 'active'
AND public_visibility = true
AND total_points > 0
```

**Response (200)**

```json
{
  "data": [
    {
      "rank": 1,
      "youtube_handle": "topdonor",
      "total_points": 1500
    },
    {
      "rank": 2,
      "youtube_handle": "viewer123",
      "total_points": 563
    }
  ],
  "meta": {
    "total": 2,
    "page": 1,
    "limit": 50,
    "pages": 1,
    "hasNext": false,
    "hasPrev": false
  }
}
```

The response must never contain:

- YouTube display name
- `youtube_user_id`
- Raw EasyDonate donor name
- Donation message
- Private member record
- Inactive member record
- Zero-point member record

---

### 5.5 POST `/api/v1/webhooks/easydonate/{path_token}` — EasyDonate Webhook

> EasyDonate's current public documentation confirms webhook URL configuration and payload examples, but does not confirm HMAC signing or an `X-EasyDonate-Signature` header. Do not implement HMAC unless the dashboard/provider contract confirms it.

**MVP Protection**

- `{path_token}` is a long random secret path segment stored only in deployment configuration.
- Strict JSON and amount/currency/time validation.
- Request body size limit.
- Rate limiting.
- `referenceNo` unique idempotency key.
- Raw donor data remains private.

**Expected Provider Payload**

```json
{
  "referenceNo": "EZDN-ABC123456",
  "channelName": "TRUEWALLET_ANGPAO",
  "donatorName": "Viewer123",
  "donateMessage": "Keep streaming!",
  "amount": 1000,
  "time": "2024-01-15T10:30:00.000Z"
}
```

The actual payload must be verified against a real EasyDonate dashboard/test event before production acceptance.

**Response — Accepted (200)**

```json
{
  "data": {
    "reference_no": "EZDN-ABC123456",
    "stored": true,
    "source": "webhook"
  }
}
```

**Response — Duplicate (200)**

```json
{
  "data": {
    "reference_no": "EZDN-ABC123456",
    "stored": false,
    "skipped": true,
    "reason": "Donation already processed"
  }
}
```

**Processing Rules**

1. Validate path token and request body.
2. Validate `amount > 0`, expected currency, and timestamp format.
3. Insert by unique `referenceNo`; reject/skip duplicates.
4. Store raw donor name privately for exact matching and reconciliation.
5. Do not award points in the ingestion handler unless the matching/cutoff transaction succeeds.

---

### 5.6 Internal EasyDonate API Sync — Fallback

| Field | Detail |
|-------|--------|
| Schedule | Every 5 minutes, configurable |
| Auth | Personal EasyDonate API key as `Authorization: Bearer <key>` |
| Scope | Provider-confirmed donation-read scope, currently documented as `read:donations` |
| Endpoint | Provider API donation-list endpoint; confirm base URL and query parameters from current OpenAPI reference |
| Source | `api_poll` |
| Idempotency | `referenceNo` |

The API key is backend-only and must never appear in browser code or logs.

---

### 5.7 Internal Exact Matching and Points Application

The matching worker processes pending donations:

1. Normalize `donatorName`: trim, remove leading `@`, lowercase.
2. Find an **active** member with exactly matching normalized `youtube_handle`.
3. Require `donation_time >= member.registered_at`.
4. If no match or cutoff fails, mark the donation uncredited and retain it privately.
5. If eligible, update `donations.matched_member_id` and apply `amount_thb` once to `members.total_points`.
6. Use a transaction/conditional update to prevent duplicate point application.

Fuzzy matching and `pg_trgm` are not used for Phase 1 member points.

---

## 6. Security and Privacy Contract

- No display name storage for members.
- Raw donor names and messages are private operational data.
- Public scoreboard returns normalized handle and points only.
- `:deer: private` hides a member while preserving private points.
- Do not assume HMAC until EasyDonate confirms it.
- Use an unpredictable webhook path token until a provider-supported authentication mechanism is verified.
- Never log API keys, path tokens, client secrets, refresh tokens, donor messages, or full provider payloads.

## 7. Superseded API Contracts

| Former Contract | Status | Replacement |
|-----------------|--------|-------------|
| `POST /api/v1/subscribers` | Superseded | `POST /api/v1/members/register` |
| `GET /api/v1/points/{handle}` | Superseded | `GET /api/v1/members/{youtube_user_id}/points` |
| YouTube API polling | Removed from MVP | streamer.bot explicit registration |
| Fuzzy `pg_trgm` matching | Removed from MVP | normalized exact active-member match |
| HMAC webhook assumption | Unverified/superseded | provider-confirmed auth or secret path MVP fallback |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[012_user_stories]] | Current user stories |
| [[013_acceptance_criteria]] | Current acceptance criteria |
| [[023_database_schema_DDL]] | Current physical data model |
| [[024_ERD]] | Current logical data model |
| [[021_architecture_decision_records]] | Decision rationale |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Approved scope change |

---

> **Template Standard:** Based on SWEBOK v4 and OpenAPI Specification 3.0
> **Usage:** This is the current contract between streamer.bot, EasyDonate, the backend, and the frontend.
