---
document_type: Acceptance Criteria (ATDD/BDD)
version: "0.1"
status: Draft
author: "PO"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
ba_owner: "PO"
qa_lead: "TBD"
classification: "Internal"
tags: [acceptance-criteria, bdd, atdd, given-when-then, vrm, viewer-relationship-management]
standard_ref:
  - SWEBOK v4 — Requirements
  - ISO/IEC/IEEE 29119 — Software Testing
  - ISO/IEC/IEEE 29148 — Requirements Engineering
---

# Acceptance Criteria (ATDD/BDD)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | PO |
| Business Analyst | PO |
| QA Lead | TBD |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | PO | Initial draft from stakeholder grill session |

---

## 1. Purpose

This document defines acceptance criteria for Deerngo Bot Phase 1 using the **Given/When/Then** (GWT) format. Each criterion is testable and traceable to a user story.

## 2. Acceptance Criteria Standards

### 2.1 Format — Given/When/Then

```gherkin
Given [precondition / initial context]
When [action / trigger]
Then [expected outcome / result]
```

### 2.2 Quality Criteria

| Criterion | Description | Example |
|-----------|-------------|---------|
| **Testable** | Can be verified by test | ✅ "Response time <2s" vs ❌ "Fast response" |
| **Unambiguous** | One interpretation only | ✅ "Email sent within 5 minutes" vs ❌ "Email sent promptly" |
| **Complete** | Covers happy path + edge cases | Multiple scenarios per requirement |
| **Independent** | Each criterion testable standalone | No dependencies between criteria |
| **Negotiable** | Agreed by PO, Dev, QA | Reviewed in 3 amigos session |

---

## 3. Acceptance Criteria by Requirement

### 3.1 E-01: Register (Subscriber Capture)

#### US-001: Capture New Subscriber (Hybrid)

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-001a | Real-time during live | A viewer subscribes to @Deer_NGO during a live stream and streamer.bot fires the subscription event | The Go backend receives the POST request | A subscriber record is created with: youtube_handle, display_name, subscribed_at, source="streamer_bot". Response time <5s | 🔴 |
| AC-001b | Offline via YouTube API polling | A viewer subscribes at 3am (streamer.bot is offline) | The YouTube API polling scheduler runs (every 15 min) | The backend fetches new subscribers from YouTube Data API and creates/updates records with source="youtube_api" | 🔴 |
| AC-001c | Re-subscriber (upsert) | A subscriber with handle "@viewer1" already exists in the database | The same subscriber is captured again (from either source) | The existing record is updated (upsert by youtube_handle) — no duplicate created | 🔴 |
| AC-001d | Both sources capture same subscriber | streamer.bot captures "@viewer1" during live, then API polling also finds "@viewer1" | The upsert runs from the second source | The record is NOT overwritten — the earliest subscription timestamp is preserved | 🔴 |
| AC-001e | Backend down during live | The Go backend is not running | streamer.bot fires a subscription event | The HTTP request fails, streamer.bot logs the error. The next API polling cycle will catch the subscriber | 🟡 |
| AC-001f | Missing display_name | A subscriber event arrives with youtube_handle but no display_name | The backend processes the request | The record is created with display_name = youtube_handle (fallback) | 🟡 |

#### US-002: Subscriber Registration API

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-002a | Valid payload | The API receives POST /api/v1/subscribers with {youtube_handle: "@viewer1", display_name: "Viewer One", subscribed_at: "2026-07-29T10:00:00Z", source: "streamer_bot"} | The backend validates and processes | 201 Created returned with the full subscriber record including id | 🔴 |
| AC-002b | Duplicate handle (upsert) | A subscriber with handle "@viewer1" exists | POST /api/v1/subscribers with same youtube_handle | 200 OK returned, existing record updated | 🔴 |
| AC-002c | Missing required field | POST /api/v1/subscribers with {display_name: "Viewer One"} (no youtube_handle) | Validation runs | 400 Bad Request with error: "youtube_handle is required" | 🔴 |
| AC-002d | Empty payload | POST /api/v1/subscribers with {} | Validation runs | 400 Bad Request with errors for all required fields | 🟡 |
| AC-002e | Invalid timestamp | POST /api/v1/subscribers with {youtube_handle: "@viewer1", subscribed_at: "not-a-date"} | Validation runs | 400 Bad Request with error: "subscribed_at must be ISO 8601" | 🟡 |

#### US-003: YouTube API Polling Scheduler

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-003a | Happy path — new subscribers found | The Go backend is running, YouTube API returns 5 new subscribers | The polling scheduler triggers (every 15 min) | The backend calls `GET /youtube/v3/subscriptions` and creates 5 subscriber records with source="youtube_api" | 🔴 |
| AC-003b | Upsert existing subscribers | YouTube API returns 3 subscribers, 2 already in DB | The polling scheduler processes them | 2 existing records are updated (no duplicate), 1 new record is created | 🔴 |
| AC-003c | Preserve earliest timestamp | Subscriber "@viewer1" was captured via streamer.bot at 10:00. API polling finds same subscriber at 10:15 | The upsert runs | The existing record is NOT overwritten — the 10:00 timestamp is preserved | 🔴 |
| AC-003d | No new subscribers | YouTube API returns 0 new subscribers | The polling scheduler processes the response | No records created/updated, scheduler completes without error | 🟡 |
| AC-003e | YouTube API quota exceeded (403) | YouTube API returns 403 quotaExceeded error | The scheduler detects the error | The scheduler logs the error and skips the next poll cycle. Quota = 10,000 units/day, 96 calls/day is well within limit | 🔴 |
| AC-003f | OAuth token expired (401) | YouTube API returns 401 Unauthorized | The scheduler detects the error | The backend logs the error and alerts the operator (manual re-auth required) | 🔴 |
| AC-003g | Polling interval | The Go backend is running | 15 minutes have passed since last poll | A new poll cycle starts automatically | 🔴 |

---

### 3.2 E-02: Bot — Commands

#### US-010: Donate Command

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-010a | Happy path | streamer.bot is running and connected to @Deer_NGO's YouTube chat | A viewer types `:deer: donate` in chat | The bot responds with "🦌 Donate here: https://easydonate.app/deerngo0" within 2 seconds | 🔴 |
| AC-010b | Bot offline | streamer.bot is not running | A viewer types `:deer: donate` in chat | No response is sent (no error spam in chat) | 🔴 |
| AC-010c | Multiple simultaneous commands | 5 viewers type `:deer: donate` at the same time | streamer.bot processes all 5 | Each viewer gets a response (5 responses total) | 🟡 |
| AC-010d | Command with extra text | A viewer types `:deer: donate please help` in chat | streamer.bot detects the command | The bot still responds with the donation link (command prefix match) | 🟡 |

#### US-011: Point Command

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-011a | Viewer has points | Viewer "@viewer1" has 500 points in the database | They type `:deer: point` in chat | The bot responds with "🦌 @viewer1 has 500 points!" within 2 seconds | 🔴 |
| AC-011b | Viewer has 0 points | Viewer "@newviewer" has never donated (0 points) | They type `:deer: point` in chat | The bot responds with "🦌 @newviewer has 0 points. Donate to earn points!" | 🔴 |
| AC-011c | Backend API down | The Go backend is not responding | A viewer types `:deer: point` in chat | The bot responds with "🦌 Points system is temporarily unavailable" | 🔴 |
| AC-011d | API timeout | The Go backend takes >5 seconds to respond | A viewer types `:deer: point` in chat | The bot responds with a timeout error message after 5 seconds | 🟡 |

#### US-012: Streamer.bot Action Configuration

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-012a | Donate action configured | streamer.bot is running with the Donate action configured | A chat message contains `:deer: donate` | The Donate action fires and sends the donation link | 🔴 |
| AC-012b | Point action configured | streamer.bot is running with the Point action configured | A chat message contains `:deer: point` | The Point action fires, calls Go backend API, sends point response | 🔴 |
| AC-012c | Point action — API error | The Go backend returns a 500 error | The Point action receives the error response | The action sends a fallback error message instead of crashing | 🔴 |
| AC-012d | Action logs | streamer.bot is running | Any action fires | The action execution is logged in streamer.bot (timestamp, trigger, result) | 🟡 |

---

### 3.3 E-03: Points Engine

#### US-020: EasyDonate Donation Sync

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-020a | New donations found | EasyDonate API returns 3 new donations | The sync process processes them | Each donation is stored with: donor_name, amount_thb, donation_time, easydonate_id | 🔴 |
| AC-020b | Idempotent sync | Donation with easydonate_id "ed-123" already exists | The sync runs again and finds the same donation | The existing record is not duplicated (skip or update) | 🔴 |
| AC-020c | Rate limit handling | EasyDonate API returns 429 Too Many Requests | The sync process detects the error | The process backs off and retries after the rate limit window (respect Retry-After header) | 🔴 |
| AC-020d | Empty response | EasyDonate API returns 0 new donations | The sync process processes the response | No records created, sync completes without error | 🟡 |
| AC-020e | API unavailable | EasyDonate API is down (500 or timeout) | The sync process tries to fetch | The process logs the error and retries on next sync cycle | 🟡 |

#### US-021: Name Matching Engine

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-021a | Exact match (case-insensitive) | Donation from "deer123", subscriber "@deer123" exists | The matching engine runs | Donation matched to subscriber "@deer123" | 🔴 |
| AC-021b | Fuzzy match (strip @) | Donation from "@viewer1", subscriber "viewer1" exists | The matching engine runs | Donation matched to subscriber "viewer1" | 🔴 |
| AC-021c | Anonymous donation | Donation from "anonymous" | The matching engine runs | Donation flagged as "unmatched", no points awarded | 🔴 |
| AC-021d | Empty donor name | Donation with donor_name = "" | The matching engine runs | Donation flagged as "unmatched" | 🔴 |
| AC-021e | Multiple possible matches | Donation from "deer", subscribers "deer123" and "deer456" exist | The matching engine runs | Best match selected (highest similarity score), match logged for review | 🟡 |
| AC-021f | No match found | Donation from "unknown_user", no close match in subscribers | The matching engine runs | Donation flagged as "unmatched" with donor_name stored for manual review | 🟡 |

#### US-022: Point Calculation & Query API

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-022a | Happy path | Viewer "@viewer1" has 3 matched donations: 100 THB + 200 THB + 200 THB | GET /api/v1/points/viewer1 | 200 OK with {youtube_handle: "@viewer1", total_points: 500} | 🔴 |
| AC-022b | No donations | Viewer "@newviewer" has 0 donations | GET /api/v1/points/newviewer | 200 OK with {youtube_handle: "@newviewer", total_points: 0} | 🔴 |
| AC-022c | Non-existent handle | Handle "@doesnotexist" is not in subscribers table | GET /api/v1/points/doesnotexist | 200 OK with {youtube_handle: "@doesnotexist", total_points: 0} (not 404) | 🔴 |
| AC-022d | Only matched donations count | Viewer has 2 matched donations (300 THB) and 1 unmatched donation (100 THB) | GET /api/v1/points/{handle} | 200 OK with total_points: 300 (unmatched excluded) | 🔴 |
| AC-022e | Case-insensitive lookup | Viewer is stored as "@Viewer1" | GET /api/v1/points/viewer1 (lowercase) | 200 OK with correct points (case-insensitive match) | 🟡 |

---

### 3.4 E-04: Web Scoreboard

#### US-030: Public Scoreboard Page

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-030a | Happy path | The scoreboard has 10 viewers with points | A viewer visits the scoreboard URL | A ranked list is displayed showing: rank, display name, points (highest first) | 🟡 |
| AC-030b | Data freshness | New donations are processed and points updated | The scoreboard is refreshed | Updated point totals are visible (data freshness <60s) | 🟡 |
| AC-030c | Responsive design | The scoreboard is accessed from a mobile device | The page loads | The layout adapts to mobile screen size | 🟡 |
| AC-030d | Backend down | The Go backend API is not responding | A viewer visits the scoreboard | A friendly error message is shown: "Scoreboard temporarily unavailable" | 🟡 |
| AC-030e | Empty scoreboard | No viewers have points yet | A viewer visits the scoreboard | A message is shown: "No contributors yet. Be the first!" | 🟡 |

#### US-031: Scoreboard API Endpoint

| AC ID | Scenario | Given | When | Then | Priority |
|-------|---------|-------|------|------|----------|
| AC-031a | Happy path | 50 viewers have points in the database | GET /api/v1/scoreboard | 200 OK with JSON array of viewers sorted by points descending, each with: rank, display_name, youtube_handle, total_points | 🟡 |
| AC-031b | Empty scoreboard | No viewers have points | GET /api/v1/scoreboard | 200 OK with empty array [] (not an error) | 🟡 |
| AC-031c | Pagination | 200 viewers have points | GET /api/v1/scoreboard?page=1&limit=50 | 200 OK with first 50 viewers | 🟡 |
| AC-031d | Page 2 | 200 viewers have points | GET /api/v1/scoreboard?page=2&limit=50 | 200 OK with viewers 51-100 | 🟡 |

---

## 4. Acceptance Criteria Summary

| Requirement | Total ACs | 🔴 Must Have | 🟡 Should Have | Status |
|------------|----------|-------------|---------------|--------|
| US-001 Capture Subscriber | 6 | 4 | 2 | Draft |
| US-002 Subscriber API | 5 | 3 | 2 | Draft |
| US-003 YouTube API Polling | 7 | 5 | 2 | Draft |
| US-010 Donate Command | 4 | 2 | 2 | Draft |
| US-011 Point Command | 4 | 3 | 1 | Draft |
| US-012 SB Action Config | 4 | 3 | 1 | Draft |
| US-020 Donation Sync | 5 | 3 | 2 | Draft |
| US-021 Name Matching | 6 | 4 | 2 | Draft |
| US-022 Point Query API | 5 | 4 | 1 | Draft |
| US-030 Scoreboard Page | 5 | 0 | 5 | Draft |
| US-031 Scoreboard API | 4 | 0 | 4 | Draft |
| **Total** | **53** | **31** | **22** | |

---

## 5. Acceptance Criteria Traceability

| AC ID | User Story | Test Case | Test Status |
|-------|-----------|-----------|------------|
| AC-001a | US-001 | TC-001 | ⬜ Not Run |
| AC-001b | US-001 | TC-002 | ⬜ Not Run |
| AC-001c | US-001 | TC-003 | ⬜ Not Run |
| AC-001d | US-001 | TC-004 | ⬜ Not Run |
| AC-002a | US-002 | TC-005 | ⬜ Not Run |
| AC-002b | US-002 | TC-006 | ⬜ Not Run |
| AC-002c | US-002 | TC-007 | ⬜ Not Run |
| AC-002d | US-002 | TC-008 | ⬜ Not Run |
| AC-002e | US-002 | TC-009 | ⬜ Not Run |
| AC-003a | US-003 | TC-047 | ⬜ Not Run |
| AC-003b | US-003 | TC-048 | ⬜ Not Run |
| AC-003c | US-003 | TC-049 | ⬜ Not Run |
| AC-003d | US-003 | TC-050 | ⬜ Not Run |
| AC-003e | US-003 | TC-051 | ⬜ Not Run |
| AC-003f | US-003 | TC-052 | ⬜ Not Run |
| AC-003g | US-003 | TC-053 | ⬜ Not Run |
| AC-010a | US-010 | TC-010 | ⬜ Not Run |
| AC-010b | US-010 | TC-011 | ⬜ Not Run |
| AC-010c | US-010 | TC-012 | ⬜ Not Run |
| AC-010d | US-010 | TC-013 | ⬜ Not Run |
| AC-011a | US-011 | TC-014 | ⬜ Not Run |
| AC-011b | US-011 | TC-015 | ⬜ Not Run |
| AC-011c | US-011 | TC-016 | ⬜ Not Run |
| AC-011d | US-011 | TC-017 | ⬜ Not Run |
| AC-012a | US-012 | TC-018 | ⬜ Not Run |
| AC-012b | US-012 | TC-019 | ⬜ Not Run |
| AC-012c | US-012 | TC-020 | ⬜ Not Run |
| AC-012d | US-012 | TC-021 | ⬜ Not Run |
| AC-020a | US-020 | TC-022 | ⬜ Not Run |
| AC-020b | US-020 | TC-023 | ⬜ Not Run |
| AC-020c | US-020 | TC-024 | ⬜ Not Run |
| AC-020d | US-020 | TC-025 | ⬜ Not Run |
| AC-020e | US-020 | TC-026 | ⬜ Not Run |
| AC-021a | US-021 | TC-027 | ⬜ Not Run |
| AC-021b | US-021 | TC-028 | ⬜ Not Run |
| AC-021c | US-021 | TC-029 | ⬜ Not Run |
| AC-021d | US-021 | TC-030 | ⬜ Not Run |
| AC-021e | US-021 | TC-031 | ⬜ Not Run |
| AC-021f | US-021 | TC-032 | ⬜ Not Run |
| AC-022a | US-022 | TC-033 | ⬜ Not Run |
| AC-022b | US-022 | TC-034 | ⬜ Not Run |
| AC-022c | US-022 | TC-035 | ⬜ Not Run |
| AC-022d | US-022 | TC-036 | ⬜ Not Run |
| AC-022e | US-022 | TC-037 | ⬜ Not Run |
| AC-030a | US-030 | TC-038 | ⬜ Not Run |
| AC-030b | US-030 | TC-039 | ⬜ Not Run |
| AC-030c | US-030 | TC-040 | ⬜ Not Run |
| AC-030d | US-030 | TC-041 | ⬜ Not Run |
| AC-030e | US-030 | TC-042 | ⬜ Not Run |
| AC-031a | US-031 | TC-043 | ⬜ Not Run |
| AC-031b | US-031 | TC-044 | ⬜ Not Run |
| AC-031c | US-031 | TC-045 | ⬜ Not Run |
| AC-031d | US-031 | TC-046 | ⬜ Not Run |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[011_business_objective]] | Objectives these criteria verify |
| [[012_user_stories]] | ACs in Given/When/Then format for user stories |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119, ISO/IEC/IEEE 29148
> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
