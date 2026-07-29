---
document_type: Wireframes (Low-fi)
version: "1.0"
status: Draft
author: "UX/UI Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [wireframes, low-fidelity, ux, layout, scoreboard, deerngo-bot]
standard_ref:
  - ISO 9241-210 — Human-Centred Design
---

# Wireframes (Low-fidelity) — Deerngo Bot Scoreboard

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-30
>
> ⚠️ **Note:** Low-fi wireframes are *structural blueprints* — they show layout and hierarchy without visual design. Implemented with Tailwind CSS + DaisyUI. For pixel-perfect mockups, see Figma.

---

## 1. Purpose

> Define the structure, content placement, and interaction states for the Deerngo Bot public scoreboard page. This is the **only UI page in Phase 1** — a single-page public leaderboard displaying viewer contribution rankings.

---

## 2. Wireframe Index

| # | Screen | Device | Status |
|---|--------|--------|--------|
| WF-001 | Scoreboard — Happy Path (with data) | Desktop | Draft |
| WF-002 | Scoreboard — Happy Path (with data) | Mobile | Draft |
| WF-003 | Scoreboard — Empty State | Desktop + Mobile | Draft |
| WF-004 | Scoreboard — Error State | Desktop + Mobile | Draft |
| WF-005 | Scoreboard — Loading State | Desktop + Mobile | Draft |

---

## 3. Wireframe Specifications

### WF-001: Scoreboard — Happy Path (Desktop)

> Viewer visits the public URL. The scoreboard has 25 viewers with points. Top 10 shown by default, pagination for more.

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│                        🦌 DEERNGO BOT                                │
│                    Viewer Contribution Leaderboard                    │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  🥇 @topdonor          Top Donor              1,500 pts       │  │
│  │     ─────────────────────────────────────  10 donations       │  │
│  │  🥈 @viewer1           Viewer One              500 pts       │  │
│  │     ─────────────────────────────────────   3 donations       │  │
│  │  🥉 @deerfan           Deer Fan                 420 pts       │  │
│  │     ─────────────────────────────────────   5 donations       │  │
│  │                                                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ #   │ Display Name    │ YouTube Handle   │ Points        │ │  │
│  │  ├─────┼─────────────────┼──────────────────┼───────────────┤ │  │
│  │  │  4  │ Forest Friend   │ @forestfriend    │    350        │ │  │
│  │  │  5  │ Nature Lover    │ @naturelover     │    280        │ │  │
│  │  │  6  │ Stream Fan      │ @streamfan       │    200        │ │  │
│  │  │  7  │ Green Heart     │ @greenheart      │    150        │ │  │
│  │  │  8  │ Deer Supporter  │ @deersupporter   │    100        │ │  │
│  │  │  9  │ Eco Viewer      │ @ecoviewer       │     75        │ │  │
│  │  │ 10  │ Wild Watcher    │ @wildwatcher     │     50        │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  │                                                               │  │
│  │            [← Previous]  Page 1 of 3  [Next →]               │  │
│  │                                                               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│              🦌 Support @Deer_NGO — Donate to earn points!            │
│              https://easydonate.app/deerngo0                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Layout Specifications:**

| Element | Position | Size | Priority |
|---------|---------|------|---------|
| Page Header (Title) | Top center | Full width, ~120px | 🔴 Brand identity |
| Podium Section (Top 3) | Below header | Full width, cards row | 🔴 Hero content |
| Scoreboard Table | Below podium | Full width, scrollable | 🔴 Core content |
| Pagination | Below table | Centered, ~48px | 🟡 Navigation |
| Footer (CTA) | Bottom | Full width, ~60px | 🟡 Engagement |

**Interaction Notes:**
- Podium cards (top 3) — highlighted with visual distinction (gold/silver/bronze)
- Table rows — zebra striping for readability, hover highlight
- Pagination — DaisyUI pagination component, disabled at boundaries
- "Previous" disabled on page 1, "Next" disabled on last page
- Auto-refresh: page polls API every 60s, shows subtle refresh indicator

---

### WF-002: Scoreboard — Happy Path (Mobile)

```
┌───────────────────────────┐
│                           │
│      🦌 DEERNGO BOT       │
│  Viewer Leaderboard       │
│                           │
│ ┌───────────────────────┐ │
│ │                       │ │
│ │  🥇 @topdonor         │ │
│ │     Top Donor         │ │
│ │     1,500 pts         │ │
│ │     10 donations      │ │
│ │                       │ │
│ │  🥈 @viewer1          │ │
│ │     Viewer One        │ │
│ │     500 pts           │ │
│ │     3 donations       │ │
│ │                       │ │
│ │  🥉 @deerfan          │ │
│ │     Deer Fan          │ │
│ │     420 pts           │ │
│ │     5 donations       │ │
│ │                       │ │
│ └───────────────────────┘ │
│                           │
│ ┌───────────────────────┐ │
│ │ #4  Forest Friend     │ │
│ │     @forestfriend     │ │
│ │     350 pts           │ │
│ ├───────────────────────┤ │
│ │ #5  Nature Lover      │ │
│ │     @naturelover      │ │
│ │     280 pts           │ │
│ ├───────────────────────┤ │
│ │ #6  Stream Fan        │ │
│ │     @streamfan        │ │
│ │     200 pts           │ │
│ ├───────────────────────┤ │
│ │ #7  Green Heart       │ │
│ │     @greenheart       │ │
│ │     150 pts           │ │
│ └───────────────────────┘ │
│                           │
│  [← Prev]  Page 1/3  [→] │
│                           │
│  🦌 Donate to earn points!│
│  easydonate.app/deerngo0  │
│                           │
└───────────────────────────┘
```

**Mobile-Specific Layout:**

| Element | Adjustment |
|---------|-----------|
| Podium Cards | Stack vertically, one card per row |
| Scoreboard Table | Card-based list instead of table — each row is a card |
| Table Columns | Display Name + Points prioritized; handle shown smaller |
| Pagination | Compact — page X of Y with arrow buttons |
| Footer CTA | Single line, smaller font |
| Rank Badges | Smaller, still distinct for top 3 |

---

### WF-003: Scoreboard — Empty State

> No viewers have donated yet. All subscribers have 0 points.

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│                        🦌 DEERNGO BOT                                │
│                    Viewer Contribution Leaderboard                    │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │                         🦌                                    │  │
│  │                                                               │  │
│  │           No contributors yet. Be the first!                   │  │
│  │                                                               │  │
│  │       Support the stream by donating at the link below.        │  │
│  │                                                               │  │
│  │                 [Donate Now →]                                │  │
│  │                                                               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│              🦌 Support @Deer_NGO — Donate to earn points!            │
│              https://easydonate.app/deerngo0                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Empty State Specifications:**

| Element | Value |
|---------|-------|
| Icon | Large deer emoji (🦌), ~64px, centered |
| Message | "No contributors yet. Be the first!" (per AC-030e) |
| Sub-message | "Support the stream by donating at the link below." |
| CTA Button | "Donate Now →" linking to EasyDonate page |
| API State | data: [], meta.total: 0 |

**Mobile:** Same layout, smaller emoji and text, button full-width.

---

### WF-004: Scoreboard — Error State

> Backend API is unreachable or returns 5xx.

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│                        🦌 DEERNGO BOT                                │
│                    Viewer Contribution Leaderboard                    │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │                         ⚠️                                    │  │
│  │                                                               │  │
│  │           Scoreboard temporarily unavailable                   │  │
│  │                                                               │  │
│  │        We're having trouble loading the leaderboard.           │  │
│  │        Please check back in a few minutes.                    │  │
│  │                                                               │  │
│  │                    [Try Again]                                │  │
│  │                                                               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│              🦌 Support @Deer_NGO — Donate to earn points!            │
│              https://easydonate.app/deerngo0                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Error State Specifications:**

| Element | Value |
|---------|-------|
| Icon | Warning sign (⚠️), ~48px, centered, muted color |
| Primary Message | "Scoreboard temporarily unavailable" (per AC-030d) |
| Sub-message | "We're having trouble loading the leaderboard. Please check back in a few minutes." |
| Retry Button | "Try Again" — re-fetches the scoreboard API |
| API State | fetch() rejected or returned non-200 |
| Footer | Still visible (donation link always works) |

**Mobile:** Same layout, button full-width.

---

### WF-005: Scoreboard — Loading State

> Initial page load or refresh. API call in progress.

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│                        🦌 DEERNGO BOT                                │
│                    Viewer Contribution Leaderboard                    │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │                                                               │  │
│  │              ┌─────────────────────────┐                      │  │
│  │              │ ████████░░░░░░░░░░░░░░ │  (skeleton row)      │  │
│  │              └─────────────────────────┘                      │  │
│  │              ┌─────────────────────────┐                      │  │
│  │              │ ██████████░░░░░░░░░░░░░ │  (skeleton row)      │  │
│  │              └─────────────────────────┘                      │  │
│  │              ┌─────────────────────────┐                      │  │
│  │              │ ████████░░░░░░░░░░░░░░  │  (skeleton row)      │  │
│  │              └─────────────────────────┘                      │  │
│  │              ┌─────────────────────────┐                      │  │
│  │              │ ██████░░░░░░░░░░░░░░░░  │  (skeleton row)      │  │
│  │              └─────────────────────────┘                      │  │
│  │              ┌─────────────────────────┐                      │  │
│  │              │ ████████████░░░░░░░░░░  │  (skeleton row)      │  │
│  │              └─────────────────────────┘                      │  │
│  │                                                               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│              🦌 Support @Deer_NGO — Donate to earn points!            │
│              https://easydonate.app/deerngo0                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Loading State Specifications:**

| Element | Value |
|---------|-------|
| Technique | DaisyUI skeleton/shimmer rows (5 placeholder rows) |
| Header | Visible immediately (static content) |
| Footer | Visible immediately |
| Podium Area | Skeleton placeholders for top 3 spots |
| Timing | Shown for initial load (API call <2s target per OBJ-04) |
| Auto-refresh indicator | Small subtle spinner or text: "Updating..." in corner |

**Mobile:** Same skeleton pattern, narrower rows.

---

## 4. Wireframe Annotations

| # | Element | Behavior | Notes |
|---|---------|----------|-------|
| 1 | Podium Cards (Top 3) | Static display, no interaction | Gold (#1), Silver (#2), Bronze (#3) styling via DaisyUI |
| 2 | Scoreboard Table Rows | Hover highlight, no click action | Zebra striping for readability |
| 3 | Pagination | Click to navigate pages | DaisyUI pagination component. "← Previous" disabled on page 1, "Next →" disabled on last page |
| 4 | Auto-Refresh | Polls `GET /api/v1/scoreboard` every 60s | Shows subtle indicator when refreshing. On error → error state |
| 5 | Donate Link | External link, opens in new tab | Links to `https://easydonate.app/deerngo0` |
| 6 | Try Again Button | Re-fetches `GET /api/v1/scoreboard` | Only visible in error state |
| 7 | Skeleton Loader | Shown during API fetch | DaisyUI skeleton component, 5 placeholder rows |

---

## 5. API Data Mapping

> How the API response maps to UI elements.

| API Field | UI Element | Notes |
|-----------|-----------|-------|
| `data[n].rank` | "#N" label | Positions 1-3 use 🥇🥈🥉 emoji instead of numeric rank |
| `data[n].display_name` | "Display Name" column | Primary identifier shown |
| `data[n].youtube_handle` | "@handle" subtext | Shown smaller, muted color |
| `data[n].total_points` | "Points" column | Formatted with locale (e.g., "1,500 pts") |
| `data[n].donation_count` | "N donations" subtext | Shown on podium cards only; hidden in table rows on mobile |
| `meta.total` | Total viewers count | Shown near header: "N contributors" |
| `meta.page` / `meta.pages` | Pagination state | Page X of Y |
| `meta.hasNext` / `meta.hasPrev` | Pagination button state | Enable/disable Previous/Next |
| `data: []`, `meta.total: 0` | Empty state trigger | Show WF-003 |
| fetch error / non-200 | Error state trigger | Show WF-004 |

---

## 6. Responsive Breakpoints

| Breakpoint | Width | Layout |
|-----------|-------|--------|
| Mobile | < 640px | WF-002 — card list, stacked podium, compact pagination |
| Tablet | 640px – 1024px | Hybrid — table with reduced columns, side-by-side podium cards |
| Desktop | > 1024px | WF-001 — full table, row podium cards, standard pagination |

---

## 7. Component Library Mapping (DaisyUI)

| UI Element | DaisyUI Class | Notes |
|-----------|---------------|-------|
| Scoreboard Container | `card bg-base-100 shadow-xl` | Main content area |
| Podium Cards | `card bg-base-200` with `badge` for rank | Custom gold/silver/bronze accents |
| Scoreboard Table | `table table-zebra` | Zebra striping built-in |
| Pagination | `join` with `btn` children | DaisyUI pagination pattern |
| Skeleton Loader | `skeleton h-4 w-full` | 5 rows during loading |
| Error Alert | `alert alert-warning` | Warning state display |
| Empty State | `hero` with centered content | Empty state layout |
| Donate Button | `btn btn-primary` | Links to EasyDonate |
| Try Again Button | `btn btn-outline` | Retry API call |
| Auto-refresh Indicator | `loading loading-spinner loading-xs` | Subtle spinner |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[022_API_specification]] | §4.3 — Scoreboard endpoint response structure |
| [[028_style_guide]] | Visual design tokens applied to these wireframes |
| [[012_user_stories]] | US-030 (Public Scoreboard Page), US-031 (Scoreboard API) |
| [[013_acceptance_criteria]] | AC-030a → AC-030e, AC-031a → AC-031e |
| [[029_architecture_overview]] | Tech stack: Next.js + Tailwind + DaisyUI |
| [[021_architecture_decision_records]] | ADR-010 — DaisyUI decision |

---

> **Template Standard:** Based on ISO 9241-210
> **Usage:** These wireframes define the scoreboard page structure. Implement with Tailwind CSS + DaisyUI. All states (loading, empty, error, happy path) must be handled per the spec above.
