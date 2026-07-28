---
document_type: User Stories
version: "0.1"
status: Draft
author: "PO"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
ba_owner: "PO"
po_owner: "PO"
classification: "Internal"
tags: [user-stories, agile, backlog, vrm, viewer-relationship-management]
standard_ref:
  - SWEBOK v4 — Requirements
  - ISO/IEC/IEEE 29148 — Requirements Engineering
---

# User Stories

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | PO |
| Product Owner | PO |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | PO | Initial draft from stakeholder grill session |

---

## 1. Purpose

This document captures requirements for Deerngo Bot Phase 1 as user stories. Each story is traceable to a business objective and includes acceptance criteria, story points, and priority.

**Tech Context:**
- **Automation Engine:** streamer.bot (Windows, local) — handles YouTube events and chat commands
- **Backend:** Go service — business logic, API, database interaction
- **Database:** PostgreSQL 18 (existing homelab)
- **Frontend:** React/Next.js — public scoreboard
- **Donation Source:** EasyDonate API (easydonate.app/deerngo0)
- **YouTube Channel:** @Deer_NGO

## 2. User Story Standards

### 2.1 INVEST Criteria

| Criterion | Description | Check |
|-----------|-------------|-------|
| **I**ndependent | Story can be developed independently | ✅ |
| **N**egotiable | Details can be discussed and refined | ✅ |
| **V**aluable | Delivers value to a user or business | ✅ |
| **E**stimable | Team can estimate effort | ✅ |
| **S**mall | Fits within a sprint | ✅ |
| **T**estable | Has clear acceptance criteria | ✅ |

### 2.2 Story Format

```markdown
**As a** [role/persona]
**I want** [feature/capability]
**So that** [benefit/value]

**Acceptance Criteria:**
Given [precondition]
When [action]
Then [outcome]

**Story Points:** [1, 2, 3, 5, 8, 13]
**Priority:** [🔴 Must Have / 🟡 Should Have / 🟢 Could Have]
**Epic:** [Parent epic]
**Sprint:** [Target sprint]
```

---

## 3. Epic Overview

| Epic ID | Epic Name | Stories | Total Points | Sprint |
|---------|-----------|---------|-------------|--------|
| E-01 | Register (Subscriber Capture) | 3 | 12 | Sprint 1 |
| E-02 | Bot — Commands | 3 | 8 | Sprint 1-2 |
| E-03 | Points Engine | 3 | 13 | Sprint 2-3 |
| E-04 | Web Scoreboard | 2 | 8 | Sprint 3 |
| **Total** | | **11** | **41** | |

---

## 4. User Stories

### Epic E-01: Register (Subscriber Capture)

#### US-001: Capture New Subscriber (Hybrid — YouTube API + streamer.bot)

**As a** streamer (Deer_NGO)
**I want** every new YouTube subscriber to be automatically saved to my database — whether I'm live or not
**So that** I can track my community growth 24/7 and use subscriber data for the points system

**Acceptance Criteria:**
- **AC-1:** Given a viewer subscribes to @Deer_NGO during a live stream, When streamer.bot fires the subscription event, Then the Go backend receives the event and creates/updates a subscriber record with YouTube handle, display name, and subscription timestamp (real-time, <5s)
- **AC-2:** Given a viewer subscribes outside of live streaming hours (streamer.bot offline), When the YouTube API polling scheduler runs (every 15 minutes), Then the Go backend fetches new subscribers from YouTube Data API and creates/updates records in PostgreSQL
- **AC-3:** Given a subscriber already exists in the database, When the same subscriber is captured again (from either source), Then the existing record is updated (upsert by youtube_handle) — no duplicates
- **AC-4:** Given both sources capture the same subscriber (streamer.bot during live + API polling), When the upsert runs, Then the record is updated with the earliest subscription timestamp (no data conflict)

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-01
**Sprint:** Sprint 1
**Status:** Draft
**Objective:** OBJ-01

---

#### US-002: Subscriber Registration API

**As a** developer
**I want** a REST API endpoint that accepts subscriber registration events from streamer.bot
**So that** real-time subscriber data can be pushed to the backend during live streams

**Acceptance Criteria:**
- **AC-1:** Given a POST request to `/api/v1/subscribers` with valid payload (youtube_handle, display_name, subscribed_at, source), When the backend processes it, Then a 201 Created response is returned with the subscriber record
- **AC-2:** Given a POST request with a duplicate youtube_handle, When the backend processes it, Then the existing record is updated (upsert) and a 200 OK response is returned
- **AC-3:** Given a POST request with missing required fields, When validation runs, Then a 400 Bad Request response is returned with specific field-level errors

**Story Points:** 2
**Priority:** 🔴 Must Have
**Epic:** E-01
**Sprint:** Sprint 1
**Status:** Draft
**Objective:** OBJ-01

---

#### US-003: YouTube API Polling Scheduler

**As a** system
**I want** to periodically poll the YouTube Data API for new subscribers
**So that** subscribers who join outside of live streaming hours are captured (24/7 coverage)

**Acceptance Criteria:**
- **AC-1:** Given the Go backend is running, When the polling scheduler triggers (every 15 minutes), Then it calls YouTube Data API `GET /youtube/v3/subscriptions` with the channel's OAuth token
- **AC-2:** Given the API returns new subscribers not in the database, When the backend processes them, Then each new subscriber is inserted with youtube_handle, display_name, subscribed_at, and source="youtube_api"
- **AC-3:** Given the API returns subscribers already in the database, When the upsert runs, Then existing records are updated (no duplicates) with the earliest subscription timestamp preserved
- **AC-4:** Given the YouTube API returns a quota exceeded error (403), When the scheduler detects it, Then it logs the error and skips the next poll cycle (quota = 10,000 units/day, polling every 15 min = 96 calls/day = well within limit)
- **AC-5:** Given the OAuth token has expired, When the API returns 401 Unauthorized, Then the backend logs the error and alerts the operator (manual re-auth required)

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-01
**Sprint:** Sprint 1
**Status:** Draft
**Objective:** OBJ-01

---

### Epic E-02: Bot — Commands

#### US-010: Donate Command

**As a** viewer watching @Deer_NGO's live stream
**I want** to type `:deer: donate` in chat and see the donation link
**So that** I can easily find and access the donation page without leaving the stream

**Acceptance Criteria:**
- **AC-1:** Given a viewer types `:deer: donate` in YouTube live chat, When streamer.bot detects the command, Then the bot responds with "🦌 Donate here: https://easydonate.app/deerngo0" within 2 seconds
- **AC-2:** Given the bot is offline or disconnected, When a viewer types `:deer: donate`, Then no response is sent (graceful degradation — no error spam)
- **AC-3:** Given multiple viewers type `:deer: donate` simultaneously, When streamer.bot processes them, Then each command gets a response (no deduplication — each viewer gets the link)

**Story Points:** 2
**Priority:** 🔴 Must Have
**Epic:** E-02
**Sprint:** Sprint 1
**Status:** Draft
**Objective:** OBJ-02

---

#### US-011: Point Command

**As a** viewer watching @Deer_NGO's live stream
**I want** to type `:deer: point` in chat and see my current points
**So that** I can track my contribution level and compare with other viewers

**Acceptance Criteria:**
- **AC-1:** Given a viewer has donated and has points, When they type `:deer: point` in chat, Then the bot responds with "🦌 @username has X points!" within 2 seconds
- **AC-2:** Given a viewer has never donated (0 points), When they type `:deer: point` in chat, Then the bot responds with "🦌 @username has 0 points. Donate to earn points!"
- **AC-3:** Given the backend API is down, When a viewer types `:deer: point`, Then the bot responds with a friendly error message (e.g., "🦌 Points system is temporarily unavailable")

**Story Points:** 3
**Priority:** 🔴 Must Have
**Epic:** E-02
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

#### US-012: Streamer.bot Action Configuration

**As a** streamer (Deer_NGO)
**I want** streamer.bot actions configured for `:deer: donate` and `:deer: point` commands
**So that** the bot responds automatically without manual intervention

**Acceptance Criteria:**
- **AC-1:** Given streamer.bot is running, When a chat message contains `:deer: donate`, Then the "Donate" action fires and sends the donation link
- **AC-2:** Given streamer.bot is running, When a chat message contains `:deer: point`, Then the "Point" action fires, calls the Go backend API, and sends the point response
- **AC-3:** Given the Go backend returns an error for the point query, When streamer.bot receives the error, Then it sends a fallback error message instead of crashing

**Story Points:** 3
**Priority:** 🔴 Must Have
**Epic:** E-02
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-02, OBJ-03

---

### Epic E-03: Points Engine

#### US-020: EasyDonate Donation Sync

**As a** system
**I want** to fetch donation data from EasyDonate API and store it in PostgreSQL
**So that** the points system has accurate, up-to-date donation records

**Acceptance Criteria:**
- **AC-1:** Given the backend polls EasyDonate API, When new donations are found, Then each donation is stored with: donor_name, amount_thb, donation_time, easydonate_id
- **AC-2:** Given a donation already exists (by easydonate_id), When the sync runs, Then the existing record is not duplicated (idempotent sync)
- **AC-3:** Given EasyDonate API returns a rate limit error (429), When the sync process detects it, Then it backs off and retries after the rate limit window

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-03
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

#### US-021: Name Matching Engine

**As a** system
**I want** to match EasyDonate donor names to YouTube subscriber handles using fuzzy matching
**So that** donations are correctly attributed to viewers even if names don't exactly match

**Acceptance Criteria:**
- **AC-1:** Given a donation from "deer123" and a subscriber "@deer123", When the matching engine runs, Then the donation is matched to the subscriber (fuzzy match — strip @, case-insensitive)
- **AC-2:** Given a donation from "anonymous" or empty name, When the matching engine runs, Then the donation is flagged as "unmatched" and does not award points
- **AC-3:** Given multiple subscribers could match a donation name, When the matching engine runs, Then the best match is selected (highest similarity score) and the match is logged for review
- **AC-4:** Given a donation name has no close match in subscribers, When the matching engine runs, Then the donation is flagged as "unmatched" with the donor_name stored for manual review

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-03
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

#### US-022: Point Calculation & Query API

**As a** system
**I want** to calculate points (1 THB = 1 point) and provide a query API
**So that** the bot can retrieve a viewer's point balance on demand

**Acceptance Criteria:**
- **AC-1:** Given a viewer has 3 matched donations totaling 500 THB, When the point query API is called with their YouTube handle, Then the response includes total_points: 500
- **AC-2:** Given a viewer has no donations, When the point query API is called, Then the response includes total_points: 0
- **AC-3:** Given the API receives a GET request to `/api/v1/points/{youtube_handle}`, When the handle exists, Then a 200 OK response is returned with the point balance
- **AC-4:** Given the API receives a GET request with a non-existent handle, When queried, Then a 200 OK response is returned with total_points: 0 (not a 404)

**Story Points:** 3
**Priority:** 🔴 Must Have
**Epic:** E-03
**Sprint:** Sprint 3
**Status:** Draft
**Objective:** OBJ-03

---

### Epic E-04: Web Scoreboard

#### US-030: Public Scoreboard Page

**As a** viewer
**I want** to see a public scoreboard showing top contributors by points
**So that** I can see my ranking and feel motivated to contribute more

**Acceptance Criteria:**
- **AC-1:** Given a viewer visits the scoreboard URL, When the page loads, Then they see a ranked list of viewers by points (highest first), showing display name and point total
- **AC-2:** Given new donations are processed, When the scoreboard is refreshed, Then the updated point totals are displayed (data freshness <60s)
- **AC-3:** Given the scoreboard is accessed from any device, When the page loads, Then it renders responsively (mobile-friendly)
- **AC-4:** Given the backend API is down, When a viewer visits the scoreboard, Then a friendly error message is shown (e.g., "Scoreboard temporarily unavailable")

**Story Points:** 5
**Priority:** 🟡 Should Have
**Epic:** E-04
**Sprint:** Sprint 3
**Status:** Draft
**Objective:** OBJ-04

---

#### US-031: Scoreboard API Endpoint

**As a** frontend developer
**I want** a REST API endpoint that returns the ranked scoreboard data
**So that** the React frontend can fetch and display it

**Acceptance Criteria:**
- **AC-1:** Given a GET request to `/api/v1/scoreboard`, When the backend processes it, Then a 200 OK response is returned with a JSON array of viewers sorted by points (descending), each with: rank, display_name, youtube_handle, total_points
- **AC-2:** Given the scoreboard has 0 viewers with points, When queried, Then an empty array is returned (not an error)
- **AC-3:** Given the API supports pagination, When a request includes `?page=1&limit=50`, Then only the top 50 viewers are returned

**Story Points:** 3
**Priority:** 🟡 Should Have
**Epic:** E-04
**Sprint:** Sprint 3
**Status:** Draft
**Objective:** OBJ-04

---

## 5. Story Estimation Summary

| Epic | Stories | Total Points | Sprint Allocation |
|------|---------|-------------|------------------|
| E-01 Register | 3 | 12 | Sprint 1 |
| E-02 Bot Commands | 3 | 8 | Sprint 1-2 |
| E-03 Points Engine | 3 | 13 | Sprint 2-3 |
| E-04 Web Scoreboard | 2 | 8 | Sprint 3 |
| **Total** | **11** | **41** | |

## 6. Story Map

| | Sprint 1 | Sprint 2 | Sprint 3 |
|--|---------|---------|---------|
| **E-01 Register** | US-001, US-002, US-003 | | |
| **E-02 Bot Commands** | US-010 | US-011, US-012 | |
| **E-03 Points Engine** | | US-020, US-021 | US-022 |
| **E-04 Scoreboard** | | | US-030, US-031 |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[011_business_objective]] | Stories trace to business objectives |
| [[013_acceptance_criteria]] | Detailed ACs in Given/When/Then format |
| [[014_stakeholder_analysis]] | Stakeholders and their stories |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29148
> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
