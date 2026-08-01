---
document_type: ADR (Architecture Decision Records)
version: "0.2"
status: Draft
author: "PO / SA / Dev"
created: "2026-07-29"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
architect: "SA / Dev"
classification: "Internal"
tags: [adr, architecture-decisions, members, points, privacy, easydonate, vrm]
standard_ref:
  - SWEBOK v4 — Architecture
  - SEBoK v2 — System Architecture
  - ISO/IEC/IEEE 42010 — Architecture Description
parent_project: "Deerngo Bot — VRM"
---

# Architecture Decision Records

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** The original subscriber-observation architecture was superseded after live YouTube API verification showed that only a limited subscriber subset is exposed. Phase 1 now uses explicit member registration through streamer.bot.

---

## 1. Decision Index

| ADR | Title | Status | Decision |
|-----|-------|--------|----------|
| ADR-001 | Go Backend | ✅ Accepted | Use Go for the backend |
| ADR-002 | PostgreSQL 18 | ✅ Accepted | Use the existing homelab PostgreSQL |
| ADR-003 | Next.js Frontend | ✅ Accepted | Use Next.js + React for the public scoreboard |
| ADR-004 | Homelab Deployment | ✅ Accepted | Backend/frontend in homelab Docker; streamer.bot on Windows |
| ADR-005 | Explicit Member Registration | ✅ Accepted | `:deer: register` creates members from actual streamer.bot identity |
| ADR-006 | Normalized Exact Matching | ✅ Accepted | Match donor name to active member handle after trim/@ removal/lowercase |
| ADR-007 | EasyDonate Ingestion | ✅ Accepted | Webhook primary + API polling fallback; provider auth must be verified |
| ADR-008 | sqlx | ✅ Accepted | Use sqlx for PostgreSQL access |
| ADR-009 | Fiber v3 | ✅ Accepted | Use Fiber v3 for HTTP API |
| ADR-010 | Tailwind + DaisyUI | ✅ Accepted | Use approved frontend design system |
| ADR-011 | Cloudflare Tunnel | ✅ Accepted | Use existing tunnel for public web/webhook routes |
| ADR-012 | HMAC Webhook Verification | ⚠️ Superseded | Do not assume HMAC until EasyDonate confirms support |
| ADR-013 | Stable Member Identity | ✅ Accepted | Store streamer.bot user ID; member_id is record primary key |
| ADR-014 | Re-registration | ✅ Accepted | New handle creates inactive old record + new active zero-point record |
| ADR-015 | Visibility and Point Bands | ✅ Accepted | Public/private commands; private members receive 100-point ranges in chat |
| ADR-016 | Public Data Minimization | ✅ Accepted | No display names; scoreboard exposes only current handle and points |

---

## 2. ADR-001: Go Backend

**Decision:** Use Go for the backend.

**Rationale:** Go is lightweight, familiar to the developer, and suitable for HTTP handlers, database access, donation ingestion, and concurrent background work.

**Consequences:** Single binary/container, explicit error handling, and a small operational footprint. The backend must still be modular so member, donation, and points behavior remain testable.

---

## 3. ADR-002: PostgreSQL 18

**Decision:** Use the existing PostgreSQL 18 homelab instance and dedicated `deerngo` database.

**Rationale:** Existing infrastructure, relational constraints, transactions, backups, and reliable concurrent writes.

**Consequences:** Migrations must be versioned. Manual corrections require a transaction, backup awareness, and a correction note.

---

## 4. ADR-003: Next.js Frontend

**Decision:** Use Next.js/React with Tailwind CSS and DaisyUI for the public scoreboard.

**Rationale:** The project already has the scaffold and design system; it supports responsive, read-only rendering.

**Consequences:** The frontend consumes the backend public scoreboard contract and must never implement its own points or matching logic.

---

## 5. ADR-004: Homelab Deployment

**Decision:** Deploy Go backend and Next.js frontend in homelab Docker on `db-network`. Keep streamer.bot on the streamer's Windows PC. Use LAN HTTP from streamer.bot to the backend.

**Rationale:** PostgreSQL and the public tunnel already exist in the homelab; streamer.bot is a Windows desktop application.

**Consequences:** The backend must be reachable from the Windows PC at the configured LAN address. The public tunnel must expose only approved web/webhook routes.

---

## 6. ADR-005: Explicit Member Registration

**Status:** Accepted; supersedes the former hybrid subscriber-capture decision.

**Decision:** Do not use YouTube subscriber polling or automatic subscriber events as the Phase 1 membership source. A viewer joins by typing `:deer: register` during live chat.

**Identity input:** streamer.bot supplies the actual chat author's user ID and current handle. A handle typed in the command text is ignored.

**Rationale:** Live testing showed the YouTube Data API returned 172 of 1,310 subscribers. Explicit registration is complete for the intended VRM membership event and avoids importing unnecessary identities/display names.

**Consequences:** Registration is available only while streamer.bot and the live chat integration are active. A viewer who registers starts at zero points. The old `subscribers` table, YouTube poller, and YouTube OAuth path are not active MVP components.

---

## 7. ADR-006: Normalized Exact Matching

**Decision:** Use normalized exact matching for Phase 1 member points.

```text
normalize(value) = trim whitespace → remove one leading @ → lowercase
```

A donation earns points only when the normalized EasyDonate donor name equals the normalized handle of exactly one active member and `donation_time >= registered_at`.

**Rationale:** Exact matching follows the channel owner's published rule and avoids fuzzy false positives that could assign money/points to the wrong person.

**Consequences:** Donors must use their registered YouTube handle as their EasyDonate name. Unmatched, inactive, and pre-registration donations remain uncredited.

> The former `pg_trgm` fuzzy-matching decision is superseded for the active MVP path.

---

## 8. ADR-007: EasyDonate Webhook Primary

**Decision:** Use EasyDonate webhook as the primary donation ingestion path and EasyDonate REST API polling as fallback reconciliation.

**Rationale:** Webhook provides low-latency events; polling can recover missed events.

**Provider contract rule:** Current public EasyDonate documentation confirms webhook URL configuration and payload examples, and documents API-key/Bearer authentication for the API. It does not confirm HMAC or an `X-EasyDonate-Signature` header. The implementation must verify the actual dashboard/provider contract before enabling provider-specific signature validation.

**MVP fallback protection if no provider signing exists:** unpredictable webhook path token, strict payload validation, request-size limit, rate limiting, and idempotency by `referenceNo`.

**Consequences:** The webhook path token is a secret and must be rotated if exposed. Raw donor names/messages stay private. The API key is backend-only and must have the provider's donation-read scope.

---

## 9. ADR-008: sqlx

**Decision:** Use sqlx over GORM for PostgreSQL access.

**Rationale:** Lean access layer, explicit SQL, parameterized queries, and good fit for a small relational schema.

**Consequences:** Repository queries and transactions are explicit and must be covered by integration tests.

---

## 10. ADR-009: Fiber v3

**Decision:** Use Fiber v3 for HTTP routing and middleware.

**Rationale:** Existing scaffold and suitable middleware for validation, CORS, rate limiting, and request-size controls.

---

## 11. ADR-010: Tailwind CSS + DaisyUI

**Decision:** Use the approved Tailwind/DaisyUI Deer_NGO theme.

**Consequences:** The public UI must implement loading, empty, error, mobile, and privacy-filtered scoreboard states.

---

## 12. ADR-011: Cloudflare Tunnel

**Decision:** Use the existing Cloudflare Tunnel for the public scoreboard and the configured EasyDonate webhook route.

**Consequences:** The backend's entire API must not be exposed publicly merely to receive donations. Route exposure must be intentionally limited.

---

## 13. ADR-012: HMAC Webhook Verification

**Status:** Superseded / unverified.

The original design assumed HMAC-SHA256 and `X-EasyDonate-Signature`. Current public EasyDonate documentation reviewed on 2026-08-02 did not confirm that contract. Do not implement or request `EASYDONATE_WEBHOOK_SECRET` unless the provider dashboard or official provider response confirms it.

If EasyDonate later confirms a signing scheme, create a follow-up ADR and update the API/security/test documents before enabling it.

---

## 14. ADR-013: Stable Member Identity

**Decision:** Store the stable streamer.bot/YouTube user ID on each member record, but use `member_id` as the row primary key.

**Rationale:** A user ID identifies the chat author supplied by streamer.bot, while `member_id` allows the MVP's simple re-registration behavior.

**Consequences:** One active record per user ID; old inactive records may remain for manual point correction. YouTube display name is not stored.

---

## 15. ADR-014: Re-registration and Handle Changes

**Decision:** Same-handle registration is idempotent. A changed handle creates a new active member with zero points and marks the old member inactive, preserving old points without automatic transfer.

**Rationale:** Avoid complex alias history/admin UI in Phase 1. The channel owner/back-office worker can manually correct points later.

**Consequences:** A handle is unique among active members. A conflict with another active user is rejected. Inactive records are hidden from matching, point queries, and scoreboard.

---

## 16. ADR-015: Visibility and Point Bands

**Decision:** New members default to `public_visibility=true`. `:deer: public` and `:deer: private` switch visibility without changing points.

- Public members with points >0 appear on the scoreboard and receive exact point responses.
- Private members continue earning points but are excluded from the scoreboard.
- Private point responses use 100-point bands: 563 → 500–600; 0 → 0–100.

**Rationale:** Preserve engagement while reducing exact public donation/score exposure.

**Consequence:** Chat responses are public; the banding rule limits precision but does not make the response private.

---

## 17. ADR-016: Public Data Minimization

**Decision:** Do not store YouTube display names in the member record. The public scoreboard returns only normalized current handle, rank, and total points for active public members with points >0.

**Rationale:** Minimize personal-data collection and avoid automatically publishing real-looking display names.

**Consequences:** The owner must provide a clear registration/public-display notice and a practical hide/remove path. Thai PDPA obligations should be reviewed by the channel owner with appropriate legal advice.

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[022_API_specification]] | Current API contracts |
| [[023_database_schema_DDL]] | Current physical data model |
| [[024_ERD]] | Current logical data model |
| [[025_software_architecture_document]] | Current software architecture |
| [[011_business_objective]] | Current business objectives |
| [[012_user_stories]] | Current user stories |
| [[013_acceptance_criteria]] | Current acceptance criteria |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Scope-change decision |

---

> **Template Standard:** Based on SWEBOK v4, SEBoK v2, ISO/IEC/IEEE 42010
> **Usage:** ADRs capture why decisions were made and identify superseded assumptions.
