---
document_type: Test Cases
version: "0.1"
status: Draft
author: "QA Engineer"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [test-cases, test-scenarios, swebok, iso-29119, vrm, deerngo-bot]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Test Cases

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

Detailed test cases — preconditions, steps, expected results, and traceability for each acceptance criteria. All 54 ACs mapped to 56 test cases (AC-031d and AC-031e split into separate pagination tests).

## 2. Test Case Index

| Module | Total | Automated | Manual | Status |
|--------|:-----:|:---------:|:------:|--------|
| E-01 Subscriber Capture | 18 | 14 | 4 | ⬜ Not Run |
| E-02 Bot Commands | 12 | 4 | 8 | ⬜ Not Run |
| E-03 Points Engine | 16 | 14 | 2 | ⬜ Not Run |
| E-04 Scoreboard | 10 | 6 | 4 | ⬜ Not Run |
| Cross-Cutting (Rate Limiting + HMAC) | 3 | 2 | 1 | ⬜ Not Run |
| **Total** | **59** | **40** | **19** | |

> Manual tests cover streamer.bot integration (black-box, Windows-only) and UI responsive design.

## 3. Test Case Template

| Field | Value |
|-------|-------|
| **Test Case ID** | TC-XXX |
| **Title** | Descriptive title |
| **Module** | E-0X: Module Name |
| **Priority** | 🔴 Critical / 🟡 High / 🟢 Medium |
| **Type** | Unit / Integration / System / Manual |
| **Automated** | Yes / No |
| **Requirement** | AC-XXX → US-XXX |

### Preconditions

| # | Condition |
|---|----------|
| 1 | Precondition description |

### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|

### Post-conditions

| # | Condition |
|---|----------|
| 1 | Post-condition description |

---

## 4. Test Cases — E-01: Subscriber Capture

### 4.1 US-001: Capture New Subscriber (Hybrid)

---

#### TC-001: Real-time Subscriber Capture via streamer.bot

| Field | Value |
|-------|-------|
| **ID** | TC-001 |
| **Title** | Capture new subscriber in real-time during live stream |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-001a → US-001 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend is running on port 8008 |
| 2 | PostgreSQL `deerngo_test` database is empty (no subscribers) |
| 3 | streamer.bot is running and connected to @Deer_NGO's YouTube chat |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `POST /api/v1/subscribers` with body: `{"youtube_handle": "@newviewer", "display_name": "New Viewer", "subscribed_at": "2026-07-30T10:00:00Z", "source": "streamer_bot"}` | Response status: `201 Created` |
| 2 | Verify response body | Contains `data.id` (UUID), `data.youtube_handle`: `"@newviewer"`, `data.source`: `"streamer_bot"` |
| 3 | Query PostgreSQL: `SELECT * FROM subscribers WHERE youtube_handle = 'newviewer'` | Record exists with `display_name = 'New Viewer'`, `source = 'streamer_bot'` |
| 4 | Measure response time | Response time < 5 seconds |

**Post-conditions:**

| # | Condition |
|---|----------|
| 1 | 1 subscriber record in database |

---

#### TC-002: Offline Subscriber Capture via YouTube API Polling

| Field | Value |
|-------|-------|
| **ID** | TC-002 |
| **Title** | Capture subscriber via YouTube API polling when streamer.bot is offline |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-001b → US-001 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend is running |
| 2 | YouTube API mock returns 1 new subscriber: `{"handle": "@offlineviewer", "displayName": "Offline Viewer"}` |
| 3 | Database is empty |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger YouTube API polling scheduler | Scheduler calls mock YouTube API |
| 2 | Verify response from mock | API called with correct OAuth token and channel ID |
| 3 | Query PostgreSQL: `SELECT * FROM subscribers WHERE youtube_handle = 'offlineviewer'` | Record exists with `source = 'youtube_api'` |
| 4 | Verify `subscribed_at` timestamp | Matches the timestamp from YouTube API response |

---

#### TC-003: Re-subscriber Upsert (No Duplicate)

| Field | Value |
|-------|-------|
| **ID** | TC-003 |
| **Title** | Re-subscriber does not create duplicate — existing record updated |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-001c → US-001 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Database contains subscriber: `youtube_handle = 'viewer1'`, `subscribed_at = '2026-07-28T10:00:00Z'` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `POST /api/v1/subscribers` with `youtube_handle: "@viewer1"`, `subscribed_at: "2026-07-30T10:00:00Z"` | Response status: `200 OK` (upsert) |
| 2 | Verify response body | `meta.upserted = true` |
| 3 | Query PostgreSQL: `SELECT COUNT(*) FROM subscribers WHERE youtube_handle = 'viewer1'` | Count = 1 (no duplicate) |
| 4 | Verify `subscribed_at` | Still `2026-07-28T10:00:00Z` (earliest preserved) |

---

#### TC-004: Dual-Source Timestamp Preservation

| Field | Value |
|-------|-------|
| **ID** | TC-004 |
| **Title** | Both sources capture same subscriber — earliest timestamp preserved |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-001d → US-001 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | streamer.bot captures `@viewer1` at `2026-07-30T10:00:00Z` (earlier) |
| 2 | Database has record with `subscribed_at = '2026-07-30T10:00:00Z'` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger YouTube API polling that returns `@viewer1` with `subscribed_at: "2026-07-30T10:15:00Z"` | Upsert runs |
| 2 | Query PostgreSQL: `SELECT subscribed_at FROM subscribers WHERE youtube_handle = 'viewer1'` | `subscribed_at = '2026-07-30T10:00:00Z'` (earliest preserved) |
| 3 | Verify `updated_at` | Updated to current time (record touched, but timestamp not overwritten) |

---

#### TC-005: Backend Down During Live — Graceful Failure

| Field | Value |
|-------|-------|
| **ID** | TC-005 |
| **Title** | Backend down — streamer.bot HTTP request fails, no crash |
| **Priority** | 🟡 High |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-001e → US-001 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend is NOT running |
| 2 | streamer.bot is running with HTTP Request action configured |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger a subscription event in streamer.bot | streamer.bot fires HTTP Request to `localhost:8008` |
| 2 | Observe streamer.bot logs | Error logged (connection refused / timeout) |
| 3 | Verify streamer.bot | Does not crash, continues operating |

---

#### TC-006: Missing Display Name — 400 Validation Error

| Field | Value |
|-------|-------|
| **ID** | TC-006 |
| **Title** | Empty display_name returns 400 — field is required |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-001f → US-001 (revised per PO decision DEF-S001) |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend is running |
| 2 | Database is empty |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `POST /api/v1/subscribers` with `youtube_handle: "@novalue"`, `display_name: ""` | `400 Bad Request` |
| 2 | Verify response body | `error.code = "VALIDATION_ERROR"`, `error.details` contains `field: "display_name"` |

> **PO Decision (DEF-S001):** `display_name` is required. AC-001f to be removed/rewritten. API Spec stays strict.

---

### 4.2 US-002: Subscriber Registration API

---

#### TC-007: Valid Subscriber Payload — 201 Created

| Field | Value |
|-------|-------|
| **ID** | TC-007 |
| **Title** | Valid POST creates subscriber — 201 response with full record |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-002a → US-002 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend running, database empty |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /api/v1/subscribers` with `{"youtube_handle": "@viewer1", "display_name": "Viewer One", "subscribed_at": "2026-07-29T10:00:00Z", "source": "streamer_bot"}` | `201 Created` |
| 2 | Verify response body | `data.id` is valid UUID, all fields match input |
| 3 | Verify response headers | `Content-Type: application/json` |

---

#### TC-008: Duplicate Handle — 200 OK Upsert

| Field | Value |
|-------|-------|
| **ID** | TC-008 |
| **Title** | Duplicate youtube_handle returns 200, updates existing record |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-002b → US-002 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Database has subscriber with `youtube_handle = 'viewer1'` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /api/v1/subscribers` with same `youtube_handle: "@viewer1"`, different `display_name: "Updated Name"` | `200 OK` |
| 2 | Verify response | `meta.upserted = true` |
| 3 | Query DB | `display_name` updated to `"Updated Name"`, only 1 record exists |

---

#### TC-009: Missing Required Field — 400 Validation Error

| Field | Value |
|-------|-------|
| **ID** | TC-009 |
| **Title** | Missing youtube_handle returns 400 with field-level error |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-002c → US-002 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /api/v1/subscribers` with `{"display_name": "Viewer One"}` (no youtube_handle) | `400 Bad Request` |
| 2 | Verify response body | `error.code = "VALIDATION_ERROR"`, `error.details` contains `field: "youtube_handle"` |

---

#### TC-010: Empty Payload — 400 Validation Error

| Field | Value |
|-------|-------|
| **ID** | TC-010 |
| **Title** | Empty payload returns 400 with errors for all required fields |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-002d → US-002 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /api/v1/subscribers` with `{}` | `400 Bad Request` |
| 2 | Verify response body | `error.details` lists all 4 required fields |

---

#### TC-011: Invalid Timestamp — 400 Validation Error

| Field | Value |
|-------|-------|
| **ID** | TC-011 |
| **Title** | Non-ISO-8601 subscribed_at returns 400 |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-002e → US-002 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /api/v1/subscribers` with `subscribed_at: "not-a-date"` | `400 Bad Request` |
| 2 | Verify error message | Contains "subscribed_at must be ISO 8601" or equivalent |

---

### 4.3 US-003: YouTube API Polling Scheduler

---

#### TC-012: Polling — New Subscribers Found

| Field | Value |
|-------|-------|
| **ID** | TC-012 |
| **Title** | Polling scheduler fetches and creates new subscriber records |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003a → US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | YouTube API mock returns 5 new subscribers |
| 2 | Database is empty |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger polling scheduler | Scheduler calls `GET /youtube/v3/subscriptions` |
| 2 | Query DB: `SELECT COUNT(*) FROM subscribers` | Count = 5 |
| 3 | Verify all records have `source = 'youtube_api'` | True |

---

#### TC-013: Polling — Upsert Existing Subscribers

| Field | Value |
|-------|-------|
| **ID** | TC-013 |
| **Title** | Polling updates existing records, creates new ones — no duplicates |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003b → US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Database has 2 subscribers: `@viewer1`, `@viewer2` |
| 2 | YouTube API mock returns 3 subscribers: `@viewer1`, `@viewer2`, `@viewer3` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger polling scheduler | Processes 3 subscribers |
| 2 | Query DB: `SELECT COUNT(*) FROM subscribers` | Count = 3 (2 updated, 1 new) |
| 3 | Verify `@viewer3` exists with `source = 'youtube_api'` | True |

---

#### TC-014: Polling — Preserve Earliest Timestamp

| Field | Value |
|-------|-------|
| **ID** | TC-014 |
| **Title** | Polling does not overwrite earlier streamer.bot timestamp |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003c → US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | `@viewer1` captured via streamer.bot at `10:00:00Z` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | YouTube API mock returns `@viewer1` with `subscribed_at: "10:15:00Z"` | Polling processes |
| 2 | Query DB: `SELECT subscribed_at FROM subscribers WHERE youtube_handle = 'viewer1'` | `10:00:00Z` preserved |

---

#### TC-015: Polling — No New Subscribers

| Field | Value |
|-------|-------|
| **ID** | TC-015 |
| **Title** | Polling with 0 new subscribers completes without error |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003d → US-003 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | YouTube API mock returns empty list | Polling completes |
| 2 | Verify no error logs | Clean completion |

---

#### TC-016: Polling — YouTube API 403 Quota Exceeded

| Field | Value |
|-------|-------|
| **ID** | TC-016 |
| **Title** | 403 quota exceeded — scheduler logs error, skips next cycle |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003e → US-003 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | YouTube API mock returns 403 `quotaExceeded` | Scheduler detects error |
| 2 | Verify error logged | Log contains quota exceeded message |
| 3 | Verify next poll cycle skipped | No API call on next scheduled trigger |

---

#### TC-017: Polling — YouTube API 401 Token Expired

| Field | Value |
|-------|-------|
| **ID** | TC-017 |
| **Title** | 401 unauthorized — logs error, alerts operator |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003f → US-003 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | YouTube API mock returns 401 | Scheduler detects error |
| 2 | Verify error logged | Log contains token expired / unauthorized message |
| 3 | Verify operator alert mechanism triggered | Alert sent (log entry or notification) |

---

#### TC-018: Polling — 15-Minute Interval

| Field | Value |
|-------|-------|
| **ID** | TC-018 |
| **Title** | Polling triggers automatically every 15 minutes |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-003g → US-003 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Start backend, note time T0 | Scheduler starts |
| 2 | Wait 15 minutes (or mock time) | Scheduler triggers at T0+15m |
| 3 | Verify API call made at T0+15m | YouTube API called |
| 4 | Verify next trigger at T0+30m | YouTube API called again |

---

## 5. Test Cases — E-02: Bot Commands

### 5.1 US-010: Donate Command

---

#### TC-019: Donate Command — Happy Path

| Field | Value |
|-------|-------|
| **ID** | TC-019 |
| **Title** | `:deer: donate` in chat → bot responds with donation link |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-010a → US-010 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | streamer.bot is running with Donate action configured |
| 2 | Connected to @Deer_NGO's YouTube live chat |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Type `:deer: donate` in YouTube live chat | Bot responds within 2 seconds |
| 2 | Verify response text | `"🦌 Donate here: https://easydonate.app/deerngo0"` |

---

#### TC-020: Donate Command — Bot Offline

| Field | Value |
|-------|-------|
| **ID** | TC-020 |
| **Title** | Bot offline — no response, no error spam |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-010b → US-010 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop streamer.bot | Bot disconnected |
| 2 | Type `:deer: donate` in chat | No bot response (graceful degradation) |
| 3 | Verify no error messages in chat | Clean — no error spam |

---

#### TC-021: Donate Command — Multiple Simultaneous

| Field | Value |
|-------|-------|
| **ID** | TC-021 |
| **Title** | 5 simultaneous donate commands → 5 responses |
| **Priority** | 🟡 High |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-010c → US-010 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | 5 viewers type `:deer: donate` at the same time | Bot sends 5 responses (one per viewer) |

---

#### TC-022: Donate Command — Extra Text After Command

| Field | Value |
|-------|-------|
| **ID** | TC-022 |
| **Title** | `:deer: donate please help` still triggers response |
| **Priority** | 🟡 High |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-010d → US-010 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Type `:deer: donate please help` in chat | Bot detects `:deer: donate` prefix, responds with donation link |

---

### 5.2 US-011: Point Command

---

#### TC-023: Point Command — Viewer Has Points

| Field | Value |
|-------|-------|
| **ID** | TC-023 |
| **Title** | `:deer: point` for viewer with 500 points → displays balance |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-011a → US-011 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | `@viewer1` has 500 points in `viewer_points` table |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Viewer `@viewer1` types `:deer: point` in chat | Bot responds within 2 seconds |
| 2 | Verify response | `"🦌 @viewer1 has 500 points!"` |

---

#### TC-024: Point Command — Viewer Has 0 Points

| Field | Value |
|-------|-------|
| **ID** | TC-024 |
| **Title** | `:deer: point` for new viewer → shows 0 points with donate prompt |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-011b → US-011 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Viewer `@newviewer` types `:deer: point` | Bot responds |
| 2 | Verify response | `"🦌 @newviewer has 0 points. Donate to earn points!"` |

---

#### TC-025: Point Command — Backend API Down

| Field | Value |
|-------|-------|
| **ID** | TC-025 |
| **Title** | Backend down → bot shows friendly error |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-011c → US-011 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop Go backend | API unavailable |
| 2 | Viewer types `:deer: point` | Bot responds |
| 3 | Verify response | `"🦌 Points system is temporarily unavailable"` |

---

#### TC-026: Point Command — API Timeout

| Field | Value |
|-------|-------|
| **ID** | TC-026 |
| **Title** | API takes >5s → bot shows timeout error |
| **Priority** | 🟡 High |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-011d → US-011 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Configure API mock to delay 6 seconds | Slow response |
| 2 | Viewer types `:deer: point` | Bot waits, then shows timeout error after 5 seconds |

---

### 5.3 US-012: Streamer.bot Action Configuration

---

#### TC-027: Donate Action — Triggers on Command

| Field | Value |
|-------|-------|
| **ID** | TC-027 |
| **Title** | Donate action fires when chat contains `:deer: donate` |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-012a → US-012 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Verify streamer.bot action "Donate" is configured | Action exists with chat trigger `:deer: donate` |
| 2 | Send `:deer: donate` in chat | Action fires, donation link posted |

---

#### TC-028: Point Action — Triggers on Command

| Field | Value |
|-------|-------|
| **ID** | TC-028 |
| **Title** | Point action fires, calls Go API, posts response |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-012b → US-012 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Verify streamer.bot action "Point" configured | Action exists with HTTP Request sub-action to `GET /api/v1/points/{handle}` |
| 2 | Send `:deer: point` in chat | Action fires, API called, point response posted |

---

#### TC-029: Point Action — API Error Handling

| Field | Value |
|-------|-------|
| **ID** | TC-029 |
| **Title** | Go backend returns 500 → streamer.bot sends fallback message |
| **Priority** | 🔴 Critical |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-012c → US-012 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Mock Go backend to return 500 | API error |
| 2 | Send `:deer: point` in chat | Action fires, receives error |
| 3 | Verify chat response | Fallback error message (not a crash) |

---

#### TC-030: Action Logs — Execution Logged

| Field | Value |
|-------|-------|
| **ID** | TC-030 |
| **Title** | Action execution logged in streamer.bot |
| **Priority** | 🟡 High |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-012d → US-012 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger any action (`:deer: donate` or `:deer: point`) | Action fires |
| 2 | Check streamer.bot logs | Log entry with timestamp, trigger, result |

---

## 6. Test Cases — E-03: Points Engine

### 6.1 US-020: EasyDonate Donation Sync

---

#### TC-031: Donation Sync — New Donations Found

| Field | Value |
|-------|-------|
| **ID** | TC-031 |
| **Title** | Sync fetches 3 new donations from EasyDonate API |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-020a → US-020 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | EasyDonate API mock returns 3 donations |
| 2 | Database donations table is empty |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Trigger donation sync | Sync calls EasyDonate API |
| 2 | Query DB: `SELECT COUNT(*) FROM donations` | Count = 3 |
| 3 | Verify fields | Each record has `donor_name`, `amount_thb`, `donation_time`, `easydonate_id` |

---

#### TC-032: Donation Sync — Idempotent (No Duplicate)

| Field | Value |
|-------|-------|
| **ID** | TC-032 |
| **Title** | Duplicate easydonate_id skipped — no new record created |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-020b → US-020 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Database has donation with `easydonate_id = 'ed-123'` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | EasyDonate API mock returns donation with `easydonate_id: "ed-123"` | Sync processes |
| 2 | Query DB: `SELECT COUNT(*) FROM donations WHERE easydonate_id = 'ed-123'` | Count = 1 (no duplicate) |

---

#### TC-033: Donation Sync — Rate Limit (429) Backoff

| Field | Value |
|-------|-------|
| **ID** | TC-033 |
| **Title** | 429 response — sync backs off and retries after Retry-After |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-020c → US-020 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | EasyDonate API mock returns 429 + `Retry-After: 60` | Sync detects rate limit |
| 2 | Verify sync pauses | Waits 60 seconds before retry |
| 3 | After retry (mock returns 200) | Donations processed |

---

#### TC-034: Donation Sync — Empty Response

| Field | Value |
|-------|-------|
| **ID** | TC-034 |
| **Title** | 0 donations returned — completes without error |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-020d → US-020 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | EasyDonate API mock returns empty array | Sync completes |
| 2 | Verify no error logs | Clean completion |

---

#### TC-035: Donation Sync — API Unavailable

| Field | Value |
|-------|-------|
| **ID** | TC-035 |
| **Title** | EasyDonate API down — logs error, retries next cycle |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-020e → US-020 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | EasyDonate API mock returns 500 | Sync detects error |
| 2 | Verify error logged | Log contains API failure message |
| 3 | Verify next sync cycle runs | Retries on next scheduled trigger |

---

### 6.2 US-021: Name Matching Engine

---

#### TC-036: Matching — Exact Match (Case-Insensitive)

| Field | Value |
|-------|-------|
| **ID** | TC-036 |
| **Title** | Donation from "deer123" matches subscriber "@deer123" |
| **Priority** | 🔴 Critical |
| **Type** | Unit |
| **Automated** | Yes |
| **Requirement** | AC-021a → US-021 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Subscriber `@deer123` exists |
| 2 | Donation from `donor_name = "deer123"`, `match_status = 'pending'` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Run matching engine | Processes pending donation |
| 2 | Query donation record | `match_status = 'matched'`, `matched_handle = '@deer123'` |
| 3 | Verify match_score | ≥ 0.7 |

---

#### TC-037: Matching — Fuzzy Match (Strip @)

| Field | Value |
|-------|-------|
| **ID** | TC-037 |
| **Title** | Donation from "@viewer1" matches subscriber "viewer1" |
| **Priority** | 🔴 Critical |
| **Type** | Unit |
| **Automated** | Yes |
| **Requirement** | AC-021b → US-021 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Subscriber `viewer1` exists, donation `@viewer1` | Matching runs |
| 2 | After normalization (strip @, lowercase) | Both become `viewer1` |
| 3 | Verify match | `match_status = 'matched'`, `matched_handle = '@viewer1'` |

---

#### TC-038: Matching — Anonymous Donation

| Field | Value |
|-------|-------|
| **ID** | TC-038 |
| **Title** | "anonymous" donor → unmatched, no points awarded |
| **Priority** | 🔴 Critical |
| **Type** | Unit |
| **Automated** | Yes |
| **Requirement** | AC-021c → US-021 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Donation with `donor_name = "anonymous"` | Matching runs |
| 2 | Verify donation record | `match_status = 'unmatched'`, `matched_handle = NULL` |
| 3 | Verify no points awarded | `viewer_points` unchanged |

---

#### TC-039: Matching — Empty Donor Name

| Field | Value |
|-------|-------|
| **ID** | TC-039 |
| **Title** | Empty donor_name → unmatched |
| **Priority** | 🔴 Critical |
| **Type** | Unit |
| **Automated** | Yes |
| **Requirement** | AC-021d → US-021 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Donation with `donor_name = ""` | Matching runs |
| 2 | Verify | `match_status = 'unmatched'` |

---

#### TC-040: Matching — Multiple Possible Matches

| Field | Value |
|-------|-------|
| **ID** | TC-040 |
| **Title** | Multiple similar subscribers → best match selected, flagged for review |
| **Priority** | 🟡 High |
| **Type** | Unit |
| **Automated** | Yes |
| **Requirement** | AC-021e → US-021 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Subscribers: `deer123`, `deer456` |
| 2 | Donation from `donor_name = "deer"` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Run matching engine | Finds 2 matches above 0.7 threshold |
| 2 | Verify donation record | `match_status = 'manual_review'` |
| 3 | Verify best match stored | Highest similarity score selected |

---

#### TC-041: Matching — No Match Found

| Field | Value |
|-------|-------|
| **ID** | TC-041 |
| **Title** | No close match → unmatched with donor_name preserved |
| **Priority** | 🟡 High |
| **Type** | Unit |
| **Automated** | Yes |
| **Requirement** | AC-021f → US-021 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Donation from `donor_name = "unknown_user"`, no similar subscriber | Matching runs |
| 2 | Verify | `match_status = 'unmatched'`, `donor_name` preserved for manual review |

---

### 6.3 US-022: Point Calculation & Query API

---

#### TC-042: Points API — Happy Path (Sum of Donations)

| Field | Value |
|-------|-------|
| **ID** | TC-042 |
| **Title** | Viewer with 3 matched donations (100+200+200 THB) → total_points: 500 |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-022a → US-022 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | `@viewer1` in `viewer_points` with `total_points = 500` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/points/viewer1` | `200 OK` |
| 2 | Verify response | `data.total_points = 500.00`, `data.youtube_handle = "@viewer1"` |

---

#### TC-043: Points API — No Donations (0 Points)

| Field | Value |
|-------|-------|
| **ID** | TC-043 |
| **Title** | Viewer with no donations → total_points: 0 |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-022b → US-022 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/points/newviewer` | `200 OK` |
| 2 | Verify response | `data.total_points = 0`, `data.donation_count = 0`, `data.last_donation = null` |

---

#### TC-044: Points API — Non-Existent Handle

| Field | Value |
|-------|-------|
| **ID** | TC-044 |
| **Title** | Handle not in DB → 200 with 0 points (not 404) |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-022c → US-022 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/points/doesnotexist` | `200 OK` (NOT 404) |
| 2 | Verify response | `data.total_points = 0` |

---

#### TC-045: Points API — Only Matched Donations Count

| Field | Value |
|-------|-------|
| **ID** | TC-045 |
| **Title** | Unmatched donations excluded from point total |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-022d → US-022 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | `@viewer1` has 2 matched donations (300 THB) + 1 unmatched (100 THB) |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/points/viewer1` | `200 OK` |
| 2 | Verify `total_points` | 300 (unmatched 100 THB excluded) |

---

#### TC-046: Points API — Case-Insensitive Lookup

| Field | Value |
|-------|-------|
| **ID** | TC-046 |
| **Title** | Stored as "@Viewer1", queried as "viewer1" → correct match |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-022e → US-022 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/points/Viewer1` (uppercase) | `200 OK` |
| 2 | `GET /api/v1/points/viewer1` (lowercase) | `200 OK` with same points |
| 3 | Both return same `total_points` | Case-insensitive match works |

---

## 7. Test Cases — E-04: Web Scoreboard

### 7.1 US-030: Public Scoreboard Page

---

#### TC-047: Scoreboard Page — Happy Path

| Field | Value |
|-------|-------|
| **ID** | TC-047 |
| **Title** | Scoreboard displays ranked viewers by points |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | AC-030a → US-030 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | 10 viewers with points in database |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Visit scoreboard URL | Page loads |
| 2 | Verify list | Ranked list (highest points first), showing rank, display name, points |

---

#### TC-048: Scoreboard Page — Data Freshness

| Field | Value |
|-------|-------|
| **ID** | TC-048 |
| **Title** | New donation processed → scoreboard reflects within 60s |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | AC-030b → US-030 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Process a new donation (matched) | Points updated in DB |
| 2 | Refresh scoreboard within 60s | Updated points visible |

---

#### TC-049: Scoreboard Page — Responsive Design

| Field | Value |
|-------|-------|
| **ID** | TC-049 |
| **Title** | Scoreboard renders correctly on mobile |
| **Priority** | 🟡 High |
| **Type** | Manual |
| **Automated** | No |
| **Requirement** | AC-030c → US-030 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Open scoreboard on mobile device (or browser DevTools mobile view) | Page loads |
| 2 | Verify layout adapts | No horizontal scroll, text readable, list usable |

---

#### TC-050: Scoreboard Page — Backend Down

| Field | Value |
|-------|-------|
| **ID** | TC-050 |
| **Title** | Backend down → friendly error message displayed |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | AC-030d → US-030 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop Go backend | API unavailable |
| 2 | Visit scoreboard | Error message: "Scoreboard temporarily unavailable" |

---

#### TC-051: Scoreboard Page — Empty Scoreboard

| Field | Value |
|-------|-------|
| **ID** | TC-051 |
| **Title** | No contributors yet → motivational message |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | AC-030e → US-030 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Database has 0 viewers with points | Empty state |
| 2 | Visit scoreboard | Message: "No contributors yet. Be the first!" |

---

### 7.2 US-031: Scoreboard API Endpoint

---

#### TC-052: Scoreboard API — Happy Path

| Field | Value |
|-------|-------|
| **ID** | TC-052 |
| **Title** | GET /api/v1/scoreboard returns ranked JSON array |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-031a → US-031 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | 50 viewers with points in database |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/scoreboard` | `200 OK` |
| 2 | Verify response | `data` is array sorted by `total_points` descending, each has `rank`, `display_name`, `youtube_handle`, `total_points` |
| 3 | Verify only viewers with `total_points > 0` | No 0-point viewers |

---

#### TC-053: Scoreboard API — Empty Scoreboard

| Field | Value |
|-------|-------|
| **ID** | TC-053 |
| **Title** | No viewers with points → empty array (not error) |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-031b → US-031 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/scoreboard` (empty DB) | `200 OK` |
| 2 | Verify response | `data = []`, `meta.total = 0` |

---

#### TC-054: Scoreboard API — 0-Point Viewers Excluded

| Field | Value |
|-------|-------|
| **ID** | TC-054 |
| **Title** | 5 subscribers but only 3 have points → only 3 in response |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-031c → US-031 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/scoreboard` | `200 OK` |
| 2 | Verify `data.length` | 3 (not 5) |
| 3 | Verify all entries have `total_points > 0` | True |

---

#### TC-055: Scoreboard API — Pagination Page 1

| Field | Value |
|-------|-------|
| **ID** | TC-055 |
| **Title** | ?page=1&limit=50 returns first 50 viewers |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-031d → US-031 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | 200 viewers with points |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/scoreboard?page=1&limit=50` | `200 OK` |
| 2 | Verify `data.length` | 50 |
| 3 | Verify `meta.page = 1`, `meta.hasNext = true` | Pagination metadata correct |

---

#### TC-056: Scoreboard API — Pagination Page 2

| Field | Value |
|-------|-------|
| **ID** | TC-056 |
| **Title** | ?page=2&limit=50 returns viewers 51-100 |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | AC-031e → US-031 |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/v1/scoreboard?page=2&limit=50` | `200 OK` |
| 2 | Verify `data[0].rank` | 51 (starts from 51st) |
| 3 | Verify `meta.page = 2`, `meta.hasPrev = true` | Pagination metadata correct |

---

### 8. Additional Test Cases — PO Decision Additions

> Test cases added based on PO decisions from MM04 (DEF-S004, DEF-S005).

---

#### TC-057: Rate Limiting — Default Tier (100 req/min)

| Field | Value |
|-------|-------|
| **ID** | TC-057 |
| **Title** | Exceeding 100 requests/min returns 429 |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | API Spec §5 (per PO decision DEF-S004) |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend running with Fiber rate limiter middleware |
| 2 | Client IP not in any special tier |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send 100 requests to `GET /api/v1/points/testviewer` within 1 minute | All 100 return `200 OK` |
| 2 | Send 101st request within the same minute | `429 Too Many Requests` |
| 3 | Verify response body | `error.code = "RATE_LIMITED"` |
| 4 | Wait for rate limit window to reset | Next request returns `200 OK` |

---

#### TC-058: Rate Limiting — Webhook Tier (200 req/min)

| Field | Value |
|-------|-------|
| **ID** | TC-058 |
| **Title** | Webhook endpoint allows 200 req/min before 429 |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | API Spec §5 (per PO decision DEF-S004) |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend running with webhook-specific rate limiter |
| 2 | Valid HMAC signature available for test |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send 200 requests to `POST /api/v1/webhooks/easydonate` within 1 minute (each with unique `easydonate_id`) | All 200 return `200 OK` |
| 2 | Send 201st request within the same minute | `429 Too Many Requests` |
| 3 | Verify response body | `error.code = "RATE_LIMITED"` |

---

#### TC-059: HMAC Key Rotation — Procedure Verification

| Field | Value |
|-------|-------|
| **ID** | TC-059 |
| **Title** | HMAC secret key rotation — webhook continues working after key change |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | No |
| **Requirement** | API Spec §4.4, ADR-012 (per PO decision DEF-S005) |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Go backend running with webhook endpoint |
| 2 | Current HMAC secret key is configured in env var |
| 3 | Key rotation procedure documented |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send webhook with valid signature (current key) | `200 OK` — donation processed |
| 2 | Generate new HMAC secret key | New key created |
| 3 | Update env var to new key, restart backend | Backend reloaded with new key |
| 4 | Send webhook with old key signature | `401 Unauthorized` — rejected |
| 5 | Send webhook with new key signature | `200 OK` — donation processed |
| 6 | Verify no donations lost during rotation | All test donations accounted for |

---

## 9. Test Execution Summary

| Sprint | Executed | Passed | Failed | Blocked | Pass Rate |
|--------|:-------:|:------:|:------:|:-------:|:---------:|
| Sprint 1 | — | — | — | — | — |
| Sprint 2 | — | — | — | — | — |
| Sprint 3 | — | — | — | — | — |
| **Total** | **—** | **—** | **—** | **—** | **—** |

> To be filled during test execution.

---

## 10. Requirements Traceability Matrix

| AC ID | User Story | Test Case | Test Status |
|-------|-----------|-----------|------------|
| AC-001a | US-001 | TC-001 | ⬜ Not Run |
| AC-001b | US-001 | TC-002 | ⬜ Not Run |
| AC-001c | US-001 | TC-003 | ⬜ Not Run |
| AC-001d | US-001 | TC-004 | ⬜ Not Run |
| AC-001e | US-001 | TC-005 | ⬜ Not Run |
| AC-001f | US-001 | TC-006 | ⬜ Not Run |
| AC-002a | US-002 | TC-007 | ⬜ Not Run |
| AC-002b | US-002 | TC-008 | ⬜ Not Run |
| AC-002c | US-002 | TC-009 | ⬜ Not Run |
| AC-002d | US-002 | TC-010 | ⬜ Not Run |
| AC-002e | US-002 | TC-011 | ⬜ Not Run |
| AC-003a | US-003 | TC-012 | ⬜ Not Run |
| AC-003b | US-003 | TC-013 | ⬜ Not Run |
| AC-003c | US-003 | TC-014 | ⬜ Not Run |
| AC-003d | US-003 | TC-015 | ⬜ Not Run |
| AC-003e | US-003 | TC-016 | ⬜ Not Run |
| AC-003f | US-003 | TC-017 | ⬜ Not Run |
| AC-003g | US-003 | TC-018 | ⬜ Not Run |
| AC-010a | US-010 | TC-019 | ⬜ Not Run |
| AC-010b | US-010 | TC-020 | ⬜ Not Run |
| AC-010c | US-010 | TC-021 | ⬜ Not Run |
| AC-010d | US-010 | TC-022 | ⬜ Not Run |
| AC-011a | US-011 | TC-023 | ⬜ Not Run |
| AC-011b | US-011 | TC-024 | ⬜ Not Run |
| AC-011c | US-011 | TC-025 | ⬜ Not Run |
| AC-011d | US-011 | TC-026 | ⬜ Not Run |
| AC-012a | US-012 | TC-027 | ⬜ Not Run |
| AC-012b | US-012 | TC-028 | ⬜ Not Run |
| AC-012c | US-012 | TC-029 | ⬜ Not Run |
| AC-012d | US-012 | TC-030 | ⬜ Not Run |
| AC-020a | US-020 | TC-031 | ⬜ Not Run |
| AC-020b | US-020 | TC-032 | ⬜ Not Run |
| AC-020c | US-020 | TC-033 | ⬜ Not Run |
| AC-020d | US-020 | TC-034 | ⬜ Not Run |
| AC-020e | US-020 | TC-035 | ⬜ Not Run |
| AC-021a | US-021 | TC-036 | ⬜ Not Run |
| AC-021b | US-021 | TC-037 | ⬜ Not Run |
| AC-021c | US-021 | TC-038 | ⬜ Not Run |
| AC-021d | US-021 | TC-039 | ⬜ Not Run |
| AC-021e | US-021 | TC-040 | ⬜ Not Run |
| AC-021f | US-021 | TC-041 | ⬜ Not Run |
| AC-022a | US-022 | TC-042 | ⬜ Not Run |
| AC-022b | US-022 | TC-043 | ⬜ Not Run |
| AC-022c | US-022 | TC-044 | ⬜ Not Run |
| AC-022d | US-022 | TC-045 | ⬜ Not Run |
| AC-022e | US-022 | TC-046 | ⬜ Not Run |
| AC-030a | US-030 | TC-047 | ⬜ Not Run |
| AC-030b | US-030 | TC-048 | ⬜ Not Run |
| AC-030c | US-030 | TC-049 | ⬜ Not Run |
| AC-030d | US-030 | TC-050 | ⬜ Not Run |
| AC-030e | US-030 | TC-051 | ⬜ Not Run |
| AC-031a | US-031 | TC-052 | ⬜ Not Run |
| AC-031b | US-031 | TC-053 | ⬜ Not Run |
| AC-031c | US-031 | TC-054 | ⬜ Not Run |
| AC-031d | US-031 | TC-055 | ⬜ Not Run |
| AC-031e | US-031 | TC-056 | ⬜ Not Run |
| Rate Limiting (default) | API Spec §5 | TC-057 | ⬜ Not Run |
| Rate Limiting (webhook) | API Spec §5 | TC-058 | ⬜ Not Run |
| HMAC Key Rotation | API Spec §4.4 | TC-059 | ⬜ Not Run |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Plan governing these cases |
| [[013_acceptance_criteria]] | ACs these cases verify |
| [[022_API_specification]] | API contracts for integration tests |
| [[023_database_schema_DDL]] | DB constraints for integrity tests |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119
> **Usage:** Every acceptance criterion has at least one test case. Every test case traces to an AC. Keep them in sync.
