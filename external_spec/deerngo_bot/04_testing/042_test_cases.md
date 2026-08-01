---
document_type: Test Cases
version: "0.2"
status: Draft
author: "QA Engineer / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [test-cases, test-scenarios, swebok, iso-29119, vrm, members, privacy]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Test Cases — Active Phase 1 MVP

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope:** Explicit member registration, streamer.bot commands, EasyDonate ingestion, normalized exact matching, member points, visibility, and public scoreboard.
>
> **Superseded:** Original subscriber polling cases TC-012–TC-018 are historical and excluded from active regression.

---

## 1. Purpose

This document defines 62 active test cases mapped one-to-one to the 62 active acceptance criteria in `013_acceptance_criteria.md`. All cases are currently **⬜ Not Run**; this document is a test design, not execution evidence.

## 2. Test Case Index

| Module | Cases | Automated Target | Manual Target | Status |
|--------|-------|:----------------:|:-------------:|--------|
| E-01 Member Registration | TC-M001–M013 | 11 | 2 | ⬜ Not Run |
| E-02 Bot Commands | TC-M014–M030 | 13 | 4 | ⬜ Not Run |
| E-03 Points Engine | TC-M031–M050 | 18 | 2 | ⬜ Not Run |
| E-04 Scoreboard | TC-M051–M062 | 9 | 3 | ⬜ Not Run |
| **Total** | **TC-M001–M062** | **51** | **11** | |

## 3. Common Test Data

| Identifier | Value |
|------------|-------|
| Active public member | user `UC-test-001`, handle `testviewer1`, points 500 |
| Active private member | user `UC-test-002`, handle `privateviewer`, points 563 |
| Active zero-point member | user `UC-test-003`, handle `newviewer`, points 0 |
| Inactive member | user `UC-test-004`, handle `oldviewer`, points 700 |
| Normalization | trim, remove one leading `@`, lowercase |
| Provider reference | `EZDN-TEST-001` |
| Test DB | isolated PostgreSQL `deerngo_test` |

---

## 4. E-01 — Member Registration

### TC-M001 — New member registration

| Field | Value |
|-------|-------|
| Requirement | AC-001a → US-001 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** no active member exists for `UC-test-001`.
**When** `POST /api/v1/members/register` receives `youtube_user_id=UC-test-001`, `youtube_handle=@TestViewer1`.
**Then** response is `201`; an active member exists with normalized handle `testviewer1`, 0 points, `public_visibility=true`, `registered_at`, and no display name column/value.

### TC-M002 — Same-handle registration is idempotent

| Field | Value |
|-------|-------|
| Requirement | AC-001b → US-001 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** active member `UC-test-001/testviewer1` has 500 points.
**When** the same registration is sent again.
**Then** response is `200` with already-registered result; member count, points, timestamps, and handle do not change.

### TC-M003 — Changed handle creates a new zero-point member

| Field | Value |
|-------|-------|
| Requirement | AC-001c → US-001 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** active member `UC-test-001/testviewer1` has 500 points.
**When** the same user registers as `NewHandle`.
**Then** old row is inactive with 500 points; new row is active with `newhandle` and 0 points; no transfer occurs.

### TC-M004 — Active handle conflict is rejected

| Field | Value |
|-------|-------|
| Requirement | AC-001d → US-001 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** active member `UC-test-001` owns `testviewer1`.
**When** `UC-other` registers `@TESTVIEWER1`.
**Then** response is `409 HANDLE_IN_USE`; no member/status/points change occurs.

### TC-M005 — Typed handle is ignored

| Field | Value |
|-------|-------|
| Requirement | AC-001e → US-001 |
| Priority | 🔴 |
| Type | System |
| Automated | No |

**Given** streamer.bot receives a command from `UC-test-001` whose message contains another viewer's handle.
**When** streamer.bot posts the registration request.
**Then** backend stores only the supplied actual identity and ignores command text.

### TC-M006 — New registration response wording

| Field | Value |
|-------|-------|
| Requirement | AC-001f → US-001 |
| Priority | 🔴 |
| Type | Manual |
| Automated | No |

**Given** a new member is created through streamer.bot.
**When** success is returned.
**Then** chat contains the approved registration/scoreboard notice.

### TC-M007 — Backend unavailable does not create partial member

| Field | Value |
|-------|-------|
| Requirement | AC-001g → US-001 |
| Priority | 🔴 |
| Type | Manual |
| Automated | No |

**Given** streamer.bot is online but backend is stopped.
**When** viewer types `:deer: register`.
**Then** friendly temporary-unavailable response is used; after recovery, no partial row exists and retry can succeed.

### TC-M008 — Registration API creates member

| Field | Value |
|-------|-------|
| Requirement | AC-002a → US-002 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** valid user ID and handle.
**When** POST endpoint is called.
**Then** `201 Created` returns new active member with 0 points.

### TC-M009 — Registration API same-handle repeat

| Field | Value |
|-------|-------|
| Requirement | AC-002b → US-002 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** same active user/handle exists.
**When** POST endpoint is called again.
**Then** `200 OK` already-registered response and no data mutation.

### TC-M010 — Registration API handle-change transaction

| Field | Value |
|-------|-------|
| Requirement | AC-002c → US-002 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** same user has an active old-handle row.
**When** new handle registration is processed.
**Then** deactivation and new active 0-point creation are atomic; a simulated failure leaves the pre-request state intact.

### TC-M011 — Registration API active-handle conflict

| Field | Value |
|-------|-------|
| Requirement | AC-002d → US-002 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** another active user owns normalized handle.
**When** endpoint receives conflicting registration.
**Then** `409 Conflict` and no mutation.

### TC-M012 — Registration API field validation

| Field | Value |
|-------|-------|
| Requirement | AC-002e → US-002 |
| Priority | 🔴 |
| Type | Integration |
| Automated | Yes |

**Given** missing/empty/invalid user ID or handle.
**When** endpoint validates request.
**Then** `400` with field-level errors and no row.

### TC-M013 — Handle normalization

| Field | Value |
|-------|-------|
| Requirement | AC-002f → US-002 |
| Priority | 🟡 |
| Type | Unit |
| Automated | Yes |

**Given** input `  @DeEr123  `.
**When** normalization runs.
**Then** stored/matched value is `deer123`.

---

## 5. E-02 — Bot Commands

### TC-M014 — Donate command happy path

**Requirement:** AC-010a → US-010 | **Priority:** 🔴 | **Type:** Manual

**Given** streamer.bot is connected.
**When** viewer types `:deer: donate`.
**Then** chat response contains `https://easydonate.app/deerngo0` within 2 seconds.

### TC-M015 — Donate command with bot offline

**Requirement:** AC-010b → US-010 | **Priority:** 🔴 | **Type:** Manual

**Given** streamer.bot is offline.
**When** command is typed.
**Then** no bot response is expected.

### TC-M016 — Concurrent donate commands

**Requirement:** AC-010c → US-010 | **Priority:** 🟡 | **Type:** System

**Given** five command messages are submitted concurrently.
**When** streamer.bot processes them.
**Then** five responses are observed.

### TC-M017 — Donate command extra text

**Requirement:** AC-010d → US-010 | **Priority:** 🟡 | **Type:** System

**Given** message starts with `:deer: donate` and includes extra text.
**When** command matcher runs.
**Then** link response is sent.

### TC-M018 — Public member exact points

**Requirement:** AC-011a → US-011 | **Priority:** 🔴 | **Type:** Integration

**Given** active public member has 500 points.
**When** points API is queried using actual user ID.
**Then** exact 500 response is returned and streamer.bot posts exact amount.

### TC-M019 — Public member zero points

**Requirement:** AC-011b → US-011 | **Priority:** 🔴 | **Type:** Integration

**Given** active public member has 0 points.
**When** point command runs.
**Then** chat shows 0 and donation prompt.

### TC-M020 — Private member band

**Requirement:** AC-011c → US-011 | **Priority:** 🔴 | **Type:** Integration

**Given** active private member has 563 points.
**When** point command runs.
**Then** response contains 500–600 and not 563.

### TC-M021 — Private member zero band

**Requirement:** AC-011d → US-011 | **Priority:** 🔴 | **Type:** Integration

**Given** active private member has 0.
**When** point command runs.
**Then** response is `between 0–100`.

### TC-M022 — Unregistered point query

**Requirement:** AC-011e → US-011 | **Priority:** 🔴 | **Type:** Integration

**Given** no active member exists.
**When** point endpoint is queried.
**Then** `MEMBER_NOT_REGISTERED` maps to `Use :deer: register first`.

### TC-M023 — Point query backend failure

**Requirement:** AC-011f → US-011 | **Priority:** 🔴 | **Type:** Manual

**Given** backend cannot be reached.
**When** streamer.bot calls points API.
**Then** friendly temporary-unavailable message appears and no internal details leak.

### TC-M024 — Register streamer.bot action identity mapping

**Requirement:** AC-012a → US-012 | **Priority:** 🔴 | **Type:** Manual

Verify action maps actual user ID/current handle and does not map message-supplied handle.

### TC-M025 — Public visibility action

**Requirement:** AC-012b → US-012 | **Priority:** 🔴 | **Type:** Manual

Verify `:deer: public` updates active member visibility and confirms.

### TC-M026 — Private visibility action

**Requirement:** AC-012c → US-012 | **Priority:** 🔴 | **Type:** Manual

Verify `:deer: private` updates active member visibility and confirms.

### TC-M027 — Donate action mapping

**Requirement:** AC-012d → US-012 | **Priority:** 🔴 | **Type:** Manual

Verify Donate action posts configured EasyDonate URL.

### TC-M028 — Point action mapping

**Requirement:** AC-012e → US-012 | **Priority:** 🔴 | **Type:** Manual

Verify Point action posts exact or banded response from API.

### TC-M029 — Action API failure fallback

**Requirement:** AC-012f → US-012 | **Priority:** 🔴 | **Type:** Manual

Stop backend; verify friendly fallback and no action crash.

### TC-M030 — Safe streamer.bot action logs

**Requirement:** AC-012g → US-012 | **Priority:** 🟡 | **Type:** System

Inspect logs; verify trigger/time/result present and no keys/path tokens/donor raw payloads.

---

## 6. E-03 — Points Engine

### TC-M031 — Valid EasyDonate webhook stored

**Requirement:** AC-020a → US-020 | **Priority:** 🔴 | **Type:** Integration

**Given** provider-shaped valid payload with `referenceNo`, donor, amount, channel, message, and time.
**When** webhook route receives it with valid path token.
**Then** donation is stored privately with `source=webhook` and pending status.

### TC-M032 — Duplicate reference is idempotent

**Requirement:** AC-020b → US-020 | **Priority:** 🔴 | **Type:** Integration

Send same `referenceNo` twice, including concurrent requests. Verify one donation and one possible point application only.

### TC-M033 — API fallback imports donations

**Requirement:** AC-020c → US-020 | **Priority:** 🔴 | **Type:** Integration

Mock EasyDonate API with Bearer credential and new records. Verify `source=api_poll` and no secret in logs.

### TC-M034 — EasyDonate 429 backoff

**Requirement:** AC-020d → US-020 | **Priority:** 🔴 | **Type:** Integration

Return 429 + Retry-After. Verify backoff and no tight retry loop.

### TC-M035 — Empty EasyDonate sync

**Requirement:** AC-020e → US-020 | **Priority:** 🟡 | **Type:** Unit

Return empty list. Verify no inserts and successful cycle.

### TC-M036 — EasyDonate unavailable retry

**Requirement:** AC-020f → US-020 | **Priority:** 🟡 | **Type:** Integration

Return provider/network failure. Verify safe log and next cycle retry.

### TC-M037 — Invalid webhook rejected

**Requirement:** AC-020g → US-020 | **Priority:** 🔴 | **Type:** Integration

Use invalid path token and invalid amount/missing reference. Verify rejection and no donation row.

### TC-M038 — Exact normalized match

**Requirement:** AC-021a → US-021 | **Priority:** 🔴 | **Type:** Unit

Active handle `deer123`, donor `@Deer123`. Verify match.

### TC-M039 — Harmless formatting differences

**Requirement:** AC-021b → US-021 | **Priority:** 🔴 | **Type:** Unit

Test whitespace/case/leading `@`; verify equal. Test punctuation/hyphen; verify not equal.

### TC-M040 — Inactive member not matched

**Requirement:** AC-021c → US-021 | **Priority:** 🔴 | **Type:** Integration

Inactive member owns matching old handle. Verify donation remains uncredited.

### TC-M041 — Before-registration cutoff

**Requirement:** AC-021d → US-021 | **Priority:** 🔴 | **Type:** Integration

Donation timestamp before registration. Verify `not_eligible`, zero point increment.

### TC-M042 — No active match remains private/uncredited

**Requirement:** AC-021e → US-021 | **Priority:** 🟡 | **Type:** Integration

Unknown donor name. Verify unmatched/private record and zero points.

### TC-M043 — Raw donor data excluded publicly

**Requirement:** AC-021f → US-021 | **Priority:** 🔴 | **Type:** System

Seed real-looking synthetic donor name/message; inspect scoreboard and public API; verify neither appears.

### TC-M044 — Eligible donation applies once

**Requirement:** AC-021g → US-021 | **Priority:** 🔴 | **Type:** Integration

Eligible donation amount 100; process matcher twice/concurrently; verify member points +100 exactly once and donation marker set.

### TC-M045 — Public exact total API

**Requirement:** AC-022a → US-022 | **Priority:** 🔴 | **Type:** Integration

Active public member with eligible total 500 returns exact 500.

### TC-M046 — Active member no eligible donations

**Requirement:** AC-022b → US-022 | **Priority:** 🔴 | **Type:** Integration

Active member with no eligible donation returns 0.

### TC-M047 — Point query by actual user ID

**Requirement:** AC-022c → US-022 | **Priority:** 🔴 | **Type:** Integration

Call user-ID route and verify active member response.

### TC-M048 — No active member response

**Requirement:** AC-022d → US-022 | **Priority:** 🔴 | **Type:** Integration

Inactive-only or unknown user ID returns `MEMBER_NOT_REGISTERED`.

### TC-M049 — Private member only exposes band

**Requirement:** AC-022e → US-022 | **Priority:** 🔴 | **Type:** Integration

Private 563 response contains lower/upper only and not `total_points`.

### TC-M050 — Visibility affects response without points mutation

**Requirement:** AC-022f → US-022 | **Priority:** 🔴 | **Type:** Integration

Toggle visibility, query response changes exact/range while total remains 563.

---

## 7. E-04 — Scoreboard

### TC-M051 — Public contributors ranked

**Requirement:** AC-030a → US-030 | **Priority:** 🟡 | **Type:** System

Seed public active contributors; page shows normalized handles and descending points.

### TC-M052 — Private member hidden

**Requirement:** AC-030b → US-030 | **Priority:** 🟡 | **Type:** System

Private member has points; page/API excludes it.

### TC-M053 — Inactive member hidden

**Requirement:** AC-030c → US-030 | **Priority:** 🟡 | **Type:** System

Inactive member has points; page/API excludes it.

### TC-M054 — Zero-point empty state

**Requirement:** AC-030d → US-030 | **Priority:** 🟡 | **Type:** Manual

Only zero-point active public members; page shows `No contributors yet. Be the first!`.

### TC-M055 — Freshness after new donation

**Requirement:** AC-030e → US-030 | **Priority:** 🟡 | **Type:** E2E

Process donation and refresh page; verify update within target 60 seconds under test conditions.

### TC-M056 — Scoreboard backend unavailable

**Requirement:** AC-030f → US-030 | **Priority:** 🟡 | **Type:** Manual

Stop API; page shows `Scoreboard temporarily unavailable`.

### TC-M057 — Scoreboard API eligible projection

**Requirement:** AC-031a → US-031 | **Priority:** 🟡 | **Type:** Integration

Verify 200 response includes rank/handle/points sorted descending.

### TC-M058 — Empty scoreboard API

**Requirement:** AC-031b → US-031 | **Priority:** 🟡 | **Type:** Integration

No eligible contributors; verify 200 empty array.

### TC-M059 — Status/visibility/points filter

**Requirement:** AC-031c → US-031 | **Priority:** 🟡 | **Type:** Integration

Seed private, inactive, and zero-point rows; verify all excluded.

### TC-M060 — First pagination page

**Requirement:** AC-031d → US-031 | **Priority:** 🟡 | **Type:** Integration

Seed >50 eligible rows; request page 1 limit 50; verify rows 1–50 and metadata.

### TC-M061 — Second pagination page

**Requirement:** AC-031e → US-031 | **Priority:** 🟡 | **Type:** Integration

Request page 2 limit 50; verify rows 51–100 without duplicates.

### TC-M062 — Public response data allowlist

**Requirement:** AC-031f → US-031 | **Priority:** 🟡 | **Type:** Integration

Inspect JSON keys; allow only rank, normalized handle, total points, and documented pagination metadata; no user ID, display name, donor data, messages, private/inactive data.

---

## 8. Execution Record

| Test Case Range | Executed On | Passed | Failed | Blocked | Tester |
|-----------------|-------------|:------:|:------:|:-------:|--------|
| TC-M001–M062 | — | — | — | — | QA |

> Execution results must be filled from real test runs. Do not mark cases passed based on specification review alone.

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[013_acceptance_criteria]] | Source criteria |
| [[041_test_plan]] | Test strategy |
| [[045_coverage_report]] | Coverage summary |
| [[022_API_specification]] | API contract |
| [[023_database_schema_DDL]] | Database rules |

---

> **Template Standard:** Based on SWEBOK v4 and ISO/IEC/IEEE 29119
> **Usage:** Active Phase 1 execution set. Superseded subscriber tests remain historical and are not evidence of current coverage.
