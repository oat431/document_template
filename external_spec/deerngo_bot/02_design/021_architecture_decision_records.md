---
document_type: ADR (Architecture Decision Records)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
architect: "SA / Designer Persona"
classification: "Internal"
tags: [adr, architecture-decisions, rationale, swebok, sebok, iso-42010, vrm]
standard_ref:
  - SWEBOK v4 — Architecture
  - SEBoK v2 — System Architecture
  - ISO/IEC/IEEE 42010 — Architecture Description
parent_project: "Deerngo Bot — VRM"
---

# ADR (Architecture Decision Records)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | SA / Designer Persona |
| Solution Architect | SA / Designer Persona |
| Stakeholder | Deer_NGO (YouTube Creator) |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | SA | Initial draft — formalize DEC-001 → DEC-007 from PO handoff + resolve DEC-D01 → DEC-D05 |

---

## 1. Purpose

> Architecture Decision Records (ADRs) capture significant architectural decisions along with their context and consequences. Each ADR answers: "What did we decide, why, and what are the trade-offs?"
>
> This document formalizes 12 decisions: 7 inherited from the PO's stakeholder grill session (DEC-001 → DEC-007) and 5 design-level decisions (DEC-D01 → DEC-D05) resolved during the PO → Designer handoff.

---

## 2. ADR Index

| ADR | Title | Status | Date | Decision |
|-----|-------|--------|------|----------|
| ADR-001 | Go Backend Language | ✅ Accepted | 2026-07-29 | Use Go for the backend service |
| ADR-002 | PostgreSQL 18 Database | ✅ Accepted | 2026-07-29 | Use existing PostgreSQL 18 homelab instance |
| ADR-003 | React/Next.js Frontend | ✅ Accepted | 2026-07-29 | Use React with Next.js for public scoreboard |
| ADR-004 | Homelab Deployment | ✅ Accepted | 2026-07-29 | Go + Next.js on homelab Docker; streamer.bot on local Windows PC |
| ADR-005 | Hybrid Subscriber Capture | ✅ Accepted | 2026-07-29 | YouTube API polling (24/7) + streamer.bot (real-time during live) |
| ADR-006 | Fuzzy Name Matching | ✅ Accepted | 2026-07-29 | Match donation names to YouTube handles via fuzzy matching |
| ADR-007 | EasyDonate Webhook Primary | ✅ Accepted | 2026-07-29 | Webhook for donation events; REST API as fallback |
| ADR-008 | sqlx for Database Access | ✅ Accepted | 2026-07-29 | Use sqlx (extends database/sql) instead of GORM or raw sql |
| ADR-009 | Fiber Web Framework | ✅ Accepted | 2026-07-29 | Use Fiber (fasthttp-based) instead of Gin, Echo, or stdlib |
| ADR-010 | Tailwind CSS + DaisyUI | ✅ Accepted | 2026-07-29 | Use Tailwind CSS with DaisyUI component library |
| ADR-011 | Cloudflare Tunnel for Public Access | ✅ Accepted | 2026-07-29 | Expose scoreboard via existing Cloudflare Tunnel |
| ADR-012 | HMAC-SHA256 Webhook Verification | ✅ Accepted | 2026-07-29 | Verify EasyDonate webhooks with HMAC-SHA256 signatures |

---

## 3. ADR-001: Go Backend Language

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO |
| **Source** | DEC-001 (PO grill session) |

### Context

> The VRM system needs a backend service that handles API requests, scheduled polling (YouTube API, EasyDonate), database interaction, and name matching logic. The stakeholder is comfortable with Go and values lightweight, fast services.

### Decision

> Use Go as the backend language. Single binary deployment, no runtime dependency, excellent concurrency primitives for handling polling schedulers and webhook receivers concurrently.

### Consequences

**Positive:**
- Single binary — easy deployment on Windows, no dependency management at runtime
- Excellent goroutine-based concurrency for polling schedulers + webhook server
- Fast startup time — suitable for a service that may restart frequently on a dev machine
- Strong standard library for HTTP, JSON, SQL

**Negative:**
- More boilerplate than dynamic languages for simple CRUD
- Smaller ecosystem of "batteries included" frameworks compared to Node.js or Python
- Error handling verbosity (explicit error returns)

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| Node.js / TypeScript | Stakeholder prefers Go; runtime dependency management on Windows |
| Python | Slower runtime; stakeholder not comfortable with it |
| Rust | Overkill for this project; steep learning curve |

---

## 4. ADR-002: PostgreSQL 18 Database

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO |
| **Source** | DEC-002 (PO grill session) |

### Context

> The system needs a relational database to store subscribers, donations, and viewer points. The stakeholder has PostgreSQL 18 running on their homelab, shared with other projects (Panomete platform).

### Decision

> Use the existing PostgreSQL 18 instance. Create a dedicated `deerngo` database. Enable the `pg_trgm` extension for fuzzy name matching.

### Consequences

**Positive:**
- No additional infrastructure cost or setup
- Proven, reliable RDBMS with excellent JSON support
- `pg_trgm` extension enables efficient fuzzy matching directly in SQL
- Shared infrastructure — backups already in place

**Negative:**
- Shared instance — resource contention with other databases under heavy load
- No isolation from other projects (homelab, not enterprise-grade)
- Must coordinate schema migrations with homelab maintenance windows

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| SQLite | No network access; single-file limitation; concurrent write issues |
| MySQL | Weaker JSON support; no `pg_trgm` equivalent |
| Dedicated PostgreSQL container | Unnecessary overhead — existing instance is sufficient |

---

## 5. ADR-003: React/Next.js Frontend

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO |
| **Source** | DEC-003 (PO grill session) |

### Context

> The public scoreboard needs a web frontend that renders a ranked list of viewers by points. It must be mobile-friendly, load fast (<2s), and be accessible to anyone on the internet.

### Decision

> Use React with Next.js (App Router). Server-side rendering for fast initial load. Static generation where possible — the scoreboard data changes frequently but the page structure is static.

### Consequences

**Positive:**
- Fast page loads via SSR/SSG — meets the <2s target
- Excellent mobile responsiveness with Tailwind/DaisyUI
- Large ecosystem — easy to find components and solutions
- SEO-friendly (public page, discoverable)

**Negative:**
- Node.js runtime requirement on the deployment machine
- Heavier than a plain HTML/JS page for a simple scoreboard
- Next.js framework overhead for what is essentially a single-page read-only app

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| Plain HTML + Vanilla JS | Too low-level; no component reuse; harder to maintain |
| Vue.js / Nuxt | Stakeholder more familiar with React ecosystem |
| Svelte / SvelteKit | Smaller ecosystem; less community support |

---

## 6. ADR-004: Homelab Deployment with streamer.bot on Local PC

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO, SA |
| **Source** | DEC-004 (PO grill session), corrected after infrastructure review |

### Context

> streamer.bot runs on the stakeholder's local Windows PC (it's a Windows desktop app). The Go backend, Next.js frontend, and PostgreSQL all run on the homelab server (Ubuntu 26.04, Docker). The Go backend needs to receive events from streamer.bot over LAN. The scoreboard needs to be accessible publicly.

### Decision

> Deploy the Go backend and Next.js frontend as Docker containers on the homelab server, connected to the existing `db-network` Docker network. Cloudflare Tunnel (running as systemd service on the homelab) exposes the scoreboard publicly. streamer.bot on the local Windows PC communicates with the Go backend over LAN (`192.168.1.121:8008`).

### Consequences

**Positive:**
- Go backend and PostgreSQL on the same Docker network (`db-network`) — fast, reliable DB access
- Cloudflare Tunnel already running as systemd service — just add a public hostname
- Homelab has Docker Compose — easy to add deerngo-bot to the stack
- No cloud hosting cost
- `deerngo` database already exists in PostgreSQL

**Negative:**
- streamer.bot → Go backend is a LAN call (~1ms), not localhost — requires stable LAN IP
- Homelab firewall must allow port 8080 from the Windows PC
- Single point of failure — homelab down = backend + scoreboard down
- No CI/CD pipeline for Phase 1

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| All on Windows PC | Windows is for streamer.bot only; homelab already has Docker + PostgreSQL |
| Cloud VPS (AWS/GCP/Azure) | Ongoing cost; DB already on homelab |
| Go backend on Windows, DB on homelab | Split deployment complexity; Docker Compose on homelab is simpler |

---

## 7. ADR-005: Hybrid Subscriber Capture

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO |
| **Source** | DEC-005 (PO grill session) |

### Context

> Subscribers can join at any time — during live streams and outside of them. streamer.bot only captures events during live streams. YouTube Data API can poll 24/7 but has quota limits (10,000 units/day). Neither source alone provides complete coverage.

### Decision

> Use a hybrid approach: YouTube Data API polling every 15 minutes for 24/7 coverage (primary), streamer.bot real-time push during live streams (secondary/backup). Upsert logic deduplicates by `youtube_handle`, preserving the earliest subscription timestamp.

### Consequences

**Positive:**
- 100% subscriber capture coverage — no gaps between live streams
- Real-time capture during live streams (streamer.bot) — <5s latency
- API polling catches everyone else — 15min max delay
- Upsert deduplication — no data conflicts

**Negative:**
- Two code paths to maintain (API poller + webhook receiver)
- Upsert logic must handle timestamp conflicts correctly
- YouTube API quota must be monitored (96 calls/day at 15-min intervals is well within 10K limit, but must not accidentally poll more frequently)

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| YouTube API only | 15-min delay during live streams; viewers expect real-time recognition |
| streamer.bot only | Misses subscribers who join outside live streams |
| Webhooks (YouTube PubSubHubbub) | Complex setup; requires publicly accessible endpoint with verification |

---

## 8. ADR-006: Fuzzy Name Matching

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO |
| **Source** | DEC-006 (PO grill session) |

### Context

> Donation names from EasyDonate may not exactly match YouTube subscriber handles. For example, a donation from "deer123" should match subscriber "@deer123". The system needs to handle case differences, leading/trailing whitespace, and the `@` prefix.

### Decision

> Use PostgreSQL `pg_trgm` extension for fuzzy matching at the database level. Normalize handles by stripping `@`, lowercasing, and trimming whitespace before comparison. Use `similarity()` function with a configurable threshold (default 0.7). Flag low-confidence matches and anonymous donations as "unmatched" for manual review.

### Consequences

**Positive:**
- Matching happens in SQL — no data transfer to Go for comparison
- `pg_trgm` is battle-tested and fast with GIN indexes
- Configurable threshold — can tune sensitivity
- Manual review queue for edge cases

**Negative:**
- `pg_trgm` extension must be enabled on the PostgreSQL instance
- False positives possible with very short or common names
- Manual review adds operational overhead for unmatched donations

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| Levenshtein distance in Go | Requires fetching all subscriber handles for comparison; doesn't scale as well |
| Exact match only | Too restrictive — donation names rarely match YouTube handles exactly |
| ML-based matching | Overkill for this use case; no training data |

---

## 9. ADR-007: EasyDonate Webhook Primary

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | Stakeholder (Deer_NGO), PO |
| **Source** | DEC-007 (PO grill session) |

### Context

> Donation events need to flow from EasyDonate to the Go backend. EasyDonate supports both webhooks (push) and REST API (pull). The webhook provides real-time delivery; the REST API can serve as a fallback for missed events.

### Decision

> Use EasyDonate webhook as the primary donation event source. Configure a webhook endpoint on the Go backend. Use the EasyDonate REST API as a fallback — poll periodically (e.g., every 5 minutes) to catch any webhook misses. Use `easydonate_id` as the idempotency key.

### Consequences

**Positive:**
- Real-time donation processing — points update within seconds
- Webhook reduces API polling load (stays well under 60 req/min limit)
- Fallback polling ensures no donation is missed even if webhook fails
- Idempotent sync prevents duplicate processing

**Negative:**
- Webhook endpoint must be publicly accessible (via Cloudflare Tunnel)
- Webhook delivery is not guaranteed — must handle failures gracefully
- Two ingestion paths to maintain and test

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| REST API polling only | Up to 5-min delay; higher API usage; rate limit risk |
| Webhook only | No fallback if webhook delivery fails silently |
| WebSocket | EasyDonate doesn't support WebSocket for donation events |

---

## 10. ADR-008: sqlx for Database Access

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | SA / Designer Persona |
| **Source** | DEC-D01 (PO → Designer handoff) |

### Context

> The Go backend needs to interact with PostgreSQL for CRUD operations, upsert logic, and fuzzy matching queries. The team needs a database access layer that is lean, type-safe, and doesn't add heavy dependencies.

### Decision

> Use `sqlx` (github.com/jmoiron/sqlx) — a library that extends Go's standard `database/sql` with struct scanning, named parameters, and bulk operations. Write SQL queries directly (no ORM magic, no code generation).

### Consequences

**Positive:**
- Thin layer over `database/sql` — minimal abstraction, full SQL control
- Struct scanning maps rows to Go structs automatically
- Named parameters (`:handle`) improve query readability
- No code generation step — faster iteration
- Easy to debug — SQL is visible in code, not hidden behind ORM

**Negative:**
- No automatic migrations (must use a separate tool like `golang-migrate`)
- No compile-time query validation (unlike sqlc)
- Manual struct tag management for column mapping

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| GORM | Heavy abstraction; magic behavior; harder to debug complex queries |
| sqlc | Code generation step adds build complexity; overkill for this project size |
| Raw `database/sql` | Too much boilerplate for struct scanning and named parameters |

---

## 11. ADR-009: Fiber Web Framework

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | SA / Designer Persona |
| **Source** | DEC-D02 (PO → Designer handoff) |

### Context

> The Go backend needs an HTTP framework for routing, middleware, request/response handling, and webhook endpoint serving. The API is simple (4 endpoints + webhook) but needs to be performant for real-time chat command responses (<2s latency).

### Decision

> Use Fiber (github.com/gofiber/fiber/v3) — a web framework built on fasthttp. Express-inspired API with built-in middleware for CORS, rate limiting, and request logging.

### Consequences

**Positive:**
- Fasthttp-based — significantly faster than net/http for high-throughput scenarios
- Express-like API — familiar if the team has Node.js experience
- Built-in middleware ecosystem (CORS, limiter, logger, compress)
- Low memory allocation — suitable for running on a shared machine

**Negative:**
- Not compatible with `net/http` handlers (fasthttp uses its own context)
- Some `net/http` middleware doesn't work directly (need Fiber-specific versions)
- fasthttp has subtle behavior differences from net/http (e.g., request body reuse)

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| Gin | Similar to Fiber but built on net/http; slightly slower; more popular |
| Echo | Similar to Gin; less active development recently |
| Chi | Stdlib-compatible but fewer built-in features; more manual setup |
| net/http (stdlib) | No built-in middleware; routing requires Go 1.22+; more boilerplate |

---

## 12. ADR-010: Tailwind CSS + DaisyUI

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | SA / Designer Persona |
| **Source** | DEC-D03 (PO → Designer handoff) |

### Context

> The public scoreboard needs a clean, responsive UI. It's a single-page read-only app showing a ranked list of viewers. The UI framework should be lightweight, mobile-friendly, and easy to maintain without a dedicated designer.

### Decision

> Use Tailwind CSS for utility-first styling with DaisyUI (daisyui.com) as a component library. DaisyUI provides pre-built components (tables, cards, badges) on top of Tailwind without adding JavaScript dependencies.

### Consequences

**Positive:**
- Utility-first CSS — fast prototyping, no custom CSS files to maintain
- DaisyUI components — tables, badges, loading states out of the box
- No JavaScript overhead — pure CSS components
- Excellent responsive design with Tailwind breakpoints
- Small bundle size — only used classes are included (tree-shaking)

**Negative:**
- Class-heavy HTML — can look cluttered in JSX
- DaisyUI theme customization requires Tailwind config changes
- Learning curve for developers new to utility-first CSS

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| shadcn/ui | Requires React + Radix; heavier for a simple scoreboard |
| Material UI | Heavy bundle; overkill for a read-only list |
| Plain CSS / CSS Modules | More manual work; no pre-built components |
| Bootstrap | jQuery dependency; less modern; harder to customize |

---

## 13. ADR-011: Cloudflare Tunnel for Public Access

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | SA / Designer Persona |
| **Source** | DEC-D04 (PO → Designer handoff) |

### Context

> The scoreboard must be accessible via the internet, but the Next.js frontend runs on the homelab server behind a NAT/firewall. The homelab already has Cloudflare Tunnel running as a systemd service (`cloudflared.service`).

### Decision

> Use the existing Cloudflare Tunnel on the homelab to expose the scoreboard. Configure a public hostname (e.g., `deerngo-viewer-score.panomete.com` or a subdomain) that tunnels to the local Next.js container at `localhost:3008`. The Go API is accessible only via Docker network (Next.js → Go) and LAN (streamer.bot → Go).

### Consequences

**Positive:**
- No port forwarding or firewall configuration needed
- Cloudflare handles TLS termination, DDoS protection, caching
- Already running — zero additional setup
- Fast global CDN for static assets

**Negative:**
- Dependency on Cloudflare service availability
- Scoreboard goes down if the local machine or internet connection drops
- Tunnel configuration is outside the Go backend's control

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| ngrok | Free tier has random URLs; paid tier unnecessary when Cloudflare Tunnel exists |
| Tailscale Funnel | Less familiar; additional dependency |
| Port forwarding | Security risk; ISP may block ports; dynamic IP issues |

---

## 14. ADR-012: HMAC-SHA256 Webhook Verification

| Field | Detail |
|-------|--------|
| **Status** | ✅ Accepted |
| **Date** | 2026-07-29 |
| **Decision Makers** | SA / Designer Persona |
| **Source** | DEC-D05 (PO → Designer handoff) |

### Context

> The Go backend exposes a webhook endpoint for EasyDonate donation events. Without verification, anyone could POST fake donation events to the endpoint, corrupting the points system.

### Decision

> Verify EasyDonate webhook payloads using HMAC-SHA256 signatures. EasyDonate signs the request body with a shared secret; the Go backend recomputes the signature and compares. Reject requests with invalid or missing signatures.

### Consequences

**Positive:**
- Cryptographic verification — cannot be spoofed without the secret
- Standard approach — well-documented, easy to implement
- Minimal performance overhead
- Secret stored as environment variable — not in code

**Negative:**
- Depends on EasyDonate supporting HMAC-SHA256 (must verify against their docs)
- Secret must be kept secure — if leaked, verification is bypassed
- Clock skew not an issue (HMAC doesn't depend on timestamps, but should add timestamp check for replay protection)

### Alternatives Considered

| Alternative | Why Not |
|------------|---------|
| Shared secret header | Less secure — secret sent in plaintext header; vulnerable to interception |
| IP whitelist | EasyDonate IPs may change; brittle; doesn't verify payload integrity |
| No verification | Unacceptable — fake donations would corrupt the points system |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[011_business_objective]] | Business objectives driving these decisions |
| [[012_user_stories]] | User stories that constrain architecture |
| [[013_acceptance_criteria]] | ACs that verify architectural properties |
| [[025_software_architecture_document]] | Architecture that these ADRs define |
| [[029_architecture_overview]] | High-level overview of the architecture |

---

> **Template Standard:** Based on SWEBOK v4, SEBoK v2, ISO/IEC/IEEE 42010
> **Usage:** ADRs capture *why* — the most valuable architectural documentation. Code shows *what*, docs show *how*, ADRs show *why*. Future team members will thank you.
