# Deerngo Bot — Viewer Relationship Management (VRM)

> **Project:** Deerngo Bot
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## What is Deerngo Bot?

A **Viewer Relationship Management (VRM)** system for the [@Deer_NGO](https://www.youtube.com/@Deer_NGO) YouTube channel. Think of it as a lightweight CRM for live stream viewers — tracking contributions (subscriptions, donations) and rewarding engagement through a points system visible in live chat and on a public scoreboard.

**Core idea:** 1 THB donated = 1 point. Points are matched to YouTube handles via fuzzy name matching. Viewers can check their points in chat with `:deer: point`.

---

## Tech Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| **Automation Engine** | streamer.bot | Windows desktop app, handles YouTube events + chat commands |
| **Backend** | Go | Business logic, API, database interaction |
| **Database** | PostgreSQL 18 | Existing homelab infrastructure |
| **Frontend** | React/Next.js | Public scoreboard (read-only, no auth) |
| **Donation Source** | EasyDonate API | [easydonate.app/deerngo0](https://easydonate.app/deerngo0) — webhook + REST API |

---

## Architecture

```mermaid
flowchart TB
    subgraph YouTube["YouTube Platform"]
        YT_Chat["Live Chat"]
        YT_Sub["Subscription Events"]
    end

    subgraph StreamerBot["Streamer.bot (Windows Local)"]
        SB_Trig["Triggers<br>Chat Message / New Sub"]
        SB_Actions["Actions<br>Send Chat / HTTP Request"]
    end

    subgraph Backend["Go Backend (Windows Local)"]
        API_Sub["/api/v1/subscribers"]
        API_Point["/api/v1/points/{handle}"]
        API_Score["/api/v1/scoreboard"]
        SyncEngine["EasyDonate Sync<br>+ Name Matching"]
    end

    subgraph DB["PostgreSQL 18 (Homelab)"]
        T_Sub["subscribers"]
        T_Don["donations"]
        T_Pts["viewer_points"]
    end

    subgraph EasyDonate["EasyDonate Platform"]
        ED_Webhook["Webhook<br>(donation events)"]
        ED_API["REST API<br>(donation history)"]
    end

    subgraph Frontend["React/Next.js Scoreboard"]
        Web_Score["Public Scoreboard Page"]
    end

    %% Subscriber flow
    YT_Sub --> SB_Trig
    SB_Trig -->|"HTTP POST"| API_Sub
    API_Sub --> T_Sub

    %% Chat command flow
    YT_Chat --> SB_Trig
    SB_Trig -->|":deer: donate"| SB_Actions
    SB_Trig -->|":deer: point"| API_Point
    API_Point --> T_Pts
    API_Point -->|"response"| SB_Actions
    SB_Actions -->|"chat message"| YT_Chat

    %% Donation flow
    ED_Webhook -->|"POST"| SyncEngine
    ED_API -->|"poll (fallback)"| SyncEngine
    SyncEngine --> T_Don
    SyncEngine --> T_Pts

    %% Scoreboard flow
    Web_Score -->|"GET"| API_Score
    API_Score --> T_Pts

    style YouTube fill:#FF0000,color:#fff
    style StreamerBot fill:#7B68EE,color:#fff
    style Backend fill:#2196F3,color:#fff
    style DB fill:#4CAF50,color:#fff
    style EasyDonate fill:#FF9800,color:#fff
    style Frontend fill:#00BCD4,color:#fff
```

---

## Phase 1 Scope (MVP)

### Features

| # | Feature | Description | Priority |
|---|---------|-------------|----------|
| 1 | **Register** | Auto-capture new YouTube subscribers to PostgreSQL via streamer.bot | 🔴 |
| 2 | **Bot — Donate** | `:deer: donate` → posts EasyDonate link in live chat | 🔴 |
| 3 | **Bot — Points** | `:deer: point` → shows viewer's points in live chat (1 THB = 1 pt) | 🔴 |
| 4 | **Web Scoreboard** | Public React page showing viewer points leaderboard | 🟡 |

### By the Numbers

| Metric | Value |
|--------|-------|
| Business Objectives | 4 |
| Epics | 4 (Register, Bot Commands, Points Engine, Scoreboard) |
| User Stories | 10 |
| Story Points | 34 |
| Acceptance Criteria | 46 (24 🔴 Must Have, 22 🟡 Should Have) |

### Sprint Plan

| Sprint | Stories | Focus |
|--------|---------|-------|
| Sprint 1 | US-001, US-002, US-010 | Subscriber registration + Donate command |
| Sprint 2 | US-011, US-012, US-020, US-021 | Point command + EasyDonate sync + Name matching |
| Sprint 3 | US-022, US-030, US-031 | Point query API + Scoreboard |

---

## Integration Points

### streamer.bot

| Capability | Port | Use Case |
|-----------|------|----------|
| HTTP Server | 7474 | `POST /DoAction` — trigger actions from Go backend |
| WebSocket Server | 8681 | Real-time bidirectional events |
| YouTube Triggers | — | Chat Message, New Subscriber (31 total) |
| User Global Variables | — | Per-user persistent state |
| Custom Webhook | — | External services can trigger actions |

### EasyDonate

| Capability | Use Case |
|-----------|----------|
| Webhook | Push donation events to Go backend (primary) |
| REST API | Poll donation history as fallback (60 req/min limit) |
| Auth | API key + OAuth 2.0 |

---

## Deployment

| Component | Location | Notes |
|-----------|----------|-------|
| streamer.bot | Local Windows PC | Same machine as streamer |
| Go Backend | Local Windows PC | Same machine as streamer.bot (localhost) |
| PostgreSQL 18 | Homelab | Existing infrastructure |
| React Frontend | Local Windows PC | Exposed via tunnel/port forwarding for public access |

---

## Future Phases (Not in Scope)

| Phase | Features |
|-------|----------|
| Phase 2 | Leaderboard, recognition (bot shouts top donors), milestones/tiers |
| Phase 3 | Point redemption, OBS overlay integration, advanced gamification |

---

## Spec Documents

| Document | Path | Status |
|----------|------|--------|
| Business Objectives | [011_business_objective.md](../external_spec/deerngo_bot/01_requirement/011_business_objective.md) | v0.1 Draft |
| User Stories | [012_user_stories.md](../external_spec/deerngo_bot/01_requirement/012_user_stories.md) | v0.1 Draft |
| Acceptance Criteria | [013_acceptance_criteria.md](../external_spec/deerngo_bot/01_requirement/013_acceptance_criteria.md) | v0.1 Draft |

---

> **Original idea (Thai notes):** ไม่อยากทำยุ่งยากมาก — ผูกกะ streamer.bot, ให้ยูสพิมพ์คำสั่ง `:deer: donate` ขึ้นลิ้งโดเนท, `:deer: point` ขึ้นคะแนน, 1 บาท = 1 คะแนน, ชื่อ subscriber ต้องตรงกับชื่อ donate, แสดงผลในเว็บ
