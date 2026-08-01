# Deerngo Bot — Viewer Relationship Management (VRM)

> **Project:** Deerngo Bot
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Phase 1 change:** YouTube subscriber polling was removed after live verification showed that the YouTube API exposes only a limited subscriber subset. Viewers now explicitly join the VRM program with `:deer: register`.

---

## What is Deerngo Bot?

A **Viewer Relationship Management (VRM)** system for the [@Deer_NGO](https://www.youtube.com/@Deer_NGO) YouTube channel. It is a lightweight community points program for viewers who explicitly register during live chat.

**Core idea:**

```text
Viewer types :deer: register
  → streamer.bot sends the actual chat identity
  → Go backend creates an active member with 0 points
  → future qualifying EasyDonate donations earn 1 point per THB
```

A viewer's donation name must match their normalized registered YouTube handle:

```text
trim whitespace → remove leading @ → lowercase
```

No fuzzy matching is used in the revised MVP.

---

## Tech Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| **Automation Engine** | streamer.bot | Windows desktop app; receives chat commands and provides the actual chat identity |
| **Backend** | Go | Member API, donation ingestion, exact matching, points, scoreboard API |
| **Database** | PostgreSQL 18 | Existing homelab infrastructure |
| **Frontend** | React/Next.js | Public read-only scoreboard |
| **Donation Source** | EasyDonate | Webhook primary + REST API polling fallback |
| **Public Access** | Cloudflare Tunnel | Scoreboard and configured webhook route |

---

## Architecture

```mermaid
flowchart TB
    subgraph LivePC["Streamer's Windows PC"]
        YT["YouTube Live Chat"]
        SB["streamer.bot"]
    end

    subgraph Backend["Go Backend — Homelab Docker :8008"]
        MemberAPI["Member API<br>register / public / private"]
        PointAPI["Points API"]
        ScoreAPI["Scoreboard API"]
        Ingest["EasyDonate Ingestion"]
        Match["Normalized Exact Matching"]
    end

    subgraph DB["PostgreSQL 18 — deerngo"]
        Members["members"]
        Donations["donations"]
        Notes["point_adjustment_notes"]
    end

    subgraph EasyDonate["EasyDonate"]
        Webhook["Webhook"]
        API["REST API fallback"]
    end

    subgraph Web["Next.js — Homelab Docker :3008"]
        Board["Public Scoreboard"]
    end

    YT --> SB
    SB -->|"HTTP over LAN"| MemberAPI
    SB -->|"HTTP over LAN"| PointAPI
    SB -->|"HTTP over LAN"| ScoreAPI
    MemberAPI --> Members
    PointAPI --> Members
    ScoreAPI --> Members
    Webhook -->|"public path"| Ingest
    API -->|"API key polling"| Ingest
    Ingest --> Donations
    Donations --> Match
    Match --> Members
    Members --> Notes
    Board -->|"GET"| ScoreAPI

    style LivePC fill:#7B68EE,color:#fff
    style Backend fill:#2196F3,color:#fff
    style DB fill:#4CAF50,color:#fff
    style EasyDonate fill:#FF9800,color:#fff
    style Web fill:#00BCD4,color:#fff
```

---

## Phase 1 Scope (Revised MVP)

| # | Feature | Description | Priority |
|---|---------|-------------|----------|
| 1 | **Member Register** | `:deer: register` creates a member from the actual streamer.bot chat identity | 🔴 |
| 2 | **Bot — Donate** | `:deer: donate` posts the EasyDonate link in live chat | 🔴 |
| 3 | **Bot — Points** | `:deer: point` shows exact points for public members or a 100-point range for private members | 🔴 |
| 4 | **Visibility** | `:deer: public` / `:deer: private` switch scoreboard visibility | 🔴 |
| 5 | **Donation Points** | EasyDonate webhook + API fallback; normalized exact donor-name matching; 1 THB = 1 point | 🔴 |
| 6 | **Web Scoreboard** | Shows only active, public members with points > 0; handle and points only | 🟡 |

### Explicitly Removed from Phase 1

- YouTube Data API subscriber polling
- Automatic subscriber registration
- YouTube OAuth client/refresh-token dependency
- Historical subscriber backfill
- `subscribers` table as a points source
- `pg_trgm` fuzzy name matching
- Automatic historical donation crediting
- Admin console

### Member Rules

| Rule | Behavior |
|------|----------|
| New registration | Active member, 0 points, public visibility enabled |
| Same-handle registration | Friendly already-registered response; no DB change |
| New handle for same user | Old member becomes inactive; old points stay; new active member starts at 0 |
| Active handle conflict | Reject registration; owner resolves manually |
| Pre-registration donation | Stored privately, never credited automatically |
| Matching | Normalized exact donor-name → active member handle |
| Private member | Keeps earning points; hidden from scoreboard; point command shows a 100-point range |
| Scoreboard | Active + public + points > 0 only |
| Display name | Not stored; public API exposes normalized handle only |

---

## Chat Commands

| Command | Registered? | Result |
|---------|:-----------:|--------|
| `:deer: register` | No | Creates member from actual chat identity |
| `:deer: donate` | No | Posts EasyDonate link |
| `:deer: point` | Yes | Exact points if public; 100-point band if private |
| `:deer: public` | Yes | Enables public scoreboard visibility |
| `:deer: private` | Yes | Hides member from scoreboard while keeping points active |

### Example Responses

```text
New member:
🦌 Registered! You start with 0 points. When you earn points, your normalized YouTube handle and score may appear on the public scoreboard.

Unregistered point query:
🦌 You are not registered yet. Use :deer: register first.

Private member with 563 points:
🦌 Your points are between 500–600.

Backend unavailable:
🦌 Registration is temporarily unavailable. Please try again later.
```

---

## Integration Points

### streamer.bot

| Capability | Use Case |
|-----------|----------|
| YouTube Chat Message trigger | Detect `:deer:` commands |
| YouTube user variables | Supply actual `userId` and current username/handle |
| HTTP Request action | Call Go backend over LAN |
| Send Chat Message action | Reply in YouTube chat |
| Action logs | Record trigger and result without secrets |

### EasyDonate

| Capability | Use Case |
|-----------|----------|
| Webhook | Primary donation event ingestion |
| REST API | Fallback reconciliation using a backend-only API key with donation-read scope |
| Webhook security | Use provider-confirmed mechanism; current public docs do not confirm HMAC |
| Idempotency | Use provider `referenceNo` |

---

## Data and Privacy Boundary

### Stored Privately

- Stable streamer.bot/YouTube user ID
- Normalized current handle
- Member status and visibility
- Registration time and point total
- Raw EasyDonate donor name, amount, time, reference, and message for reconciliation

### Publicly Exposed

Only active, public members with points greater than zero:

```text
rank
youtube_handle
total_points
```

The public API does not expose display names, stable user IDs, raw donor names, donation messages, inactive members, private members, or zero-point members.

> A YouTube handle can still be personal data under Thai PDPA. Registration notice, public visibility behavior, privacy/opt-out handling, and owner/operator responsibilities must be documented before public release.

---

## Repository and Deployment

| Repository | Content | Port |
|------------|---------|:----:|
| [`oat431/deerngo-bot`](https://github.com/oat431/deerngo-bot) | Go backend, migrations, integration contracts | 8008 |
| [`oat431/deerngo-web`](https://github.com/oat431/deerngo-web) | Next.js scoreboard | 3008 |

| Component | Location |
|-----------|----------|
| streamer.bot | Streamer's Windows PC |
| Go backend | Homelab Docker, `db-network` |
| PostgreSQL | Existing homelab PostgreSQL 18, `deerngo` database |
| Next.js | Homelab Docker, `db-network` |
| Cloudflare Tunnel | Homelab systemd service |

---

## Sprint Plan (Revised)

| Sprint | Stories | Focus |
|--------|---------|-------|
| Sprint 1 | US-001, US-002, US-010 | Member registration + donate command |
| Sprint 2 | US-011, US-012, US-020, US-021, US-022 | Visibility/point commands + EasyDonate ingestion + exact matching + points |
| Sprint 3 | US-030, US-031 | Public scoreboard and release hardening |

### Superseded Work

US-003 and its merged YouTube polling implementation are retained as historical technical work, but are no longer part of the active Phase 1 product scope. See `07_pm/072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801.md`.

---

## Spec Documents

| Document | Path | Version |
|----------|------|:-------:|
| Business Objectives | `01_requirement/011_business_objective.md` | 0.2 |
| User Stories | `01_requirement/012_user_stories.md` | 0.2 |
| Acceptance Criteria | `01_requirement/013_acceptance_criteria.md` | 0.2 |
| API Specification | `02_design/022_API_specification.md` | 0.2 |
| Database Schema | `02_design/023_database_schema_DDL.md` | 0.2 |
| ERD | `02_design/024_ERD.md` | 0.2 |
| Architecture Overview | `02_design/029_architecture_overview.md` | 0.2 |
| Scope Decision | `07_pm/072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801.md` | Final |

---

> **Original idea (Thai notes):** ไม่อยากทำยุ่งยากมาก — ผูกกะ streamer.bot, ให้ยูสพิมพ์คำสั่ง `:deer: donate` ขึ้นลิ้งโดเนท, `:deer: point` ขึ้นคะแนน, 1 บาท = 1 คะแนน, ชื่อ subscriber ต้องตรงกับชื่อ donate, แสดงผลในเว็บ
> **Current product decision:** Use explicit registered members, not automatic subscriber collection.
---
