---
document_type: Architecture Overview (HLD)
version: "0.2"
status: Draft
author: "PO / SA"
created: "2026-07-29"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
architect: "SA / Dev"
classification: "Internal"
tags: [architecture-overview, hld, high-level-design, members, privacy, vrm]
standard_ref:
  - SWEBOK v4 — Design
  - ISO/IEC/IEEE 42010 — Architecture Description
parent_project: "Deerngo Bot — VRM"
---

# Architecture Overview

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** Phase 1 uses explicit `:deer: register` membership. YouTube subscriber polling and OAuth are no longer active MVP components.

---

## 1. Purpose

This document provides the high-level architecture map for the current Deerngo Bot MVP: live-chat member registration, EasyDonate donation ingestion, exact member matching, points, and a privacy-aware public scoreboard.

## 2. Architecture at a Glance

```mermaid
flowchart LR
    subgraph LocalPC["Windows PC"]
        SB["Streamer.bot"]
        YT["YouTube Live Chat"]
    end

    subgraph Homelab["Homelab Server (Docker)"]
        API["Go Backend<br>:8008"]
        WEB["Next.js Frontend<br>:3008"]
        DB[("PostgreSQL 18")]
        MEMBERS["Member Service"]
        POINTS["Points Service"]
        INGEST["EasyDonate Ingestion"]
        CF["Cloudflare Tunnel"]
    end

    subgraph EasyDonate["EasyDonate"]
        ED_WEBHOOK["Webhook"]
        ED_API["REST API fallback"]
    end

    YT --> SB
    SB -->|"register/public/private/point"| API
    API --> MEMBERS
    API --> POINTS
    API --> DB
    ED_WEBHOOK -->|"public webhook path"| INGEST
    ED_API -->|"API key polling fallback"| INGEST
    INGEST --> DB
    WEB -->|"public scoreboard API"| API
    CF --> WEB

    style LocalPC fill:#7B68EE,color:#fff
    style Homelab fill:#2196F3,color:#fff
    style EasyDonate fill:#FF9800,color:#fff
```

---

## 3. Component Map

| Component | Technology | Port | Location | Purpose |
|-----------|-----------|------|----------|---------|
| **Go Backend** | Go / Fiber / sqlx | :8008 | Homelab Docker | Member API, donation ingestion, points, scoreboard |
| **Next.js Frontend** | Next.js / Tailwind / DaisyUI | :3008 | Homelab Docker | Public contributor scoreboard |
| **PostgreSQL 18** | PostgreSQL | :5432 | Homelab Docker | Members, private donations, points |
| **Streamer.bot** | Windows app | :7474, :8681 | Local Windows PC | Live chat commands and actual chat identity |
| **EasyDonate** | Webhook + REST API | HTTPS | External | Donation events and fallback reconciliation |
| **Cloudflare Tunnel** | cloudflared | — | Homelab systemd | Public web and configured webhook access |

---

## 4. Data Flows

### 4.1 Member Registration

```mermaid
sequenceDiagram
    participant V as Viewer
    participant YT as YouTube Chat
    participant SB as streamer.bot
    participant Go as Go Backend
    participant DB as PostgreSQL

    V->>YT: ":deer: register"
    YT->>SB: Chat event with actual userId + current handle
    SB->>Go: POST /api/v1/members/register
    Go->>DB: Create/identify active member
    DB-->>Go: Member with 0/current points
    Go-->>SB: Registration result
    SB->>YT: Confirmation or error message
```

### 4.2 Visibility Commands

```mermaid
sequenceDiagram
    participant V as Viewer
    participant SB as streamer.bot
    participant Go as Go Backend
    participant DB as PostgreSQL

    V->>SB: ":deer: public" or ":deer: private"
    SB->>Go: PUT /api/v1/members/{userId}/visibility
    Go->>DB: Update public_visibility
    DB-->>Go: Updated status
    Go-->>SB: Confirmation
    SB-->>V: Chat response
```

### 4.3 Donation to Points

```mermaid
sequenceDiagram
    participant ED as EasyDonate
    participant Go as Go Backend
    participant DB as PostgreSQL

    Note over ED,DB: Primary: provider webhook
    ED->>Go: POST /api/v1/webhooks/easydonate/{path_token}
    Go->>DB: Insert by referenceNo (private)

    Note over ED,DB: Fallback: API polling
    Go->>ED: GET donations with API key
    ED-->>Go: Recent donations
    Go->>DB: Insert by referenceNo (idempotent)

    Note over Go,DB: Exact member match
    Go->>DB: Normalize donor name
    Go->>DB: Find active member, handle exact match, donation_time >= registered_at
    Go->>DB: Increment members.total_points once
```

### 4.4 Chat Point Query

```mermaid
sequenceDiagram
    participant V as Viewer
    participant SB as streamer.bot
    participant Go as Go Backend
    participant DB as PostgreSQL

    V->>SB: ":deer: point"
    SB->>Go: GET /api/v1/members/{userId}/points
    Go->>DB: Find active member
    DB-->>Go: Points + visibility
    Go-->>SB: Exact total or 100-point range
    SB-->>V: Public chat response
```

### 4.5 Public Scoreboard

```mermaid
sequenceDiagram
    participant Browser as Public Browser
    participant Web as Next.js
    participant Go as Go Backend
    participant DB as PostgreSQL

    Browser->>Web: Open scoreboard
    Web->>Go: GET /api/v1/scoreboard
    Go->>DB: active AND public AND points > 0
    DB-->>Go: handle + points only
    Go-->>Web: Public projection
    Web-->>Browser: Ranked scoreboard
```

---

## 5. Port Map

| Port | Service | Location | Access |
|------|---------|----------|--------|
| 3008 | Next.js frontend | Homelab Docker | Cloudflare Tunnel → public |
| 5432 | PostgreSQL | Homelab Docker | Internal only |
| 7474 | streamer.bot HTTP | Windows PC | Local integration only |
| 8008 | Go backend | Homelab Docker | LAN from streamer.bot + Docker network |
| 8681 | streamer.bot WebSocket | Windows PC | Local integration only |

## 6. Current Technology Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Backend | Go + Fiber + sqlx | API and business logic |
| Database | PostgreSQL 18 | Member/donation/points persistence |
| Frontend | Next.js + Tailwind + DaisyUI | Public scoreboard |
| Live automation | streamer.bot | Identity and chat commands |
| Donations | EasyDonate webhook + API fallback | Donation ingestion |
| Public access | Cloudflare Tunnel | HTTPS exposure |

## 7. Privacy and Data Boundaries

### Stored Privately

- Stable streamer.bot/YouTube user ID
- Normalized current handle
- Active/inactive membership status
- Registration timestamp
- Point totals and donation counts
- Raw EasyDonate donor name, reference, amount, time, and message for reconciliation

### Publicly Exposed

Only active, public members with points greater than zero:

```text
rank
youtube_handle
total_points
```

No YouTube display name, stable user ID, raw donor name, donation message, private member, inactive member, or zero-point member is returned by the scoreboard API.

## 8. Sprint Plan

| Sprint | Stories | Focus |
|--------|---------|-------|
| Sprint 1 | US-001, US-002, US-010 | Member registration + donate command |
| Sprint 2 | US-011, US-012, US-020, US-021, US-022 | Visibility/point commands + EasyDonate ingestion + exact matching + points |
| Sprint 3 | US-030, US-031 | Public scoreboard and release hardening |

## 9. Superseded Components

| Component | Status | Reason |
|-----------|--------|--------|
| YouTube subscriber polling | Removed from active MVP | API exposed only a limited subscriber subset |
| YouTube OAuth token storage | Removed from active MVP | No YouTube polling in current design |
| `subscribers` table | Superseded | Explicit members are the source of eligibility |
| `viewer_points` table | Superseded | `members.total_points` is the MVP summary |
| `pg_trgm` fuzzy matching | Removed from active MVP | Normalized exact match is safer and simpler |
| HMAC EasyDonate assumption | Unverified | Current provider docs do not confirm HMAC/signature header |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[021_architecture_decision_records]] | Decision rationale; update required |
| [[022_API_specification]] | Current API contracts |
| [[023_database_schema_DDL]] | Current physical data model |
| [[024_ERD]] | Current logical data model |
| [[011_business_objective]] | Current business objectives |
| [[012_user_stories]] | Current user stories |
| [[013_acceptance_criteria]] | Current criteria |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Scope-change decision |

---

> **Template Standard:** Based on SWEBOK v4 and ISO/IEC/IEEE 42010
> **Usage:** Current entry point for the member-based Phase 1 architecture.
