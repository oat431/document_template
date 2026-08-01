---
document_type: Meeting Minutes
version: "1.0"
status: Final
author: "Dev Persona / PO Persona"
created: "2026-08-01"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
meeting_type: "Dev → PO/QA — YouTube API Subscriber Limit Finding"
participants: ["Dev Persona", "PO Persona", "QA Persona"]
classification: "Internal"
tags: [meeting-minutes, youtube-api, limitation, subscriber-capture, us-003, risk, deerngo-bot]
standard_ref:
  - SWEBOK v4 — Requirements
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 12207 — Software Life Cycle Processes
---

# Meeting Minutes — Dev → PO/QA: YouTube Data API Subscriber List Limitation

> **Date:** 2026-08-01
> **Type:** Technical finding notification
> **From:** Dev Persona
> **To:** PO Persona, QA Persona
> **Status:** ✅ PO decision complete — subscriber polling superseded by explicit member registration MVP

---

## 1. Purpose

> Notify PO and QA of a verified limitation in the YouTube Data API v3 that affects **US-003 (YouTube polling scheduler)** and **US-001 (hybrid subscriber capture)**. The finding was discovered and proven with live API calls during US-003 implementation verification. A decision is required on how to interpret the affected acceptance criteria before final acceptance of the sprint.

---

## 2. Finding Summary

| # | Finding | Severity | Evidence |
|---|---------|:--------:|----------|
| 1 | `GET /youtube/v3/subscriptions?mySubscribers=true` returns only **172 of 1,310** actual subscribers | 🔴 High | Live API verification, 2026-08-01 |
| 2 | The API spec §4.6 example URL (`channelId=...&order=date`) is **invalid**: `channelId` returns the channels the owner *follows*, and `order=date` returns `400 INVALID_ARGUMENT` | 🔴 High | Live API verification (PR #15 fix) |
| 3 | `subscriberSnippet` in the subscriptions response is **empty** — subscriber handle/display name requires a second `channels.list` enrichment call | 🟡 Medium | Live API response inspection |

### 2.1 Live Verification Evidence

Run against the real `@Deer_NGO` channel (OAuth token stored in `oauth_tokens`), 2026-08-01:

```
channels.list statistics:    subscriberCount=1310, hiddenSubscriberCount=False
subscriptions mySubscribers: pageInfo.totalResults=172
First poll (live):           fetched=172  created=172  upserted=0
Second poll (live):          fetched=172  created=0   upserted=172   (no duplicates)
```

The 172 captured subscribers are the **most recent** signups (2026-07-09 → 2026-07-31).

---

## 3. Impact Analysis

### 3.1 What Still Works ✅

- Polling every 15 minutes continues to capture **new** subscribers (they appear in the recent window).
- Upsert/dedup preserves earliest `subscribed_at` (verified: second poll created 0 duplicates).
- 403 `quotaExceeded` skip-cycle and 401 re-auth alert behaviors (AC-003e/f) are unaffected.
- streamer.bot live events (US-002/Issue #2, already merged) capture subscribers during live streams independently of the API.

### 3.2 What Is Limited ❌

- **Historical backfill is impossible via the Data API v3**: the API exposes only a subset (172) of the channel's 1,310 subscribers. There is no API endpoint that returns the full subscriber list for a channel.
- **Database will not match the public subscriber count.** Expect the `subscribers` table to grow from ~172 by new-signup capture only, until live events add more.

### 3.3 Affected Acceptance Criteria

| AC | Spec expectation | Reality |
|----|------------------|---------|
| AC-003a | Scheduler calls `GET /youtube/v3/subscriptions` every 15 min | ✅ Call happens (with corrected parameters, PR #15) |
| AC-003b | New API subscribers inserted with `source=youtube_api` | ✅ Works for the subset the API returns |
| AC-003c | Upsert without duplicates, earliest timestamp preserved | ✅ Verified live |
| AC-003d/e/f/g | Empty/403/401/interval behaviors | ✅ Verified |
| AC-001b (US-001) | "backend fetches new subscribers from YouTube Data API" | ⚠️ Fetches **available** subscribers; cannot fetch all 1,310 |

> **Key question for PO:** Is the acceptance interpretation "capture all subscribers the platform reports" (impossible via API) or "capture new subscribers going forward via the best available API mechanism" (achievable)?

---

## 4. Recommended Resolution (for PO decision)

### Option A — Accept the limitation (recommended)

- Interpret AC-003a/b and AC-001b as "capture subscribers the API exposes, going forward".
- Document the limitation in the issue and the runbook.
- The system captures: all new subscribers via 15-min polling + all live-stream subscribers via streamer.bot events.
- No further code change required.

### Option B — Hybrid supplementary capture

- Keep API polling + streamer.bot, and additionally consider periodic manual/community import (not API-based) if historical subscriber data matters for points.
- Requires PO to define an import mechanism and a data-ownership decision (e.g., "historical subscribers get 0 points unless they subscribe again").

### Option C — Re-scope US-003 acceptance

- PO formally amends AC-003a/b wording to reflect the platform limitation, and QA maps the test cases (TC-012/TC-013) to "mock returns N subscribers → N records" (which already passes), plus a live smoke test that documents the 172-vs-1310 gap.

---

## 5. PO Decision — Replace Subscriber Capture with Explicit Member Registration

### Decision

The YouTube subscriber polling approach is removed from the Phase 1 MVP. The `subscribers` table is removed from the active product model.

A viewer must explicitly join the VRM program by typing `:deer: register` during a live stream. streamer.bot supplies the real chat author's YouTube user ID and current handle to the Go backend.

### New MVP Model

```text
streamer.bot chat identity
    → :deer: register
    → members table
    → member starts at 0 points
    → future EasyDonate donations may earn points
```

| Rule | Approved Behavior |
|------|-------------------|
| YouTube polling | Removed from MVP; US-003 is superseded/cancelled |
| Automatic subscriber capture | Removed from MVP |
| Member registration | Explicit `:deer: register` only during live chat |
| Member identity | `member_id` primary key; store `youtube_user_id` and normalized handle |
| Display name | Do not store YouTube display name |
| First registration | Creates active member with 0 points and `public_visibility=true` |
| Same-handle re-registration | Friendly already-registered response; no DB change |
| New-handle re-registration | Old member becomes inactive; old points remain; new active member starts at 0 |
| Inactive members | Hidden from point query, matching, and scoreboard; retained for manual correction |
| Active handle conflict | Reject registration; owner resolves manually |
| Donation cutoff | Only `donation_time >= member.registered_at` is eligible |
| Donation matching | Normalized exact match: trim, remove leading `@`, lowercase |
| Old donations | Not credited automatically |
| Point storage | `members.total_points`; manual correction is direct DB work in MVP |
| Public scoreboard | Active + public + points greater than 0 only |
| Visibility | `:deer: public` / `:deer: private` |
| Private member points | Continue earning; public chat shows a 100-point band, not the exact total |
| Unregistered commands | Ask viewer to register; never auto-create |
| EasyDonate ingestion | Webhook primary, API polling fallback |
| Webhook security | Do not assume HMAC; use provider-confirmed mechanism. MVP fallback: unpredictable URL path, strict validation, `referenceNo` idempotency, body limit, rate limiting |

### Rationale

- The live API cannot provide the complete historical subscriber list: 172 of 1,310 were available in verification.
- Explicit registration is a clearer membership event than incomplete subscriber observation.
- The new flow reduces unnecessary collection of YouTube display names.
- Starting everyone at zero avoids retroactive donation attribution and historical matching disputes.
- The owner can correct exceptional point transfers directly until an admin console exists.

### Superseded Actions

| Action | Status | Reason |
|--------|--------|--------|
| MM06-A1 — Choose API limitation interpretation | ✅ Superseded | Polling no longer drives membership |
| MM06-A2 — Map polling tests | ❌ Cancelled | US-003 removed from MVP |
| MM06-A3 — Correct API spec §4.6 | 🟡 Deferred | YouTube polling is no longer active MVP scope |
| MM06-A4 — Document 172-vs-1310 gap | ✅ Done | Finding retained as rationale for the change |

### New Actions

| Action ID | Action | Owner | Due | Status |
|-----------|--------|:-----:|:---:|:------:|
| MM06-A5 | Replace subscriber requirements with member-registration requirements | PO | Before next Dev sprint | ⬜ Open |
| MM06-A6 | Replace subscriber schema/API with members schema/API | Dev/SA | Before implementation | ⬜ Open |
| MM06-A7 | Add `:deer: register`, `:deer: public`, and `:deer: private` streamer.bot actions | Dev | Before member-flow QA | ⬜ Open |
| MM06-A8 | Verify EasyDonate webhook payload/security from the dashboard; do not assume HMAC | Dev/DevOps | Before webhook deployment | ⬜ Open |
| MM06-A9 | Add privacy notice and public-display/removal behavior | PO/Dev | Before scoreboard release | ⬜ Open |

---

## 6. Original Technical Finding (Retained for Record)

The original finding remains valid: YouTube Data API v3 returned 172 of 1,310 subscribers, the original `channelId`/`order=date` example was invalid, and subscriber enrichment required a second API call. This finding is now the rationale for removing polling from the MVP, not an open acceptance decision.

---

## 7. Action Items

| Action ID | Action | Owner | Due | Status |
|-----------|--------|:-----:|:---:|:------:|
| MM06-A10 | Archive or close the superseded US-003 implementation path after documenting the merged code | Dev | Before next sprint | ⬜ Open |
| MM06-A11 | Preserve the YouTube API work as non-MVP code or remove it after PO/Dev review | Dev | Before next sprint | ⬜ Open |
| MM06-A12 | Update QA coverage and regression scope for the new member flow | QA | Before regression | ⬜ Open |

---

## 8. Related PRs / Commits

| Item | Reference | Note |
|------|-----------|------|
| PR #12 | US-003 initial implementation (merged) | Used spec URL; worked against mocks |
| PR #15 | `fix/issue-3-youtube-my-subscribers` (merged) | Corrected endpoint to `mySubscribers=true` + channels enrichment; live-verified. Superseded as active MVP path. |
| Live poll evidence | DB `subscribers` table, 172 rows, `source=youtube_api` | 2026-08-01 |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[013_acceptance_criteria]] | AC-003a–g, AC-001b affected by this finding |
| [[022_API_specification]] | §4.6 contains the invalid example URL |
| [[042_test_cases]] | TC-012–TC-018 map to US-003 |
| [[072_MM05_po-to-dev-development-workflow_20260731]] | Sprint workflow and issue rules |

---

> **Status:** ✅ PO decision complete. YouTube polling is removed from the active Phase 1 MVP and replaced by explicit member registration. The live API limitation remains documented for the record.
> **Meeting ID:** MM06
> **File:** `07_pm/072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801.md`
