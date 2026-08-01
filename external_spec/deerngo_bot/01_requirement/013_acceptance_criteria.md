---
document_type: Acceptance Criteria (ATDD/BDD)
version: "0.2"
status: Draft
author: "PO"
created: "2026-07-29"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
ba_owner: "PO"
qa_lead: "TBD"
classification: "Internal"
tags: [acceptance-criteria, bdd, atdd, given-when-then, vrm, members, privacy]
standard_ref:
  - SWEBOK v4 — Requirements
  - ISO/IEC/IEEE 29119 — Software Testing
  - ISO/IEC/IEEE 29148 — Requirements Engineering
---

# Acceptance Criteria (ATDD/BDD)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Change note:** The YouTube subscriber-polling model is superseded by explicit member registration through streamer.bot. US-003 and its YouTube polling criteria are retired from the active Phase 1 scope.

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
| 0.1 | 2026-07-29 | PO | Initial subscriber-based criteria |
| 0.2 | 2026-08-02 | PO | Replaced subscriber capture with member registration, privacy visibility, and exact matching criteria |

---

## 1. Purpose

This document defines acceptance criteria for Deerngo Bot Phase 1 using Given/When/Then-style scenarios. Each active criterion is testable and traceable to a current user story.

## 2. Acceptance Criteria Standards

| Criterion | Description |
|-----------|-------------|
| **Testable** | Can be verified by an automated or manual test |
| **Unambiguous** | Has one interpretation |
| **Complete** | Covers happy path, edge cases, and failure behavior |
| **Independent** | Can be evaluated without hidden assumptions |
| **Privacy-aware** | Does not expose data beyond the approved purpose |

---

## 3. Acceptance Criteria by Requirement

### 3.1 E-01: Member Registration

#### US-001: Register Viewer as Member

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-001a | New registration | streamer.bot receives `:deer: register` with the actual chat author's YouTube user ID and current handle | The backend processes the registration | An active member is created with 0 points, `public_visibility=true`, `registered_at`, and no stored YouTube display name | 🔴 |
| AC-001b | Same member registers again | An active member with the same YouTube user ID and normalized handle exists | The viewer registers again | No database record or point total changes; the bot returns the current-points already-registered response | 🔴 |
| AC-001c | Handle change | An active member with the same YouTube user ID registers a different normalized handle | The backend processes the registration | The old member becomes inactive with points preserved; a new active member is created with 0 points; no automatic transfer occurs | 🔴 |
| AC-001d | Active handle conflict | The normalized handle belongs to another active YouTube user ID | A new registration is processed | Registration is rejected with a handle-in-use response; no member or points are changed | 🔴 |
| AC-001e | Typed handle ignored | The command text contains a handle different from the actual chat author | streamer.bot sends the event | The backend uses only the streamer.bot identity and ignores the typed handle | 🔴 |
| AC-001f | Registration confirmation | A new member is created | The backend returns success | The bot says: `🦌 Registered! You start with 0 points. When you earn points, your normalized YouTube handle and score may appear on the public scoreboard.` | 🔴 |
| AC-001g | Backend unavailable | streamer.bot is online but the Go backend cannot be reached | A viewer uses `:deer: register` | The bot says registration is temporarily unavailable and no partial member is created | 🔴 |

#### US-002: Member Registration API

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-002a | Valid registration | A request contains valid `youtube_user_id` and `youtube_handle` | `POST /api/v1/members/register` is processed | `201 Created` returns the new active member with 0 points | 🔴 |
| AC-002b | Same-handle repeat | An active member with the same user ID and normalized handle exists | The endpoint is called again | `200 OK` returns an already-registered result and no member data changes | 🔴 |
| AC-002c | New handle for same user | An active member with the same user ID has a different current handle | The endpoint is called | The previous row becomes inactive and a new active row with 0 points is created | 🔴 |
| AC-002d | Active handle conflict | The normalized handle belongs to another active user ID | The endpoint is called | `409 Conflict` returns a handle-in-use error | 🔴 |
| AC-002e | Invalid required fields | `youtube_user_id` or `youtube_handle` is missing or invalid | Validation runs | `400 Bad Request` returns field-level errors | 🔴 |
| AC-002f | Handle normalization | A handle has whitespace or a leading `@` | The endpoint normalizes it | Stored and matched value is trimmed lowercase text without the leading `@` | 🟡 |

### 3.2 E-02: Bot Commands

#### US-010: Donate Command

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-010a | Happy path | streamer.bot is running and connected to the live chat | A viewer types `:deer: donate` | The bot responds with `🦌 Donate here: https://easydonate.app/deerngo0` within 2 seconds | 🔴 |
| AC-010b | streamer.bot offline | streamer.bot is not running | A viewer types the command | No response is sent because no bot is available | 🔴 |
| AC-010c | Concurrent commands | Five viewers type the command simultaneously | streamer.bot processes the messages | Five responses are sent | 🟡 |
| AC-010d | Extra text | A message begins with `:deer: donate` and contains extra text | The command matcher evaluates it | The donation link response is sent | 🟡 |

#### US-011: Point Command

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-011a | Public member with points | An active public member has points greater than 0 | The member types `:deer: point` | The bot shows the exact point total | 🔴 |
| AC-011b | Public member with zero points | An active public member has 0 points | The member types `:deer: point` | The bot shows 0 points and a donation prompt | 🔴 |
| AC-011c | Private member with points | An active private member has 563 points | The member types `:deer: point` | The bot shows `between 500–600`, not the exact total | 🔴 |
| AC-011d | Private member with zero points | An active private member has 0 points | The member types `:deer: point` | The bot shows `🦌 Your points are between 0–100.` | 🔴 |
| AC-011e | Unregistered viewer | The chat author has no active member | The viewer types `:deer: point` | The bot says: `🦌 You are not registered yet. Use :deer: register first.` | 🔴 |
| AC-011f | Backend unavailable | A registered viewer asks for points while the backend is unavailable | streamer.bot calls the API | A friendly temporary-unavailable response is sent without internal details | 🔴 |

#### US-012: Streamer.bot Member and Command Actions

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-012a | Register action | A viewer types `:deer: register` | streamer.bot processes the message | It sends the actual YouTube user ID and current handle to the backend; typed handle text is ignored | 🔴 |
| AC-012b | Public action | An active member types `:deer: public` | streamer.bot calls the visibility API | The member is set to `public_visibility=true` and receives confirmation | 🔴 |
| AC-012c | Private action | An active member types `:deer: private` | streamer.bot calls the visibility API | The member is set to `public_visibility=false` and receives confirmation | 🔴 |
| AC-012d | Donate action | A viewer types `:deer: donate` | streamer.bot evaluates the command | The EasyDonate link is posted | 🔴 |
| AC-012e | Point action | A viewer types `:deer: point` | streamer.bot calls the backend | It posts the exact or banded result according to member visibility | 🔴 |
| AC-012f | API failure fallback | The backend returns an error | streamer.bot receives the error | A friendly fallback message is sent; streamer.bot does not crash | 🔴 |
| AC-012g | Safe action logs | Any action executes | streamer.bot records the action | Logs contain trigger, timestamp, and result but no secrets | 🟡 |

### 3.3 E-03: Points Engine

#### US-020: EasyDonate Donation Ingestion

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-020a | Webhook donation | EasyDonate sends the documented payload with `referenceNo`, `donatorName`, `amount`, `channelName`, `donateMessage`, and `time` | The webhook request passes path and payload validation | A private donation record is stored with normalized fields and `source=webhook` | 🔴 |
| AC-020b | Duplicate webhook | A donation with the same `referenceNo` already exists | EasyDonate sends it again | No duplicate donation or points are created | 🔴 |
| AC-020c | API fallback | The EasyDonate API key has donation-read scope | The fallback sync runs | New donations are imported with `source=api_poll` | 🔴 |
| AC-020d | Provider rate limit | EasyDonate returns `429` | The fallback sync handles the response | The process backs off and follows provider rate-limit guidance | 🔴 |
| AC-020e | Empty sync | EasyDonate returns no new donations | The sync completes | No records are created and no error is raised | 🟡 |
| AC-020f | Provider unavailable | Webhook or polling cannot reach EasyDonate | The ingestion path fails | The error is logged safely and the next reconciliation cycle retries | 🟡 |
| AC-020g | Invalid webhook | The webhook path token is invalid or payload validation fails | The backend receives the request | It rejects the request without storing a donation | 🔴 |

#### US-021: Exact Member Donation Matching

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-021a | Exact normalized match | Donor name is `deer123` and an active member handle is `@Deer123` | Matching runs | The donation matches the active member | 🔴 |
| AC-021b | Harmless formatting | Donor and handle differ only by whitespace, capitalization, or leading `@` | Normalization runs | Values compare equal | 🔴 |
| AC-021c | Inactive member | An inactive member has the matching handle | Matching runs | The donation is not awarded to that inactive member | 🔴 |
| AC-021d | Before registration | `donation_time` is earlier than `member.registered_at` | Matching runs | The donation is marked not eligible and awards no points | 🔴 |
| AC-021e | No active match | No active member has the normalized donor name | Matching runs | The donation remains uncredited and is retained privately for reconciliation | 🟡 |
| AC-021f | Private donor data | The donor name or donation message may identify a person | The donation is stored | It is never returned by the public scoreboard API | 🔴 |
| AC-021g | Single point application | An eligible exact match exists | Points are applied | `amount_thb` is added to the member's `total_points` exactly once | 🔴 |

#### US-022: Member Points Query API

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-022a | Public member points | An active public member has eligible matched donations totaling 500 THB | The member is queried | The response contains exact total 500 points | 🔴 |
| AC-022b | No eligible donations | An active member has no eligible donations | The member is queried | The response contains 0 points | 🔴 |
| AC-022c | Identity query | An active member is queried by actual chat identity | `GET /api/v1/members/{youtube_user_id}/points` runs | `200 OK` returns a visibility-aware point response | 🔴 |
| AC-022d | Not registered | The user ID has no active member | The endpoint is called | A member-not-registered result is returned | 🔴 |
| AC-022e | Private band | An active private member has 563 points | The endpoint is called | Only `lower=500` and `upper=600` are exposed | 🔴 |
| AC-022f | Visibility change | A member changes `public_visibility` | The query runs afterward | The response follows the latest visibility without changing points | 🔴 |

### 3.4 E-04: Web Scoreboard

#### US-030: Public Scoreboard Page

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-030a | Active public contributors | Active public members have points greater than 0 | The page loads | Normalized handles and point totals appear ranked descending | 🟡 |
| AC-030b | Private member hidden | An active member is private | The page loads | The member is not displayed | 🟡 |
| AC-030c | Inactive member hidden | A member is inactive | The page loads | The member is not displayed | 🟡 |
| AC-030d | No contributors | Active public members have 0 points only | The page loads | `No contributors yet. Be the first!` appears | 🟡 |
| AC-030e | Freshness | A new donation is processed | The page refreshes | Data is fresher than 60 seconds | 🟡 |
| AC-030f | Backend unavailable | The API cannot be reached | The page loads | `Scoreboard temporarily unavailable` appears | 🟡 |

#### US-031: Public Scoreboard API

| AC ID | Scenario | Given | When | Then | Priority |
|-------|----------|-------|------|------|----------|
| AC-031a | Eligible ranking | Active public members have points greater than 0 | `GET /api/v1/scoreboard` runs | `200 OK` returns rank, normalized handle, and total points sorted descending | 🟡 |
| AC-031b | Empty result | No active public member has points greater than 0 | The endpoint is queried | `200 OK` returns an empty array | 🟡 |
| AC-031c | Visibility/status filtering | Private, inactive, and 0-point members exist | The endpoint is queried | None of those members appear | 🟡 |
| AC-031d | First page | Eligible members exceed 50 | `?page=1&limit=50` is supplied | The first 50 eligible members are returned | 🟡 |
| AC-031e | Second page | Eligible members exceed 100 | `?page=2&limit=50` is supplied | Eligible members 51–100 are returned | 🟡 |
| AC-031f | Public data minimization | The response is serialized | The response is inspected | It contains no YouTube display name, raw donor name, donation message, or private-member data | 🟡 |

---

## 4. Acceptance Criteria Summary

| Requirement | Total ACs | 🔴 Must Have | 🟡 Should Have | Status |
|------------|----------:|-------------:|---------------:|--------|
| US-001 Member Registration | 7 | 7 | 0 | Draft |
| US-002 Member API | 6 | 5 | 1 | Draft |
| US-010 Donate Command | 4 | 2 | 2 | Draft |
| US-011 Point Command | 6 | 6 | 0 | Draft |
| US-012 Bot Actions | 7 | 6 | 1 | Draft |
| US-020 Donation Ingestion | 7 | 5 | 2 | Draft |
| US-021 Exact Matching | 7 | 6 | 1 | Draft |
| US-022 Member Points API | 6 | 6 | 0 | Draft |
| US-030 Scoreboard Page | 6 | 0 | 6 | Draft |
| US-031 Scoreboard API | 6 | 0 | 6 | Draft |
| **Total active** | **62** | **43** | **19** | |

> **Superseded:** US-003 and its 7 former ACs are retired from the active Phase 1 count.

---

## 5. Acceptance Criteria Traceability

| AC ID | User Story | Test Case | Test Status |
|-------|-----------|-----------|-------------|
| AC-001a | US-001 | TC-M001 | ⬜ Not Run |
| AC-001b | US-001 | TC-M002 | ⬜ Not Run |
| AC-001c | US-001 | TC-M003 | ⬜ Not Run |
| AC-001d | US-001 | TC-M004 | ⬜ Not Run |
| AC-001e | US-001 | TC-M005 | ⬜ Not Run |
| AC-001f | US-001 | TC-M006 | ⬜ Not Run |
| AC-001g | US-001 | TC-M007 | ⬜ Not Run |
| AC-002a | US-002 | TC-M008 | ⬜ Not Run |
| AC-002b | US-002 | TC-M009 | ⬜ Not Run |
| AC-002c | US-002 | TC-M010 | ⬜ Not Run |
| AC-002d | US-002 | TC-M011 | ⬜ Not Run |
| AC-002e | US-002 | TC-M012 | ⬜ Not Run |
| AC-002f | US-002 | TC-M013 | ⬜ Not Run |
| AC-010a | US-010 | TC-M014 | ⬜ Not Run |
| AC-010b | US-010 | TC-M015 | ⬜ Not Run |
| AC-010c | US-010 | TC-M016 | ⬜ Not Run |
| AC-010d | US-010 | TC-M017 | ⬜ Not Run |
| AC-011a | US-011 | TC-M018 | ⬜ Not Run |
| AC-011b | US-011 | TC-M019 | ⬜ Not Run |
| AC-011c | US-011 | TC-M020 | ⬜ Not Run |
| AC-011d | US-011 | TC-M021 | ⬜ Not Run |
| AC-011e | US-011 | TC-M022 | ⬜ Not Run |
| AC-011f | US-011 | TC-M023 | ⬜ Not Run |
| AC-012a | US-012 | TC-M024 | ⬜ Not Run |
| AC-012b | US-012 | TC-M025 | ⬜ Not Run |
| AC-012c | US-012 | TC-M026 | ⬜ Not Run |
| AC-012d | US-012 | TC-M027 | ⬜ Not Run |
| AC-012e | US-012 | TC-M028 | ⬜ Not Run |
| AC-012f | US-012 | TC-M029 | ⬜ Not Run |
| AC-012g | US-012 | TC-M030 | ⬜ Not Run |
| AC-020a | US-020 | TC-M031 | ⬜ Not Run |
| AC-020b | US-020 | TC-M032 | ⬜ Not Run |
| AC-020c | US-020 | TC-M033 | ⬜ Not Run |
| AC-020d | US-020 | TC-M034 | ⬜ Not Run |
| AC-020e | US-020 | TC-M035 | ⬜ Not Run |
| AC-020f | US-020 | TC-M036 | ⬜ Not Run |
| AC-020g | US-020 | TC-M037 | ⬜ Not Run |
| AC-021a | US-021 | TC-M038 | ⬜ Not Run |
| AC-021b | US-021 | TC-M039 | ⬜ Not Run |
| AC-021c | US-021 | TC-M040 | ⬜ Not Run |
| AC-021d | US-021 | TC-M041 | ⬜ Not Run |
| AC-021e | US-021 | TC-M042 | ⬜ Not Run |
| AC-021f | US-021 | TC-M043 | ⬜ Not Run |
| AC-021g | US-021 | TC-M044 | ⬜ Not Run |
| AC-022a | US-022 | TC-M045 | ⬜ Not Run |
| AC-022b | US-022 | TC-M046 | ⬜ Not Run |
| AC-022c | US-022 | TC-M047 | ⬜ Not Run |
| AC-022d | US-022 | TC-M048 | ⬜ Not Run |
| AC-022e | US-022 | TC-M049 | ⬜ Not Run |
| AC-022f | US-022 | TC-M050 | ⬜ Not Run |
| AC-030a | US-030 | TC-M051 | ⬜ Not Run |
| AC-030b | US-030 | TC-M052 | ⬜ Not Run |
| AC-030c | US-030 | TC-M053 | ⬜ Not Run |
| AC-030d | US-030 | TC-M054 | ⬜ Not Run |
| AC-030e | US-030 | TC-M055 | ⬜ Not Run |
| AC-030f | US-030 | TC-M056 | ⬜ Not Run |
| AC-031a | US-031 | TC-M057 | ⬜ Not Run |
| AC-031b | US-031 | TC-M058 | ⬜ Not Run |
| AC-031c | US-031 | TC-M059 | ⬜ Not Run |
| AC-031d | US-031 | TC-M060 | ⬜ Not Run |
| AC-031e | US-031 | TC-M061 | ⬜ Not Run |
| AC-031f | US-031 | TC-M062 | ⬜ Not Run |

### Superseded Traceability

| Former AC | Former Story | Status | Reason |
|-----------|--------------|--------|--------|
| AC-003a → AC-003g | US-003 | ❌ Superseded | YouTube polling removed from active MVP after MM06 |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[011_business_objective]] | Objectives these criteria verify |
| [[012_user_stories]] | User stories these criteria detail |
| [[022_API_specification]] | API contracts to verify |
| [[023_database_schema_DDL]] | Member and donation data constraints |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Approved change decision |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119, ISO/IEC/IEEE 29148
> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Active criteria:** 62 across 10 active stories.
