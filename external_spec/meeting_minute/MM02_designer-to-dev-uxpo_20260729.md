---
document_type: Meeting Minutes
version: "1.0"
status: Final
author: "SA / Designer Persona"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
meeting_type: "Designer → Dev / UX-UI / PO Handoff"
participants: ["SA / Designer Persona", "PO Persona", "Dev Persona", "UX/UI Persona"]
classification: "External"
tags: [meeting-minutes, handoff, deerngo-bot, phase-1, designer, dev, ux-ui, po]
---

# Meeting Minutes — Designer → Dev / UX-UI / PO Handoff: Deerngo Bot Phase 1

> **Date:** 2026-07-29
> **Type:** Design Handoff (Multi-Persona)
> **From:** SA / Designer Persona
> **To:** Dev Persona, UX/UI Persona, PO Persona
> **Status:** ✅ Design phase complete. Ready for implementation + wireframes + requirements review.

---

## 1. Purpose

> Record completion of Phase 1 design for Deerngo Bot VRM and hand off to three personas simultaneously: Dev (implementation), UX/UI (wireframes + style guide), and PO (requirements reconciliation). The Designer has produced 6 design documents — now each persona needs to pick up their deliverables.

---

## 2. What Designer Produced

### Design Documents (Phase 1)

| Document | Path | Status | Key Content |
|----------|------|--------|-------------|
| Architecture Decision Records | `02_design/021_architecture_decision_records.md` | ✅ v0.1 Draft | 12 ADRs — 7 from PO grill + 5 design decisions |
| API Specification | `02_design/022_API_specification.md` | ✅ v0.1 Draft | 4 REST endpoints + 1 webhook + 3 internal schedulers |
| Database Schema (DDL) | `02_design/023_database_schema_DDL.md` | ✅ v0.1 Draft | 4 tables, pg_trgm indexes, auto-sync trigger |
| ERD | `02_design/024_ERD.md` | ✅ v0.1 Draft | Mermaid ERD + entity definitions + data flow |
| Software Architecture Document | `02_design/025_software_architecture_document.md` | ✅ v0.1 Draft | Modular monolith, component design, Docker Compose, Go project structure |
| Architecture Overview | `02_design/029_architecture_overview.md` | ✅ v0.1 Draft | One-page map — component map, port table, sequence diagrams |

### Design Decisions Resolved

| ID | Question | Decision | Rationale |
|----|----------|----------|-----------|
| DEC-D01 | Go database layer | **sqlx** | Lean, type-safe, full SQL control |
| DEC-D02 | Go web framework | **Fiber v3** | Fast (fasthttp), Express-like API, built-in middleware |
| DEC-D03 | Scoreboard UI | **Tailwind CSS + DaisyUI** | Utility-first + pre-built components, no JS overhead |
| DEC-D04 | Public access | **Cloudflare Tunnel** | Already running as systemd on homelab |
| DEC-D05 | Webhook verification | **HMAC-SHA256** | Standard, cryptographic, cannot be spoofed |

### Infrastructure Discovery

| Component | Location | Details |
|-----------|----------|---------|
| Go Backend | Homelab (Docker, `db-network`) | Port **8008**, joins existing Docker network |
| Next.js Frontend | Homelab (Docker, `db-network`) | Port **3008**, exposed via Cloudflare Tunnel |
| PostgreSQL 18 | Homelab (Docker, `db-network`) | `local-postgres` container, `deerngo` database already created |
| Cloudflare Tunnel | Homelab (systemd) | `cloudflared.service`, hostname: `deerngo-viewer-score.panomete.com` |
| streamer.bot | Local Windows PC | Connects to Go backend at `192.168.1.121:8008` over LAN |

---

## 3. Design Changes That Affect Requirements

> These are design-level decisions that may require PO to update acceptance criteria.

### 3.1 Scoreboard Excludes 0-Point Viewers

| Field | Detail |
|-------|--------|
| Decision | Scoreboard (`GET /api/v1/scoreboard`) only returns viewers with `total_points > 0` |
| Rationale | Keeps the scoreboard meaningful — only real contributors shown |
| Impact on ACs | AC-031b updated: "empty array" now means "no viewers have points OR all have 0". New AC-031c added: "5 subscribers exist, but only 3 have donated → only 3 shown" |
| Impact on Stories | US-030 and US-031 — no story changes needed, AC update sufficient |
| `:deer: point` | Unaffected — still returns `total_points: 0` for any viewer |

### 3.2 OAuth Tokens Table Simplified

| Field | Detail |
|-------|--------|
| Decision | `oauth_tokens` table stores only the YouTube refresh token (not access_token, expires_at, scopes) |
| Rationale | Access tokens are short-lived (~1hr) and regenerated at runtime from refresh token |
| Impact | None on user stories — internal implementation detail |
| Future-proofing | `youtube_channel_id` column added for multi-channel support |

### 3.3 Deployment Topology Corrected

| Field | Detail |
|-------|--------|
| Original PO Assumption | Go backend + frontend on local Windows PC (same as streamer.bot) |
| Actual | Go backend + frontend on homelab server (Docker). Only streamer.bot on Windows PC. |
| Impact | streamer.bot → Go backend is a LAN call (`192.168.1.121:8008`), not localhost. No user story impact — internal infrastructure detail. |

---

## 4. Handoff to Dev Persona

### 🔴 Must Have (Implementation Tasks)

| # | Task | Depends On | Reference |
|---|------|-----------|-----------|
| 1 | Set up Go project structure | — | SAD §9 (project structure) |
| 2 | Implement `POST /api/v1/subscribers` | DB Schema, API Spec | API Spec §4.1 |
| 3 | Implement `GET /api/v1/points/{handle}` | DB Schema, API Spec | API Spec §4.2 |
| 4 | Implement `GET /api/v1/scoreboard` | DB Schema, API Spec | API Spec §4.3 (exclude 0-point viewers) |
| 5 | Implement `POST /api/v1/webhooks/easydonate` | DB Schema, API Spec | API Spec §4.4 (HMAC-SHA256 verification) |
| 6 | Implement YouTube API Polling Scheduler | OAuth table, API Spec | API Spec §4.6, SAD §3.6 |
| 7 | Implement EasyDonate Sync Scheduler | API Spec | API Spec §4.5, SAD §3.7 |
| 8 | Implement Name Matching Engine | DB Schema (pg_trgm) | API Spec §4.7, SAD §3.5 |
| 9 | Docker Compose for homelab | SAD | SAD §6.3 |
| 10 | Database migrations (golang-migrate) | DB Schema | DB Schema §6 |

### Dev Receives

| Document | What They Use It For |
|----------|---------------------|
| [[021_architecture_decision_records]] | Why each technology was chosen |
| [[022_API_specification]] | Exact endpoint contracts to implement |
| [[023_database_schema_DDL]] | DDL to run, table structures, triggers |
| [[024_ERD]] | Data model relationships |
| [[025_software_architecture_document]] | Component layering, Go project structure, Docker Compose |
| [[029_architecture_overview]] | Quick reference — ports, data flow, tech stack |

---

## 5. Handoff to UX/UI Persona

### 🟡 Should Have (Design Tasks)

| # | Task | Priority | Depends On | Reference |
|---|------|----------|-----------|-----------|
| 1 | Wireframes for Scoreboard page | 🟡 | — | Template: `02_design/026_wireframes_lofi.md` |
| 2 | Style Guide (colors, typography, components) | 🟡 | — | Template: `02_design/028_style_guide.md` |

### Design Constraints (UX/UI Must Respect)

| Constraint | Detail |
|-----------|--------|
| CSS Framework | Tailwind CSS + DaisyUI (already decided — ADR-010) |
| Responsive | Must work on mobile (AC-030c) |
| Scoreboard Data | Rank, display_name, youtube_handle, total_points, donation_count |
| Empty State | "No contributors yet. Be the first!" (AC-030e) |
| Error State | "Scoreboard temporarily unavailable" (AC-030d) |
| 0-Point Viewers | Not shown on scoreboard |
| Public Access | No auth — anyone can view |

### UX/UI Receives

| Document | What They Use It For |
|----------|---------------------|
| [[029_architecture_overview]] | Understanding the system at a glance |
| [[022_API_specification]] | Scoreboard response structure (§4.3) |
| [[013_acceptance_criteria]] | AC-030a → AC-030e (scoreboard page criteria) |
| [[012_user_stories]] | US-030 (Public Scoreboard Page) |

---

## 6. Handoff to PO Persona

### 🟡 Review Tasks

| # | Task | Priority | Action |
|---|------|----------|--------|
| 1 | Review 5 new design decisions (DEC-D01 → DEC-D05) | 🟡 | Confirm sqlx, Fiber v3, DaisyUI, Cloudflare Tunnel, HMAC-SHA256 align with vision |
| 2 | Review scoreboard 0-point exclusion | 🟡 | Confirm this is the desired behavior |
| 3 | Review deployment topology change | 🟡 | Confirm homelab Docker deployment is correct |
| 4 | Update requirements if needed | 🟡 | Bump version on affected docs if changes made |

### PO Receives

| Document | What They Use It For |
|----------|---------------------|
| [[021_architecture_decision_records]] | Reviewing design decisions (especially DEC-D01 → DEC-D05) |
| [[029_architecture_overview]] | High-level understanding of the designed system |
| This Meeting Minute | Understanding what changed from their original requirements |

### PO Requirements Reconciliation

| PO Document | Change Needed | Severity | Details |
|------------|---------------|----------|---------|
| `013_acceptance_criteria.md` | ✅ Already updated | 🟡 | AC-031a → AC-031e updated for 0-point exclusion |
| `012_user_stories.md` | No change needed | — | Stories are still valid |
| `011_business_objective.md` | No change needed | — | Objectives are still valid |

---

## 7. Action Items

| Action ID | Action | Owner | Priority | Depends On |
|-----------|--------|:-----:|:--------:|:----------:|
| DEV-001 | Review all design documents (021–025, 029) | Dev | 🔴 | — |
| DEV-002 | Set up Go project + Docker Compose | Dev | 🔴 | DEV-001 |
| DEV-003 | Run database migrations (023 DDL) | Dev | 🔴 | DEV-002 |
| DEV-004 | Implement Sprint 1 stories (US-001, US-002, US-003, US-010) | Dev | 🔴 | DEV-003 |
| DEV-005 | Implement Sprint 2 stories (US-011, US-012, US-020, US-021) | Dev | 🔴 | DEV-004 |
| DEV-006 | Implement Sprint 3 stories (US-022, US-030, US-031) | Dev | 🔴 | DEV-005 |
| UX-001 | Review API Spec §4.3 (scoreboard response) | UX/UI | 🟡 | — |
| UX-002 | Produce Scoreboard Wireframes (026) | UX/UI | 🟡 | UX-001 |
| UX-003 | Produce Style Guide (028) | UX/UI | 🟡 | — |
| PO-001 | Review design decisions (DEC-D01 → DEC-D05) | PO | 🟡 | — |
| PO-002 | Confirm 0-point viewer exclusion behavior | PO | 🟡 | — |
| PO-003 | Confirm deployment topology (homelab Docker) | PO | 🟡 | — |

---

## 8. Documents Produced This Session

| Document | Path | Purpose |
|----------|------|---------|
| ADR | `02_design/021_architecture_decision_records.md` | 12 architecture decisions |
| API Specification | `02_design/022_API_specification.md` | REST endpoint contracts |
| Database Schema | `02_design/023_database_schema_DDL.md` | PostgreSQL DDL + triggers |
| ERD | `02_design/024_ERD.md` | Entity-relationship model |
| SAD | `02_design/025_software_architecture_document.md` | Software architecture + project structure |
| Architecture Overview | `02_design/029_architecture_overview.md` | One-page architecture map |
| This Meeting Minute | `meeting_minute/MM02_designer-to-dev-uxpo_20260729.md` | Multi-persona handoff |

---

## 9. Sprint Plan (Confirmed)

| Sprint | Stories | Focus | Owner |
|--------|---------|-------|:-----:|
| Sprint 1 | US-001, US-002, US-003, US-010 | Subscriber registration (hybrid) + Donate command | Dev |
| Sprint 2 | US-011, US-012, US-020, US-021 | Point command + EasyDonate sync + Name matching | Dev |
| Sprint 3 | US-022, US-030, US-031 | Point query API + Scoreboard | Dev + UX/UI |

---

## 10. Handoff Summary

### What Each Persona Gets

| Persona | Documents | Key Deliverable |
|---------|-----------|----------------|
| **Dev** | 021, 022, 023, 024, 025, 029 | Implement the Go backend + Next.js frontend |
| **UX/UI** | 029, 022 (§4.3), 013, 012 | Wireframes (026) + Style Guide (028) |
| **PO** | 021, 029, this MM | Review design decisions, confirm requirements alignment |

### What's Next

1. **Dev** starts Sprint 1 implementation after reviewing design docs
2. **UX/UI** produces wireframes and style guide in parallel
3. **PO** reviews design decisions and confirms requirements are still aligned
4. **QA** receives API Spec + Acceptance Criteria for test planning (handoff from PO)

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[011_business_objective]] | Business objectives driving design |
| [[012_user_stories]] | Stories to be implemented |
| [[013_acceptance_criteria]] | ACs to verify implementation |
| [[MM01_po-to-designer_20260729]] | Previous handoff (PO → Designer) |
| [[021_architecture_decision_records]] | Design decisions |
| [[025_software_architecture_document]] | Software architecture |

---

> **Status:** Design phase complete. Designer hands off to Dev (implementation), UX/UI (wireframes), and PO (review).
> **Cross-persona handoff:** Designer defines HOW to build it. Dev implements. UX/UI designs the UI. PO validates alignment.
