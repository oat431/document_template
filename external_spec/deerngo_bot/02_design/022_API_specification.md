---
document_type: API Specification
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
tech_lead: "SA / Designer Persona"
classification: "Internal"
tags: [api-specification, openapi, rest, swebok, vrm, fiber]
standard_ref:
  - SWEBOK v4 — Design
  - OpenAPI Specification 3.0
parent_project: "Deerngo Bot — VRM"
---

# API Specification

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | SA / Designer Persona |
| Framework | Fiber v3 (fasthttp-based) |
| Database Access | sqlx |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | SA | Initial API spec — 4 public endpoints + 1 webhook + internal schedulers |

---

## 1. Purpose

> This document defines the API contracts for the Deerngo Bot Go backend. It serves as the contract between the streamer.bot integration, the EasyDonate webhook, and the React/Next.js frontend.

---

## 2. API Overview

| Field | Detail |
|-------|--------|
| Base URL | `http://192.168.1.121:8008` (LAN — streamer.bot connects here) |
| Docker Internal | `http://deerngo-bot:8008` (Next.js connects here via `db-network`) |
| Public URL | Via Cloudflare Tunnel (e.g., `https://deerngo-viewer-score.panomete.com`) — scoreboard only |
| Protocol | HTTP (internal), HTTPS (public via Cloudflare) |
| Format | JSON |
| Authentication | None for Phase 1 (all endpoints are internal or public read-only) |
| Rate Limiting | 100 requests/minute per IP (Fiber middleware) |
| Versioning | URL path — `/api/v1/` |

---

## 3. Common Response Formats

### Success Response

```json
{
  "data": { ... },
  "meta": {
    "timestamp": "2026-07-29T10:00:00Z"
  }
}
```

### Error Response

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

| Code | HTTP Status | Description |
|------|-----------|-------------|
| VALIDATION_ERROR | 400 | Input validation failed |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists (upsert returns 200 instead) |
| RATE_LIMITED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |
| WEBHOOK_INVALID | 401 | Webhook signature verification failed |

---

## 4. API Endpoints

### 4.1 POST /api/v1/subscribers — Register Subscriber

> Called by streamer.bot when a new YouTube subscriber event fires during a live stream.

| Field | Detail |
|-------|--------|
| Description | Create or update a subscriber record (upsert by youtube_handle) |
| Auth | None (internal endpoint, localhost only) |
| Rate Limit | 100/min |
| Called By | streamer.bot (HTTP Request sub-action) |

**Request Body:**

```json
{
  "youtube_handle": "@viewer1",
  "display_name": "Viewer One",
  "subscribed_at": "2026-07-29T10:00:00Z",
  "source": "streamer_bot"
}
```

**Validation Rules:**

| Field | Rule | Error |
|-------|------|-------|
| youtube_handle | Required, string, 1–100 chars | VALIDATION_ERROR |
| display_name | Required, string, 1–255 chars | VALIDATION_ERROR |
| subscribed_at | Required, ISO 8601 timestamp | VALIDATION_ERROR |
| source | Required, must be "streamer_bot" | VALIDATION_ERROR |

**Response — New Subscriber (201):**

```json
{
  "data": {
    "id": "uuid",
    "youtube_handle": "@viewer1",
    "display_name": "Viewer One",
    "subscribed_at": "2026-07-29T10:00:00Z",
    "source": "streamer_bot",
    "created_at": "2026-07-29T10:00:01Z",
    "updated_at": "2026-07-29T10:00:01Z"
  }
}
```

**Response — Existing Subscriber Upserted (200):**

```json
{
  "data": {
    "id": "uuid",
    "youtube_handle": "@viewer1",
    "display_name": "Viewer One",
    "subscribed_at": "2026-07-28T08:00:00Z",
    "source": "youtube_api",
    "created_at": "2026-07-28T08:00:01Z",
    "updated_at": "2026-07-29T10:00:01Z"
  },
  "meta": {
    "upserted": true,
    "message": "Existing record preserved (earliest timestamp kept)"
  }
}
```

**Error — Validation (400):**

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

**Upsert Logic:**
1. Normalize `youtube_handle`: lowercase, strip `@`, trim whitespace
2. `INSERT ... ON CONFLICT (youtube_handle) DO UPDATE`
3. On update: preserve the **earliest** `subscribed_at`, update `display_name` and `source`
4. Return 201 for new, 200 for upsert

---

### 4.2 GET /api/v1/points/{handle} — Query Viewer Points

> Called by streamer.bot when a viewer types `:deer: point` in chat.

| Field | Detail |
|-------|--------|
| Description | Get a viewer's point balance by YouTube handle |
| Auth | None (internal endpoint, localhost only) |
| Rate Limit | 100/min |
| Called By | streamer.bot (HTTP Request sub-action) |

**Path Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| handle | string | YouTube handle (without @) |

**Response — Viewer Has Points (200):**

```json
{
  "data": {
    "youtube_handle": "@viewer1",
    "display_name": "Viewer One",
    "total_points": 500.00,
    "donation_count": 3,
    "last_donation": "2026-07-29T09:30:00Z"
  }
}
```

**Response — Viewer Has No Points (200):**

```json
{
  "data": {
    "youtube_handle": "@newviewer",
    "display_name": "New Viewer",
    "total_points": 0,
    "donation_count": 0,
    "last_donation": null
  }
}
```

> **Note:** Always returns 200, never 404. A viewer with no points gets `total_points: 0`. The handle is normalized (lowercase, strip @) before lookup.

**Error — Internal (500):**

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Failed to query points"
  }
}
```

---

### 4.3 GET /api/v1/scoreboard — Public Scoreboard

> Called by the React/Next.js frontend to display the ranked scoreboard.

| Field | Detail |
|-------|--------|
| Description | Get ranked list of viewers by points (descending) |
| Auth | None (public read-only endpoint) |
| Rate Limit | 100/min |
| Called By | React frontend, public browsers |

**Query Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | integer | 1 | Page number (1-based) |
| limit | integer | 50 | Items per page (max 100) |

**Response — Scoreboard (200):**

> **Note:** Only viewers with `total_points > 0` are included. Viewers who haven't donated are excluded from the scoreboard.

```json
{
  "data": [
    {
      "rank": 1,
      "display_name": "Top Donor",
      "youtube_handle": "@topdonor",
      "total_points": 1500.00,
      "donation_count": 10
    },
    {
      "rank": 2,
      "display_name": "Viewer One",
      "youtube_handle": "@viewer1",
      "total_points": 500.00,
      "donation_count": 3
    }
  ],
  "meta": {
    "total": 25,
    "page": 1,
    "limit": 50,
    "pages": 1,
    "hasNext": false,
    "hasPrev": false
  }
}
```

**Response — Empty Scoreboard (200):**

> No viewers have donated yet (or all viewers have 0 points).

```json
{
  "data": [],
  "meta": {
    "total": 0,
    "page": 1,
    "limit": 50,
    "pages": 0,
    "hasNext": false,
    "hasPrev": false
  }
}
```

---

### 4.4 POST /api/v1/webhooks/easydonate — EasyDonate Webhook

> Receives donation events from EasyDonate. Verifies HMAC-SHA256 signature before processing.

| Field | Detail |
|-------|--------|
| Description | Receive and process donation webhook events |
| Auth | HMAC-SHA256 signature verification |
| Rate Limit | 100/min |
| Called By | EasyDonate webhook system |

**Request Headers:**

| Header | Description |
|--------|-------------|
| X-EasyDonate-Signature | HMAC-SHA256 signature of the request body |
| Content-Type | application/json |

**Request Body:**

```json
{
  "id": "ed-12345",
  "donor_name": "viewer1",
  "amount": 100.00,
  "currency": "THB",
  "message": "Keep streaming!",
  "created_at": "2026-07-29T10:00:00Z"
}
```

**Response — Processed (200):**

```json
{
  "data": {
    "id": "uuid",
    "easydonate_id": "ed-12345",
    "donor_name": "viewer1",
    "amount_thb": 100.00,
    "match_status": "pending",
    "source": "webhook"
  }
}
```

**Error — Invalid Signature (401):**

```json
{
  "error": {
    "code": "WEBHOOK_INVALID",
    "message": "Invalid webhook signature"
  }
}
```

**Error — Duplicate (200 with skip):**

```json
{
  "data": {
    "easydonate_id": "ed-12345",
    "skipped": true,
    "reason": "Donation already processed"
  }
}
```

**Webhook Verification Flow:**
1. Read raw request body
2. Compute `HMAC-SHA256(body, secret_key)` where `secret_key` is from environment variable
3. Compare with `X-EasyDonate-Signature` header (constant-time comparison)
4. If mismatch → 401
5. If match → parse JSON, check `easydonate_id` uniqueness, store donation

---

### 4.5 Internal: EasyDonate Sync (Polling Fallback)

> Background goroutine that polls EasyDonate REST API every 5 minutes as a fallback for missed webhooks.

| Field | Detail |
|-------|--------|
| Description | Poll EasyDonate API for recent donations not yet in the database |
| Schedule | Every 5 minutes |
| Auth | OAuth 2.0 (token from `oauth_tokens` table) |
| Endpoint | `GET https://easydonate.app/api/v1/shop/deerngo0/donations` |

**Sync Logic:**
1. Fetch last 50 donations from EasyDonate API
2. For each donation, check if `easydonate_id` exists in `donations` table
3. If not exists → insert with `source = 'api_poll'`, `match_status = 'pending'`
4. If exists → skip (idempotent)
5. On 429 (rate limit) → back off, respect `Retry-After` header

---

### 4.6 Internal: YouTube API Polling Scheduler

> Background goroutine that polls YouTube Data API every 15 minutes for new subscribers.

| Field | Detail |
|-------|--------|
| Description | Poll YouTube Data API for new subscribers |
| Schedule | Every 15 minutes |
| Auth | OAuth 2.0 (token from `oauth_tokens` table) |
| Endpoint | `GET https://www.googleapis.com/youtube/v3/subscriptions` |
| Quota | 1 unit per call × 96 calls/day = 96 units (well within 10,000/day limit) |

**Polling Logic:**
1. Call `GET /youtube/v3/subscriptions?part=snippet&channelId={channel_id}&maxResults=50&order=date`
2. For each subscriber, normalize handle (lowercase, strip @)
3. `INSERT ... ON CONFLICT (youtube_handle) DO UPDATE` — preserve earliest `subscribed_at`
4. On 403 (quota exceeded) → log error, skip next cycle
5. On 401 (token expired) → log error, alert operator (manual re-auth required)

---

### 4.7 Internal: Name Matching Engine

> Background goroutine that processes `pending` donations and matches them to subscribers.

| Field | Detail |
|-------|--------|
| Description | Match pending donations to subscribers via fuzzy name matching |
| Schedule | Every 2 minutes (after donation sync) |
| Algorithm | PostgreSQL `pg_trgm` `similarity()` function |

**Matching Logic:**
1. `SELECT * FROM donations WHERE match_status = 'pending'`
2. For each donation:
   a. Normalize `donor_name`: lowercase, strip `@`, trim whitespace
   b. Skip if `donor_name` is 'anonymous' or empty → set `match_status = 'unmatched'`
   c. Query: `SELECT youtube_handle, similarity(youtube_handle, $1) AS score FROM subscribers WHERE similarity(youtube_handle, $1) > 0.7 ORDER BY score DESC LIMIT 2`
   d. If exactly 1 match with score > 0.7 → set `match_status = 'matched'`, `matched_handle`, `match_score`
   e. If multiple matches with score > 0.7 → set `match_status = 'manual_review'`, store best match and score
   f. If no match > 0.7 → set `match_status = 'unmatched'`
3. Update donation record
4. Trigger fires → `viewer_points` updated automatically

---

## 5. Rate Limiting

| Tier | Limit | Window | Response |
|------|-------|--------|---------|
| Default | 100 requests | 1 minute | 429 when exceeded |
| Webhook | 200 requests | 1 minute | 429 when exceeded |

> Implemented via Fiber's `limiter` middleware. Per-IP tracking.

---

## 6. Deployment Notes

| Aspect | Detail |
|--------|--------|
| Container | Docker, joins `db-network` on homelab |
| Port | `:8008` (configurable via `PORT` env var) |
| LAN Access | `http://192.168.1.121:8008` (streamer.bot on Windows PC connects here) |
| Docker Internal | `http://deerngo-bot:8008` (Next.js frontend connects here) |
| Public Access | Via Cloudflare Tunnel — only scoreboard endpoint exposed publicly |
| CORS | Allow origin from scoreboard frontend URL (Cloudflare hostname) |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[023_database_schema_DDL]] | Database tables these endpoints read/write |
| [[024_ERD]] | Data model underlying the API |
| [[021_architecture_decision_records]] | ADR-009 (Fiber), ADR-012 (HMAC-SHA256) |
| [[012_user_stories]] | User stories these endpoints implement |
| [[013_acceptance_criteria]] | ACs that verify endpoint behavior |

---

> **Template Standard:** Based on SWEBOK v4, OpenAPI Specification 3.0
> **Usage:** This is the *contract* between streamer.bot, EasyDonate, and the frontend. Both sides code against this spec. Generate OpenAPI YAML from this document for tooling integration.
