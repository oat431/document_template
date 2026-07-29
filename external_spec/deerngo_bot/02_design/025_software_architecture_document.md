---
document_type: SAD (Software Architecture Document)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
architect: "SA / Designer Persona"
classification: "Internal"
tags: [sad, software-architecture, swebok, iso-42010, vrm]
standard_ref:
  - SWEBOK v4 — Architecture
  - ISO/IEC/IEEE 42010 — Architecture Description
parent_project: "Deerngo Bot — VRM"
---

# Software Architecture Document (SAD)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | SA / Designer Persona |
| Solution Architect | SA / Designer Persona |
| Stakeholder | Deer_NGO (YouTube Creator) |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | SA | Initial SAD — monolith architecture, component design, data flow |

---

## 1. Introduction

### 1.1 Purpose

> This document describes the software architecture of Deerngo Bot — the fundamental concepts, properties, and characteristics. It serves as the primary architectural reference for the developer implementing the system.

### 1.2 Scope

> Deerngo Bot is a Viewer Relationship Management (VRM) system for the @Deer_NGO YouTube channel. It captures subscribers, processes donations, calculates points, and provides a public scoreboard. Phase 1 MVP scope only.

### 1.3 References

| Document | Version |
|---------|---------|
| [[011_business_objective]] | v0.1 |
| [[012_user_stories]] | v0.1 |
| [[013_acceptance_criteria]] | v0.1 |
| [[021_architecture_decision_records]] | v0.1 |
| [[022_API_specification]] | v0.1 |
| [[023_database_schema_DDL]] | v0.1 |

---

## 2. Architecture Overview

### 2.1 Architectural Style

| Aspect | Choice | Rationale |
|--------|-------|----------|
| Overall Style | **Modular Monolith** | Single Go binary, internally layered. Simple to deploy on Windows. No microservice overhead for Phase 1. |
| Communication | **REST (sync) + Goroutines (async)** | REST for API endpoints; goroutines for background schedulers (YouTube polling, EasyDonate sync, name matching) |
| Data Management | **Single PostgreSQL database** | All tables in one `deerngo` database. Shared instance on homelab. |
| Deployment | **Local Windows binary** | Single `.exe` file, runs alongside streamer.bot. Cloudflare Tunnel for public access. |

### 2.2 High-Level Architecture

```mermaid
flowchart TB
    subgraph YouTube["YouTube Platform"]
        YT_Chat["Live Chat"]
        YT_Sub["Subscription Events"]
        YT_API["YouTube Data API v3"]
    end

    subgraph SB["Streamer.bot (Windows PC :7474/:8681)"]
        SB_Trig["Triggers<br>Chat Message / New Sub"]
        SB_Actions["Actions<br>Send Chat / HTTP Request"]
    end

    subgraph Homelab["Homelab Server (Ubuntu 26.04, Docker)"]
        subgraph Backend["Go Backend Container (:8008)"]
            direction TB
            subgraph API_Layer["API Layer (Fiber)"]
                EP_Sub["POST /api/v1/subscribers"]
                EP_Points["GET /api/v1/points/{handle}"]
                EP_Score["GET /api/v1/scoreboard"]
                EP_Webhook["POST /api/v1/webhooks/easydonate"]
            end
            subgraph Services["Service Layer"]
                SVC_Sub["Subscriber Service"]
                SVC_Don["Donation Service"]
                SVC_Pts["Points Service"]
                SVC_Match["Name Matching Engine"]
            end
            subgraph Schedulers["Background Schedulers"]
                SCH_YT["YouTube API Poller<br>(every 15 min)"]
                SCH_ED["EasyDonate Sync<br>(every 5 min)"]
                SCH_Match["Name Matcher<br>(every 2 min)"]
            end
        end

        subgraph DB["PostgreSQL 18 Container (db-network)"]
            T_Sub["subscribers"]
            T_Don["donations"]
            T_Pts["viewer_points"]
            T_OAuth["oauth_tokens"]
        end

        subgraph Frontend["Next.js Frontend Container (:3008)"]
            Web_Score["Public Scoreboard Page"]
        end

        CF["Cloudflare Tunnel (systemd)"]
    end

    subgraph EasyDonate["EasyDonate Platform"]
        ED_Webhook["Webhook (push)"]
        ED_API["REST API (poll fallback)"]
    end

    subgraph Internet["Internet"]
        DNS["deerngo-viewer-score.panomete.com"]
    end

    %% Subscriber flow — HYBRID
    YT_Sub --> SB_Trig
    SB_Trig -->|"LAN 192.168.1.121:8008"| EP_Sub
    YT_API -->|"poll every 15min (24/7)"| SCH_YT
    EP_Sub --> SVC_Sub
    SCH_YT --> SVC_Sub
    SVC_Sub -->|"upsert"| T_Sub

    %% Chat command flow
    YT_Chat --> SB_Trig
    SB_Trig -->|":deer: donate"| SB_Actions
    SB_Trig -->|":deer: point"| EP_Points
    EP_Points --> SVC_Pts
    SVC_Pts --> T_Pts
    EP_Points -->|"response"| SB_Actions
    SB_Actions -->|"chat message"| YT_Chat

    %% Donation flow
    ED_Webhook -->|"POST"| EP_Webhook
    EP_Webhook --> SVC_Don
    ED_API -->|"poll (fallback)"| SCH_ED
    SCH_ED --> SVC_Don
    SVC_Don --> T_Don

    %% Name matching flow
    SCH_Match --> SVC_Match
    SVC_Match --> T_Don
    SVC_Match --> T_Sub
    SVC_Match -->|"update match_status"| T_Don

    %% Points sync (trigger)
    T_Don -->|"trigger on match"| T_Pts

    %% Scoreboard flow
    DNS -->|"HTTPS"| CF
    CF -->|"tunnel"| Web_Score
    Web_Score -->|"GET"| EP_Score
    EP_Score --> SVC_Pts
    SVC_Pts --> T_Pts

    style YouTube fill:#FF0000,color:#fff
    style SB fill:#7B68EE,color:#fff
    style Homelab fill:#2196F3,color:#fff
    style Backend fill:#1565C0,color:#fff
    style DB fill:#4CAF50,color:#fff
    style EasyDonate fill:#FF9800,color:#fff
    style Frontend fill:#00BCD4,color:#fff
    style Internet fill:#607D8B,color:#fff
```

---

## 3. Component Design

### 3.1 Go Backend — Internal Layering

```
┌─────────────────────────────────────────────┐
│                 API Layer (Fiber)            │
│  Routes, middleware, request/response DTOs   │
├─────────────────────────────────────────────┤
│               Service Layer                  │
│  Business logic, orchestration, validation   │
├─────────────────────────────────────────────┤
│              Repository Layer (sqlx)         │
│  SQL queries, struct scanning, transactions  │
├─────────────────────────────────────────────┤
│            Scheduler Layer (goroutines)      │
│  YouTube poller, EasyDonate sync, matcher    │
└─────────────────────────────────────────────┘
```

### 3.2 Subscriber Service

| Aspect | Detail |
|--------|--------|
| Responsibility | Accept subscriber events, upsert to database, normalize handles |
| Layer | Service |
| Dependencies | `subscribers` table (sqlx) |
| API | `POST /api/v1/subscribers` (streamer.bot push), YouTube API poller (scheduled) |
| Key Logic | Normalize handle (lowercase, strip @), `INSERT ... ON CONFLICT DO UPDATE`, preserve earliest `subscribed_at` |

### 3.3 Donation Service

| Aspect | Detail |
|--------|--------|
| Responsibility | Receive donation events (webhook + API poll), store in database, trigger matching |
| Layer | Service |
| Dependencies | `donations` table (sqlx), EasyDonate API client |
| API | `POST /api/v1/webhooks/easydonate` (webhook), EasyDonate poller (scheduled) |
| Key Logic | Idempotent insert (by `easydonate_id`), HMAC-SHA256 verification, rate limit handling |

### 3.4 Points Service

| Aspect | Detail |
|--------|--------|
| Responsibility | Query viewer points, generate scoreboard rankings |
| Layer | Service |
| Dependencies | `viewer_points` table (sqlx) |
| API | `GET /api/v1/points/{handle}`, `GET /api/v1/scoreboard` |
| Key Logic | Case-insensitive handle lookup, pagination for scoreboard, rank calculation |

### 3.5 Name Matching Engine

| Aspect | Detail |
|--------|--------|
| Responsibility | Match pending donations to subscribers using fuzzy name matching |
| Layer | Scheduler (background goroutine) |
| Dependencies | `donations` table, `subscribers` table (sqlx) |
| Schedule | Every 2 minutes |
| Key Logic | `pg_trgm` `similarity()` function, threshold 0.7, flag unmatched/ambiguous |

### 3.6 YouTube API Poller

| Aspect | Detail |
|--------|--------|
| Responsibility | Poll YouTube Data API for new subscribers every 15 minutes |
| Layer | Scheduler (background goroutine) |
| Dependencies | YouTube Data API v3, `subscribers` table, `oauth_tokens` table |
| Schedule | Every 15 minutes |
| Key Logic | Load refresh token from `oauth_tokens` → exchange for access token via Google OAuth → call YouTube API → upsert subscribers → handle 403/401 errors |

### 3.7 EasyDonate Sync

| Aspect | Detail |
|--------|--------|
| Responsibility | Poll EasyDonate REST API for recent donations as webhook fallback |
| Layer | Scheduler (background goroutine) |
| Dependencies | EasyDonate REST API, `donations` table, `oauth_tokens` table |
| Schedule | Every 5 minutes |
| Key Logic | Idempotent insert, rate limit handling (429 + Retry-After), backoff |

---

## 4. Data Architecture

### 4.1 Data Model (High-Level)

```mermaid
erDiagram
    SUBSCRIBERS ||--o{ DONATIONS : "matched via fuzzy name"
    SUBSCRIBERS ||--|| VIEWER_POINTS : "points summary"

    SUBSCRIBERS {
        string youtube_handle UK
        string display_name
        timestamp subscribed_at
        string source
    }
    DONATIONS {
        string easydonate_id UK
        string donor_name
        decimal amount_thb
        string matched_handle FK
        string match_status
    }
    VIEWER_POINTS {
        string youtube_handle UK
        decimal total_points
        integer donation_count
    }
```

### 4.2 Data Storage Strategy

| Data Type | Store | Retention | Backup |
|-----------|-------|-----------|--------|
| Subscribers | PostgreSQL (`subscribers`) | Indefinite | Homelab backup schedule |
| Donations | PostgreSQL (`donations`) | Indefinite | Homelab backup schedule |
| Points | PostgreSQL (`viewer_points`) | Indefinite | Homelab backup schedule |
| OAuth Tokens | PostgreSQL (`oauth_tokens`) | Until refreshed | Homelab backup schedule |
| Logs | stdout / file | 7 days (manual rotation) | — |

---

## 5. Security Architecture

### 5.1 Security Model (Phase 1 — Minimal)

| Layer | Mechanism | Implementation |
|-------|----------|---------------|
| Webhook | HMAC-SHA256 signature | Verify `X-EasyDonate-Signature` header |
| API | No auth (internal) | All endpoints are localhost-only except scoreboard |
| Scoreboard | No auth (public read-only) | No sensitive data exposed |
| Database | Connection string in env var | `DATABASE_URL` environment variable |
| OAuth Tokens | Stored in PostgreSQL | Access + refresh tokens in `oauth_tokens` table |
| Cloudflare | TLS termination + DDoS protection | Cloudflare Tunnel handles external security |

### 5.2 Security Controls

| Control | Implementation | Standard |
|---------|---------------|---------|
| Webhook Verification | HMAC-SHA256 constant-time comparison | OWASP |
| Input Validation | Fiber request binding + manual validation | OWASP |
| Rate Limiting | Fiber limiter middleware (100 req/min) | OWASP API |
| SQL Injection | sqlx parameterized queries | OWASP |
| Environment Variables | Secrets in env vars, not in code | 12-Factor App |
| CORS | Allow only scoreboard origin | OWASP |

---

## 6. Deployment Architecture

### 6.1 Deployment Topology

```mermaid
flowchart TB
    subgraph LocalPC["Windows Local PC"]
        SB["Streamer.bot<br>:7474 (HTTP), :8681 (WS)"]
    end

    subgraph Homelab["Homelab Server (Ubuntu 26.04, Docker)"]
        subgraph DockerNet["Docker Network: db-network"]
            Go["Go Backend Container<br>:8008"]
            Next["Next.js Frontend Container<br>:3008"]
            PG[("PostgreSQL 18 Container<br>:5432")]
            Valkey["Valkey 9 Container<br>:6379"]
        end
        CF["Cloudflare Tunnel<br>(systemd service)"]
    end

    subgraph Internet["Internet"]
        DNS["deerngo-viewer-score.panomete.com<br>(or subdomain)"]
    end

    SB -->|"LAN 192.168.1.121:8008"| Go
    Go -->|"db-network: local-postgres:5432"| PG
    Next -->|"db-network: deerngo-bot:8008"| Go
    CF -->|"tunnel"| Next
    DNS --> CF

    style LocalPC fill:#7B68EE,color:#fff
    style Homelab fill:#2196F3,color:#fff
    style DockerNet fill:#1a237e,color:#fff
    style Internet fill:#607D8B,color:#fff
```

### 6.2 Component Deployment

| Component | Location | Port | Process | Notes |
|-----------|----------|------|---------|-------|
| Go Backend | Homelab (Docker, `db-network`) | :8008 | `deerngo-bot` container | Single binary in container, joins `db-network` |
| Next.js Frontend | Homelab (Docker, `db-network`) | :3008 | `deerngo-web` container | Exposed via Cloudflare Tunnel |
| PostgreSQL 18 | Homelab (Docker, `db-network`) | :5432 | `local-postgres` container | Existing — `deerngo` database already created |
| Valkey 9 | Homelab (Docker, `db-network`) | :6379 | `local-valkey` container | Available for future caching/rate limiting |
| streamer.bot | Local Windows PC | :7474, :8681 | Windows app | Connects to Go backend at `192.168.1.121:8008` |
| Cloudflare Tunnel | Homelab (systemd) | — | `cloudflared.service` | Already running — add scoreboard hostname |

### 6.3 Docker Compose

```yaml
# docker-compose.yml for deerngo-bot
version: "3.8"

services:
  deerngo-bot:
    build: .
    container_name: deerngo-bot
    restart: unless-stopped
    ports:
      - "8080:8008"
    environment:
      - DATABASE_URL=postgres://postgres:postgres@local-postgres:5432/deerngo?sslmode=disable
      - PORT=8080
      - EASYDONATE_WEBHOOK_SECRET=${EASYDONATE_WEBHOOK_SECRET}
      - YOUTUBE_CHANNEL_ID=${YOUTUBE_CHANNEL_ID}
      - SCOREBOARD_ORIGIN=${SCOREBOARD_ORIGIN}
    networks:
      - db-network
    depends_on:
      local-postgres:
        condition: service_healthy

  deerngo-web:
    build:
      context: ../deerngo-web
      dockerfile: Dockerfile
    container_name: deerngo-web
    restart: unless-stopped
    ports:
      - "3000:3008"
    environment:
      - NEXT_PUBLIC_API_URL=http://deerngo-bot:8008
    networks:
      - db-network
    depends_on:
      - deerngo-bot

networks:
  db-network:
    external: true
```

### 6.4 Network Communication

| From | To | Protocol | Address | Notes |
|------|-----|----------|---------|-------|
| streamer.bot (Windows PC) | Go Backend (Homelab) | HTTP | `192.168.1.121:8008` | LAN call, ~1ms |
| Go Backend | PostgreSQL | TCP | `local-postgres:5432` | Docker `db-network` |
| Next.js Frontend | Go Backend | HTTP | `deerngo-bot:8008` | Docker `db-network` |
| Cloudflare Tunnel | Next.js Frontend | HTTP | `localhost:3008` | Systemd → Docker port |
| Public Internet | Cloudflare | HTTPS | `deerngo-viewer-score.panomete.com` | Cloudflare TLS |

### 6.5 Environment Variables

| Variable | Description | Example |
|----------|------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgres://postgres:postgres@local-postgres:5432/deerngo?sslmode=disable` |
| `PORT` | Go backend listen port | `8080` |
| `EASYDONATE_WEBHOOK_SECRET` | HMAC-SHA256 secret for webhook verification | `secret-key-here` |
| `YOUTUBE_CHANNEL_ID` | YouTube channel ID for API polling | `UCxxxxxxxxxxxxxxxxxxxxxxx` |
| `SCOREBOARD_ORIGIN` | CORS origin for scoreboard | `https://deerngo-viewer-score.panomete.com` |
| `NEXT_PUBLIC_API_URL` | Go backend URL (from Next.js) | `http://deerngo-bot:8008` |

---

## 7. Quality Attributes

| Attribute | Scenario | Architecture Response | Verification |
|-----------|---------|---------------------|-------------|
| Performance | `:deer: point` command must respond <2s | sqlx direct query on indexed `viewer_points` table; no ORM overhead | Load test with 100 concurrent queries |
| Availability | Scoreboard must be accessible during streams | Cloudflare Tunnel with automatic reconnection; local process restart on crash | Manual monitoring |
| Data Integrity | No duplicate donations, no lost subscribers | Idempotent inserts (`easydonate_id` UNIQUE, `youtube_handle` UNIQUE); upsert with earliest timestamp preservation | Integration tests |
| Maintainability | Single developer, weekly changes | Modular monolith with clear layering; sqlx for readable SQL; no framework magic | Code review |
| Security | Fake donation prevention | HMAC-SHA256 webhook verification; input validation on all endpoints | Penetration test (manual) |

---

## 8. Architecture Decision Summary

| # | Decision | Rationale | ADR |
|---|---------|----------|-----|
| 1 | Go backend | Lightweight, fast, stakeholder comfortable | ADR-001 |
| 2 | PostgreSQL 18 | Existing homelab infrastructure | ADR-002 |
| 3 | React/Next.js frontend | Nice UI for public scoreboard | ADR-003 |
| 4 | Local Windows deployment | Same PC as streamer.bot | ADR-004 |
| 5 | Hybrid subscriber capture | 24/7 coverage (API) + real-time (streamer.bot) | ADR-005 |
| 6 | Fuzzy name matching (pg_trgm) | Donation name ≠ YouTube handle | ADR-006 |
| 7 | EasyDonate webhook primary | Real-time + fallback polling | ADR-007 |
| 8 | sqlx for DB access | Lean, type-safe, full SQL control | ADR-008 |
| 9 | Fiber web framework | Fast, Express-like API | ADR-009 |
| 10 | Tailwind + DaisyUI | Fast UI prototyping, no JS overhead | ADR-010 |
| 11 | Cloudflare Tunnel | Already running, handles TLS + DDoS | ADR-011 |
| 12 | HMAC-SHA256 verification | Standard webhook security | ADR-012 |

---

## 9. Go Project Structure

```
deerngo-bot/
├── cmd/
│   └── server/
│       └── main.go              # Entry point, Fiber app setup, scheduler startup
├── internal/
│   ├── config/
│   │   └── config.go            # Environment variable loading
│   ├── handler/
│   │   ├── subscriber.go        # POST /api/v1/subscribers
│   │   ├── points.go            # GET /api/v1/points/{handle}
│   │   ├── scoreboard.go        # GET /api/v1/scoreboard
│   │   └── webhook.go           # POST /api/v1/webhooks/easydonate
│   ├── service/
│   │   ├── subscriber.go        # Subscriber business logic
│   │   ├── donation.go          # Donation business logic
│   │   ├── points.go            # Points business logic
│   │   └── matcher.go           # Name matching engine
│   ├── repository/
│   │   ├── subscriber.go        # Subscriber SQL queries (sqlx)
│   │   ├── donation.go          # Donation SQL queries (sqlx)
│   │   ├── points.go            # Points SQL queries (sqlx)
│   │   └── oauth.go             # OAuth token queries (sqlx)
│   ├── scheduler/
│   │   ├── youtube.go           # YouTube API poller (every 15 min)
│   │   ├── easydonate.go        # EasyDonate sync (every 5 min)
│   │   └── matcher.go           # Name matcher (every 2 min)
│   ├── client/
│   │   ├── youtube.go           # YouTube Data API client
│   │   └── easydonate.go        # EasyDonate API client
│   └── middleware/
│       ├── cors.go              # CORS configuration
│       ├── ratelimit.go         # Rate limiting
│       └── logger.go            # Request logging
├── migrations/
│   ├── 001_initial_schema.up.sql
│   └── 001_initial_schema.down.sql
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[021_architecture_decision_records]] | Decision rationale |
| [[022_API_specification]] | API contracts |
| [[023_database_schema_DDL]] | Database schema |
| [[024_ERD]] | Data model |
| [[029_architecture_overview]] | Stakeholder-friendly summary |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 42010
> **Usage:** The SAD is the *software-level* architecture document. Developers use it as their primary architectural reference.
