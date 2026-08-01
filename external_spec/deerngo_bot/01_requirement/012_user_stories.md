---
document_type: User Stories
version: "0.2"
status: Draft
author: "PO"
created: "2026-07-29"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
ba_owner: "PO"
po_owner: "PO"
classification: "Internal"
tags: [user-stories, agile, backlog, vrm, viewer-relationship-management, members]
standard_ref:
  - SWEBOK v4 — Requirements
  - ISO/IEC/IEEE 29148 — Requirements Engineering
---

# User Stories

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Change note:** Phase 1 no longer imports YouTube subscribers. Viewers explicitly join the VRM program with `:deer: register` during live chat. The former YouTube polling story (US-003) is superseded by MM06.

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | PO |
| Product Owner | PO |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | PO | Initial subscriber-based draft |
| 0.2 | 2026-08-02 | PO | Replaced subscriber capture with explicit member registration; added privacy and visibility behavior |

---

## 1. Purpose

This document captures requirements for Deerngo Bot Phase 1 as user stories. Phase 1 now focuses on an explicit viewer membership program rather than incomplete YouTube subscriber-list collection.

**Tech Context:**
- **Automation Engine:** streamer.bot (Windows, local) — handles YouTube live chat and commands
- **Backend:** Go service — member registration, donation ingestion, matching, points, and API
- **Database:** PostgreSQL 18 (existing homelab)
- **Frontend:** React/Next.js — public scoreboard
- **Donation Source:** EasyDonate webhook (primary) + EasyDonate API polling (fallback)
- **YouTube Channel:** @Deer_NGO

### Member Rules

- The real chat identity comes from streamer.bot; a handle typed in chat is ignored.
- A new member starts with 0 points.
- A member's public visibility defaults to enabled, but only active members with points greater than 0 appear on the scoreboard.
- `:deer: public` and `:deer: private` switch scoreboard visibility.
- Private members keep earning points and can query an approximate point band in public chat.
- Donations earn points only when `donation_time >= registered_at` and the normalized donor name exactly matches an active member handle.
- Normalization means trim whitespace, remove one leading `@`, and lowercase.
- No YouTube display name is stored in the member record.
- Historical subscriber import and automatic YouTube polling are out of scope for this MVP.

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

---

## 3. Epic Overview

| Epic ID | Epic Name | Stories | Total Points | Sprint |
|---------|-----------|---------|-------------|--------|
| E-01 | Member Registration | 2 | 8 | Sprint 1 |
| E-02 | Bot Commands | 3 | 8 | Sprint 1-2 |
| E-03 | Points Engine | 3 | 13 | Sprint 2 |
| E-04 | Web Scoreboard | 2 | 8 | Sprint 3 |
| **Total** | | **10** | **37** | |

### Superseded Story

| Story | Status | Reason |
|-------|--------|--------|
| US-003 — YouTube API Polling Scheduler | ❌ Superseded | YouTube API exposed only a limited subscriber subset; explicit member registration is now the MVP identity event. See `07_pm/072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801.md`. |

---

## 4. User Stories

### Epic E-01: Member Registration

#### US-001: Register Viewer as Member

**As a** viewer watching @Deer_NGO's live stream
**I want** to register myself with `:deer: register`
**So that** I can join the points program and earn points from future qualifying donations.

**Acceptance Criteria:**
- **AC-001a:** Given streamer.bot receives `:deer: register` from a viewer, When it sends the actual chat author's YouTube user ID and current handle to the backend, Then an active member is created with 0 points, `public_visibility=true`, `registered_at`, and no stored YouTube display name.
- **AC-001b:** Given an active member registers again with the same normalized handle and user ID, When registration is processed, Then no database record or point total changes and the bot responds with the member's current points.
- **AC-001c:** Given an active member registers again with a changed normalized handle, When registration is processed, Then the old member becomes inactive with its points preserved and a new active member is created with 0 points; no points transfer occurs automatically.
- **AC-001d:** Given a new registration uses a normalized handle already held by another active YouTube user ID, When registration is processed, Then registration is rejected and the bot reports that the handle is already in use.
- **AC-001e:** Given a viewer types a different handle in the command text, When streamer.bot sends the event, Then the backend ignores the typed value and uses the actual identity supplied by streamer.bot.
- **AC-001f:** Given a new member is created, When the bot confirms registration, Then it says: `🦌 Registered! You start with 0 points. When you earn points, your normalized YouTube handle and score may appear on the public scoreboard.`
- **AC-001g:** Given the Go backend is unavailable but streamer.bot is running, When a viewer registers, Then the bot responds that registration is temporarily unavailable and no partial member is created.

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-01
**Sprint:** Sprint 1
**Status:** Draft
**Objective:** OBJ-01

---

#### US-002: Member Registration API

**As a** developer
**I want** an API endpoint that registers a viewer using streamer.bot's actual identity
**So that** member creation and re-registration are handled consistently by the backend.

**Acceptance Criteria:**
- **AC-002a:** Given a valid request containing `youtube_user_id` and `youtube_handle`, When `POST /api/v1/members/register` processes it, Then a `201 Created` response returns the new active member with 0 points.
- **AC-002b:** Given an active member with the same `youtube_user_id` and normalized handle exists, When the endpoint is called again, Then it returns `200 OK` with an already-registered result and does not change the database.
- **AC-002c:** Given an active member with the same `youtube_user_id` registers a different handle, When the endpoint is called, Then the old row becomes inactive and a new active row is created with 0 points.
- **AC-002d:** Given the normalized handle belongs to a different active `youtube_user_id`, When the endpoint is called, Then it returns `409 Conflict` with a handle-in-use error.
- **AC-002e:** Given `youtube_user_id` or `youtube_handle` is missing or invalid, When validation runs, Then the endpoint returns `400 Bad Request` with field-level errors.
- **AC-002f:** Given a valid handle contains whitespace or a leading `@`, When normalized, Then storage and matching use trimmed lowercase text without the leading `@`.

**Story Points:** 3
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
**So that** I can access the donation page without leaving the stream.

**Acceptance Criteria:**
- **AC-010a:** Given streamer.bot is running and connected to chat, When a viewer types `:deer: donate`, Then the bot responds with `🦌 Donate here: https://easydonate.app/deerngo0` within 2 seconds.
- **AC-010b:** Given streamer.bot is offline, When a viewer types the command, Then no response is sent because no bot is available to process it.
- **AC-010c:** Given 5 viewers type the command simultaneously, When streamer.bot processes them, Then each receives a response.
- **AC-010d:** Given a message begins with `:deer: donate` and contains extra text, When streamer.bot evaluates it, Then the donation link response is sent.

**Story Points:** 2
**Priority:** 🔴 Must Have
**Epic:** E-02
**Sprint:** Sprint 1
**Status:** Draft
**Objective:** OBJ-02

---

#### US-011: Point Command

**As a** registered viewer
**I want** to type `:deer: point` and receive my point status
**So that** I can track my contribution without exposing more information than my visibility setting allows.

**Acceptance Criteria:**
- **AC-011a:** Given an active public member has points greater than 0, When they type `:deer: point`, Then the bot shows the exact point total.
- **AC-011b:** Given an active public member has 0 points, When they type `:deer: point`, Then the bot shows 0 points and a donation prompt.
- **AC-011c:** Given an active private member has 563 points, When they type `:deer: point`, Then the bot shows a 100-point band: `between 500–600`, not the exact total.
- **AC-011d:** Given an active private member has 0 points, When they type `:deer: point`, Then the bot shows `🦌 Your points are between 0–100.`
- **AC-011e:** Given the chat author has no active member, When they type `:deer: point`, Then the bot says: `🦌 You are not registered yet. Use :deer: register first.`
- **AC-011f:** Given the backend is unavailable, When a registered viewer queries points, Then the bot sends a temporary-unavailable message without exposing internal details.

**Story Points:** 3
**Priority:** 🔴 Must Have
**Epic:** E-02
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

#### US-012: Streamer.bot Member and Command Actions

**As a** streamer (Deer_NGO)
**I want** reproducible streamer.bot actions for registration, visibility, donation, and points
**So that** viewers can use the VRM program during live streams.

**Acceptance Criteria:**
- **AC-012a:** Given a viewer types `:deer: register`, Then streamer.bot sends the actual YouTube user ID and current handle to the backend and ignores a handle typed in the message.
- **AC-012b:** Given a registered viewer types `:deer: public`, Then streamer.bot calls the visibility API and confirms public visibility.
- **AC-012c:** Given a registered viewer types `:deer: private`, Then streamer.bot calls the visibility API and confirms private visibility.
- **AC-012d:** Given a viewer types `:deer: donate`, Then the Donate action posts the EasyDonate link.
- **AC-012e:** Given a viewer types `:deer: point`, Then the Point action calls the backend and posts the exact or banded result according to visibility.
- **AC-012f:** Given the backend returns an error, Then streamer.bot sends a friendly fallback message instead of crashing.
- **AC-012g:** Given any action executes, Then the execution is logged with trigger, timestamp, and result without secrets.

**Story Points:** 3
**Priority:** 🔴 Must Have
**Epic:** E-02
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-01, OBJ-02, OBJ-03

---

### Epic E-03: Points Engine

#### US-020: EasyDonate Donation Ingestion

**As a** system
**I want** to receive new EasyDonate donations through a webhook and reconcile them through the API
**So that** donation events are available for future member-point matching.

**Acceptance Criteria:**
- **AC-020a:** Given EasyDonate sends the documented webhook payload containing `referenceNo`, `donatorName`, `amount`, `channelName`, `donateMessage`, and `time`, When the webhook is valid, Then the donation is stored with `reference_no`, raw donor name, THB amount, donation time, and `source=webhook`.
- **AC-020b:** Given a donation with the same `referenceNo` is received again, When idempotency is checked, Then no duplicate donation or points are created.
- **AC-020c:** Given the fallback API is called, When the backend authenticates with the EasyDonate personal API key and donation-read scope, Then new donations are imported with `source=api_poll`.
- **AC-020d:** Given EasyDonate returns `429`, When the fallback sync detects it, Then the process backs off and respects the provider rate-limit guidance.
- **AC-020e:** Given EasyDonate returns no new donations, When the sync completes, Then no records are created and no error is raised.
- **AC-020f:** Given EasyDonate is unavailable, When webhook or polling fails, Then the error is logged safely and the next reconciliation cycle retries.
- **AC-020g:** Given a webhook request uses an invalid path token or fails payload validation, When the backend receives it, Then it rejects the request without storing a donation.

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-03
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

#### US-021: Exact Member Donation Matching

**As a** system
**I want** to match EasyDonate donor names to active member handles using normalized exact matching
**So that** points are awarded only when the community's naming rule is satisfied.

**Acceptance Criteria:**
- **AC-021a:** Given donor name `deer123` and an active member handle `@Deer123`, When matching runs, Then the donation matches that active member.
- **AC-021b:** Given harmless formatting differences such as whitespace, capitalization, or a leading `@`, When normalization runs, Then the values compare equal.
- **AC-021c:** Given an inactive member has the matching handle, When matching runs, Then the donation is not awarded to that inactive member.
- **AC-021d:** Given `donation_time` is before `member.registered_at`, When matching runs, Then the donation is marked not eligible and awards no points.
- **AC-021e:** Given no active member has the normalized donor name, When matching runs, Then the donation remains uncredited and is retained privately for reconciliation.
- **AC-021f:** Given donor names contain real names or messages, When the system stores them, Then they are private operational data and never appear on the public scoreboard.
- **AC-021g:** Given an eligible exact match exists, When points are applied, Then `amount_thb` is added to that member's `total_points` at 1 THB = 1 point exactly once.

**Story Points:** 5
**Priority:** 🔴 Must Have
**Epic:** E-03
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

#### US-022: Member Points Query API

**As a** system
**I want** to calculate and return a member's point status
**So that** streamer.bot can respond safely in public chat.

**Acceptance Criteria:**
- **AC-022a:** Given an active member has eligible matched donations totaling 500 THB, When queried, Then the member's exact total is 500 points for public visibility.
- **AC-022b:** Given an active member has no eligible donations, When queried, Then the member has 0 points.
- **AC-022c:** Given an active member is queried by the actual chat identity, When `GET /api/v1/members/{youtube_user_id}/points` is called, Then `200 OK` returns the member's visibility-aware point response.
- **AC-022d:** Given the chat identity has no active member, When queried, Then the API returns a member-not-registered result that streamer.bot maps to the registration message.
- **AC-022e:** Given an active private member has 563 points, When queried, Then the response exposes only `lower=500` and `upper=600`, not the exact total.
- **AC-022f:** Given an active member changes visibility, When the visibility-aware query runs, Then the response follows the latest `public_visibility` value without changing points.

**Story Points:** 3
**Priority:** 🔴 Must Have
**Epic:** E-03
**Sprint:** Sprint 2
**Status:** Draft
**Objective:** OBJ-03

---

### Epic E-04: Web Scoreboard

#### US-030: Public Scoreboard Page

**As a** viewer
**I want** to see a public scoreboard showing active public contributors
**So that** I can see the community's contribution ranking.

**Acceptance Criteria:**
- **AC-030a:** Given active public members have points greater than 0, When the page loads, Then it displays normalized YouTube handle and point total ranked highest first.
- **AC-030b:** Given a member is private, When the page loads, Then that member is not displayed.
- **AC-030c:** Given a member is inactive, When the page loads, Then that member is not displayed.
- **AC-030d:** Given active public members have 0 points only, When the page loads, Then the empty state says `No contributors yet. Be the first!`.
- **AC-030e:** Given new donations are processed, When the page refreshes, Then data freshness is under 60 seconds.
- **AC-030f:** Given the backend is unavailable, When the page loads, Then it shows `Scoreboard temporarily unavailable`.

**Story Points:** 5
**Priority:** 🟡 Should Have
**Epic:** E-04
**Sprint:** Sprint 3
**Status:** Draft
**Objective:** OBJ-04

---

#### US-031: Public Scoreboard API

**As a** frontend developer
**I want** a REST API that returns only eligible public members
**So that** the frontend cannot accidentally expose private or inactive records.

**Acceptance Criteria:**
- **AC-031a:** Given active public members have points greater than 0, When `GET /api/v1/scoreboard` runs, Then it returns `200 OK` with `rank`, normalized `youtube_handle`, and `total_points`, sorted descending.
- **AC-031b:** Given no active public member has points greater than 0, When queried, Then it returns `200 OK` with an empty array.
- **AC-031c:** Given private, inactive, and 0-point members exist, When queried, Then none of them appear.
- **AC-031d:** Given `?page=1&limit=50`, When queried, Then the first 50 eligible members are returned.
- **AC-031e:** Given `?page=2&limit=50`, When queried, Then eligible members 51–100 are returned.
- **AC-031f:** Given the response is serialized, Then it contains no YouTube display name, raw donor name, donation message, or private member data.

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
| E-01 Member Registration | 2 | 8 | Sprint 1 |
| E-02 Bot Commands | 3 | 8 | Sprint 1-2 |
| E-03 Points Engine | 3 | 13 | Sprint 2 |
| E-04 Web Scoreboard | 2 | 8 | Sprint 3 |
| **Total** | **10** | **37** | |

## 6. Story Map

| | Sprint 1 | Sprint 2 | Sprint 3 |
|--|---------|---------|---------|
| **E-01 Member Registration** | US-001, US-002 | | |
| **E-02 Bot Commands** | US-010 | US-011, US-012 | |
| **E-03 Points Engine** | | US-020, US-021, US-022 | |
| **E-04 Scoreboard** | | | US-030, US-031 |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[011_business_objective]] | Stories trace to business objectives |
| [[013_acceptance_criteria]] | Detailed ACs in Given/When/Then format |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Change decision that superseded subscriber polling |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29148
> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Change:** Explicit member registration replaces incomplete YouTube subscriber capture for Phase 1.
---
