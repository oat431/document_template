---
document_type: Wireframes (Low-fi)
version: "1.1"
status: Draft
author: "UX/UI Persona / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [wireframes, low-fidelity, ux, layout, scoreboard, deerngo-bot, privacy]
standard_ref:
  - ISO 9241-210 — Human-Centred Design
parent_project: "Deerngo Bot — VRM"
---

# Wireframes (Low-fidelity) — Deerngo Bot Scoreboard

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 1.1 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Privacy change:** The scoreboard displays normalized YouTube handles and points only. It does not display YouTube display names, donor names, donation counts, or stable user IDs.

---

## 1. Purpose

Define the structure and states for the single Phase 1 public scoreboard page. The backend supplies only active, public members with points greater than zero.

## 2. Wireframe Index

| # | Screen | Device | Status |
|---|--------|--------|--------|
| WF-001 | Scoreboard — Happy Path | Desktop | Draft |
| WF-002 | Scoreboard — Happy Path | Mobile | Draft |
| WF-003 | Scoreboard — Empty State | Desktop + Mobile | Draft |
| WF-004 | Scoreboard — Error State | Desktop + Mobile | Draft |
| WF-005 | Scoreboard — Loading State | Desktop + Mobile | Draft |

## 3. Wireframe Specifications

### WF-001: Scoreboard — Happy Path (Desktop)

> The scoreboard has eligible active public members with points. The UI shows normalized YouTube handles and point totals only; pagination is available for more.

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        🦌 DEERNGO BOT                              │
│                    Viewer Contribution Leaderboard                 │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  🥇 @topdonor                                      1,500 pts  │  │
│  │  🥈 @viewer1                                         500 pts  │  │
│  │  🥉 @deerfan                                         420 pts  │  │
│  │                                                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ #   │ YouTube Handle       │ Points                    │ │  │
│  │  ├─────┼──────────────────────┼───────────────────────────┤ │  │
│  │  │  4  │ @forestfriend        │ 350                       │ │  │
│  │  │  5  │ @naturelover         │ 280                       │ │  │
│  │  │  6  │ @streamfan           │ 200                       │ │  │
│  │  │  7  │ @greenheart          │ 150                       │ │  │
│  │  │  8  │ @deersupporter       │ 100                       │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  │                                                               │  │
│  │            [← Previous]  Page 1 of 3  [Next →]               │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│       🦌 Support @Deer_NGO — Donate to earn points!                │
│       https://easydonate.app/deerngo0                             │
└─────────────────────────────────────────────────────────────────────┘
```

**Layout Specifications:**

| Element | Position | Priority |
|---------|----------|----------|
| Page Header | Top center | 🔴 Brand identity |
| Podium Section (Top 3) | Below header | 🔴 Hero content |
| Scoreboard Table | Below podium | 🔴 Core content |
| Pagination | Below table | 🟡 Navigation |
| Footer CTA | Bottom | 🟡 Engagement |

**Interaction Notes:**

- Top three receive gold/silver/bronze visual distinction.
- Table rows use zebra striping and hover highlight.
- Pagination is keyboard accessible and disabled at boundaries.
- Auto-refresh target is every 60 seconds with a subtle indicator.

### WF-002: Scoreboard — Happy Path (Mobile)

```text
┌───────────────────────────┐
│      🦌 DEERNGO BOT       │
│  Viewer Leaderboard       │
│                           │
│  🥇 @topdonor             │
│      1,500 pts            │
│                           │
│  🥈 @viewer1              │
│        500 pts            │
│                           │
│  🥉 @deerfan              │
│        420 pts            │
│                           │
│  #4  @forestfriend  350   │
│  #5  @naturelover   280   │
│  #6  @streamfan     200   │
│                           │
│  [← Prev] Page 1/3 [→]    │
│  🦌 Donate to earn points │
│  easydonate.app/deerngo0  │
└───────────────────────────┘
```

| Element | Adjustment |
|---------|-----------|
| Podium Cards | Stack vertically |
| Scoreboard | Card/list rows |
| Fields | Normalized handle + points; no display name |
| Pagination | Compact, keyboard accessible |
| Footer CTA | Single line, smaller font |

### WF-003: Scoreboard — Empty State

> No eligible active public members have points yet.

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        🦌 DEERNGO BOT                              │
│                    Viewer Contribution Leaderboard                 │
│                                                                     │
│                         🦌                                          │
│             No contributors yet. Be the first!                     │
│       Support the stream by donating at the link below.            │
│                         [Donate Now →]                             │
│                                                                     │
│       🦌 Support @Deer_NGO — Donate to earn points!                │
│       https://easydonate.app/deerngo0                             │
└─────────────────────────────────────────────────────────────────────┘
```

### WF-004: Scoreboard — Error State

> Backend API is unreachable or returns 5xx.

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        🦌 DEERNGO BOT                              │
│                    Viewer Contribution Leaderboard                 │
│                                                                     │
│              ⚠️ Scoreboard temporarily unavailable                 │
│        We're having trouble loading the leaderboard.                │
│                         [Try Again]                                │
│                                                                     │
│       🦌 Support @Deer_NGO — Donate to earn points!                │
│       https://easydonate.app/deerngo0                             │
└─────────────────────────────────────────────────────────────────────┘
```

### WF-005: Scoreboard — Loading State

> Initial page load or refresh. Use five accessible skeleton rows and keep static header/footer visible.

## 4. API Data Mapping

| API Field | UI Element | Notes |
|-----------|-----------|-------|
| `data[n].rank` | Rank label | Top three may use medal icons |
| `data[n].youtube_handle` | Handle column | Normalized handle is the only public identity field |
| `data[n].total_points` | Points column | Locale-formatted points |
| `meta.total` | Contributor count | Eligible public contributors only |
| `meta.page` / `meta.pages` | Pagination state | Page X of Y |
| `meta.hasNext` / `meta.hasPrev` | Button state | Enable/disable navigation |
| `data: []` | Empty state | Show WF-003 |
| Fetch error/non-200 | Error state | Show WF-004 |

## 5. Responsive Breakpoints

| Breakpoint | Width | Layout |
|-----------|-------|--------|
| Mobile | <640px | Card/list, stacked podium |
| Tablet | 640–1024px | Reduced table columns |
| Desktop | >1024px | Full table and podium |

## 6. Accessibility and Privacy Annotations

- Use semantic headings/table/list roles.
- Ensure keyboard access to pagination and retry.
- Never render/display-name, user-ID, donor-name, message, donation-count, or private fields.
- Do not make hidden data available through HTML attributes, tooltips, analytics events, or client logs.
- Confirm the backend response allowlist in frontend tests.

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[022_API_specification]] | Public scoreboard contract |
| [[028_style_guide]] | Visual design tokens |
| [[012_user_stories]] | US-030 and US-031 |
| [[013_acceptance_criteria]] | AC-030a–f and AC-031a–f |
| [[029_architecture_overview]] | Current architecture |

---

> **Template Standard:** Based on ISO 9241-210
> **Usage:** Current privacy-safe scoreboard blueprint.
---
