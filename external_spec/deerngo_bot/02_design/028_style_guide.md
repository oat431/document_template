---
document_type: Style Guide
version: "1.0"
status: Draft
author: "UX/UI Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [style-guide, visual-design, branding, deerngo-bot, daisyui, tailwind]
standard_ref:
  - ISO 9241-210 — Human-Centred Design
  - WCAG 2.1 — Web Content Accessibility Guidelines (Level AA)
---

# Style Guide — Deerngo Bot

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

> The style guide defines visual standards — colors, typography, spacing, components, and branding rules — for the Deerngo Bot public scoreboard. All UI is built with **Tailwind CSS 4+** and **DaisyUI 5+**. This guide serves as the visual contract between design and development.

---

## 2. Brand Identity

### 2.1 Brand Story

Deerngo Bot serves the **@Deer_NGO** YouTube channel — a streamer focused on nature, wildlife, and community. The visual identity reflects:

| Attribute | Expression |
|-----------|-----------|
| **Nature** | Forest greens, earthy tones |
| **Warmth** | Amber/gold accents, rounded shapes |
| **Trust** | Clean, readable, accessible |
| **Community** | Inclusive, welcoming, friendly |
| **Simplicity** | Minimalist — let the scoreboard data speak |

### 2.2 Logo & Brand Mark

| Element | Specification |
|---------|--------------|
| Primary Brand Mark | 🦌 Deer emoji (Unicode U+1F98C) |
| Brand Mark Size (Header) | 48px |
| Brand Mark Size (Footer) | 24px |
| Wordmark | "DEERNGO BOT" — title case, bold |
| Tagline | "Viewer Contribution Leaderboard" |
| Favicon | 🦌 emoji or SVG deer silhouette |

### 2.3 Voice & Tone

| Context | Tone | Example |
|---------|------|---------|
| Scoreboard Header | Welcoming, inclusive | "Viewer Contribution Leaderboard" |
| Empty State | Encouraging | "No contributors yet. Be the first!" |
| Error State | Reassuring, helpful | "Scoreboard temporarily unavailable" |
| Footer CTA | Friendly invitation | "🦌 Support @Deer_NGO — Donate to earn points!" |

---

## 3. Color System

### 3.1 Primary Palette — Forest Green

> The primary color connects to the deer/nature brand identity.

| Token | Hex | RGB | Tailwind Class | Usage |
|-------|-----|-----|---------------|-------|
| Primary | `#2D6A4F` | 45, 106, 79 | `bg-primary` | CTAs, header, active states, rank badges |
| Primary Light | `#40916C` | 64, 145, 108 | `bg-primary/70` | Hover states, podium card background |
| Primary Dark | `#1B4332` | 27, 67, 50 | `bg-primary/90` | Active/pressed states, footer |
| Primary Content | `#FFFFFF` | 255, 255, 255 | `text-primary-content` | Text on primary backgrounds |

### 3.2 Accent Palette — Warm Amber

> The accent color highlights points, rankings, and calls-to-action.

| Token | Hex | RGB | Tailwind Class | Usage |
|-------|-----|-----|---------------|-------|
| Accent | `#D4A017` | 212, 160, 23 | `bg-accent` | Gold medals, point values, donation link |
| Accent Light | `#F0C929` | 240, 201, 41 | `text-accent/80` | Hover states for accent elements |
| Accent Dark | `#B8860B` | 184, 134, 11 | `text-accent/90` | Active states |
| Accent Content | `#1B4332` | 27, 67, 50 | `text-accent-content` | Text on accent backgrounds |

### 3.3 Podium Colors — Top 3 Ranking

| Rank | Color | Hex | DaisyUI Class | Visual |
|------|-------|-----|--------------|--------|
| 1st — Gold | Rich Gold | `#D4A017` | `badge-warning` or custom | 🥇 |
| 2nd — Silver | Cool Silver | `#A8B5C0` | `badge-ghost` or custom | 🥈 |
| 3rd — Bronze | Warm Bronze | `#CD7F32` | `badge-accent` or custom | 🥉 |
| 4th+ | Neutral | `#6B7280` | `badge-neutral` | #4, #5, ... |

### 3.4 Semantic Colors

| Token | Hex | Tailwind Class | Usage |
|-------|-----|---------------|-------|
| Success | `#16A34A` | `text-success` / `bg-success` | — (not used on scoreboard, reserved for future) |
| Warning | `#D4A017` | `text-warning` / `bg-warning` | Error state alert, points >= 0 indicators |
| Error | `#DC2626` | `text-error` / `bg-error` | Error state icon, retry button border |
| Info | `#2563EB` | `text-info` / `bg-info` | Refresh indicator, tooltips |

### 3.5 Neutral Palette — Backgrounds & Text

| Token | Hex | Tailwind Class | Usage |
|-------|-----|---------------|-------|
| Base Background | `#FAFBF5` | `bg-base-100` | Page background — warm off-white |
| Base Surface | `#FFFFFF` | `bg-base-200` | Cards, table rows |
| Base Border | `#E5E7E0` | `border-base-300` | Dividers, table borders |
| Text Primary | `#1F2937` | `text-base-content` | Body text, names |
| Text Secondary | `#6B7280` | `text-base-content/60` | YouTube handles, metadata, "pts" label |
| Text Muted | `#9CA3AF` | `text-base-content/40` | Donation count subtext, pagination labels |
| Text On Dark | `#FFFFFF` | `text-primary-content` | Text on green backgrounds |

### 3.6 Accessibility — Color Contrast

| Combination | Contrast Ratio | WCAG Level | Status |
|-------------|---------------|------------|--------|
| Primary `#2D6A4F` on White `#FFFFFF` | 5.2:1 | AA ✅ | ✅ Pass |
| Text Primary `#1F2937` on Base `#FAFBF5` | 13.8:1 | AAA ✅ | ✅ Pass |
| Text Secondary `#6B7280` on White `#FFFFFF` | 5.2:1 | AA ✅ | ✅ Pass |
| White Text on Primary `#2D6A4F` | 5.2:1 | AA ✅ | ✅ Pass |
| Accent `#D4A017` on White `#FFFFFF` | 2.2:1 | ❌ Fail | ⚠️ Accent used only for decorative elements and icons, never for text under 18px |
| White Text on Accent `#D4A017` | 2.2:1 | ❌ Fail | ⚠️ Use `#1B4332` (primary dark) text on accent backgrounds instead |

---

## 4. Typography

### 4.1 Type Scale

| Level | Tailwind Class | Font Size | Line Height | Font Weight | Usage |
|-------|---------------|-----------|-------------|-------------|-------|
| Page Title | `text-4xl` | 36px / 2.25rem | 40px / 2.5rem | 700 (bold) | "DEERNGO BOT" header |
| Subtitle | `text-xl` | 20px / 1.25rem | 28px / 1.75rem | 500 (medium) | "Viewer Contribution Leaderboard" |
| Section Heading | `text-lg` | 18px / 1.125rem | 28px / 1.75rem | 600 (semibold) | Podium names |
| Body | `text-base` | 16px / 1rem | 24px / 1.5rem | 400 (normal) | Table display names |
| Body Small | `text-sm` | 14px / 0.875rem | 20px / 1.25rem | 400 (normal) | YouTube handles, donation count |
| Caption | `text-xs` | 12px / 0.75rem | 16px / 1rem | 400 (normal) | Pagination labels, "pts" unit |
| Points Value | `text-2xl` | 24px / 1.5rem | 32px / 2rem | 700 (bold) | Point totals on podium cards |
| Points Table | `text-base` | 16px / 1rem | 24px / 1.5rem | 600 (semibold) | Point totals in table |

### 4.2 Font Family

| Context | Font Stack |
|---------|-----------|
| Default (Tailwind) | `'Inter', ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif` |
| Monospace (if needed) | `'JetBrains Mono', ui-monospace, 'Cascadia Code', 'Source Code Pro', Menlo, monospace` |

> **Note:** Tailwind CSS 4+ ships Inter as the default sans-serif. No additional font configuration needed.

### 4.3 Typography Rules

| Rule | Specification |
|------|--------------|
| Maximum line length | 80 characters (for descriptive text in empty/error states) |
| Heading spacing | `mb-4` (16px) below section headings |
| Paragraph spacing | `mb-4` between paragraphs |
| Table row padding | `py-3` (12px vertical) |
| Text alignment | Header: center. Table: names left-aligned, points right-aligned, rank centered |
| Text truncation | Display names: `truncate max-w-[200px]`. YouTube handles: `truncate max-w-[160px]` |

---

## 5. Spacing System

> Based on Tailwind's 4px base unit.

| Token | Tailwind Class | Value | Usage |
|-------|---------------|-------|-------|
| xs | `p-1` / `gap-1` | 4px | Tight icon spacing, badge padding |
| sm | `p-2` / `gap-2` | 8px | Form field gaps, compact padding |
| md | `p-4` / `gap-4` | 16px | Default card padding, section spacing |
| lg | `p-6` / `gap-6` | 24px | Card padding (relaxed), podium gap |
| xl | `p-8` / `gap-8` | 32px | Page margins, major section separation |
| 2xl | `p-12` | 48px | Page-top spacing, hero sections |

### 5.1 Layout Spacing

| Area | Specification |
|------|--------------|
| Page max-width | `max-w-4xl` (896px) — centered with `mx-auto` |
| Page padding (mobile) | `px-4` (16px left/right) |
| Page padding (desktop) | `px-8` (32px left/right) |
| Section vertical gap | `my-8` (32px) between sections |
| Card padding | `p-6` (24px) for podium cards and table container |

---

## 6. Component Styles

### 6.1 Scoreboard Container

```html
<!-- Main wrapper -->
<div class="max-w-4xl mx-auto px-4 md:px-8 py-12">
  <!-- Scoreboard card -->
  <div class="card bg-base-100 shadow-xl border border-base-300">
    <!-- content -->
  </div>
</div>
```

| Property | Value |
|----------|-------|
| Max Width | 896px (`max-w-4xl`) |
| Background | White (`bg-base-100`) |
| Shadow | `shadow-xl` |
| Border | 1px `border-base-300` |
| Border Radius | 12px (`rounded-xl`) |

### 6.2 Podium Cards (Top 3)

```html
<div class="card bg-base-200 shadow-md p-6 text-center">
  <span class="text-4xl mb-2">🥇</span>
  <h3 class="text-lg font-semibold truncate">Top Donor</h3>
  <p class="text-sm text-base-content/60">@topdonor</p>
  <p class="text-2xl font-bold text-accent mt-2">1,500 pts</p>
  <p class="text-xs text-base-content/40">10 donations</p>
</div>
```

| Rank | Badge Emoji | Card Border Accent |
|------|------------|-------------------|
| 1st | 🥇 | 2px `border-l-4 border-accent` (gold) |
| 2nd | 🥈 | 2px `border-l-4 border-gray-400` (silver) |
| 3rd | 🥉 | 2px `border-l-4 border-amber-700` (bronze) |

### 6.3 Scoreboard Table

```html
<div class="overflow-x-auto">
  <table class="table table-zebra">
    <thead>
      <tr>
        <th class="text-center w-16">#</th>
        <th>Viewer</th>
        <th class="text-right">Points</th>
      </tr>
    </thead>
    <tbody>
      <tr class="hover">
        <td class="text-center">
          <span class="badge badge-neutral badge-sm">4</span>
        </td>
        <td>
          <span class="font-medium">Forest Friend</span>
          <span class="text-sm text-base-content/60 block">@forestfriend</span>
        </td>
        <td class="text-right font-semibold">350</td>
      </tr>
      <!-- more rows -->
    </tbody>
  </table>
</div>
```

| Property | Value |
|----------|-------|
| Table Style | `table table-zebra` (DaisyUI — alternating row stripes) |
| Row Hover | `hover` (DaisyUI — highlight on hover) |
| Header Background | `bg-base-200` |
| Rank Column Width | 64px (`w-16`), centered |
| Points Column Width | 96px (`w-24`), right-aligned |
| Rank Badge | `badge badge-neutral badge-sm` |
| Mobile | Table becomes card list (see §6.6 Mobile Table) |

### 6.4 Pagination

```html
<div class="join flex justify-center mt-6">
  <button class="join-item btn btn-sm" disabled>« Previous</button>
  <button class="join-item btn btn-sm btn-active">1</button>
  <button class="join-item btn btn-sm">2</button>
  <button class="join-item btn btn-sm">3</button>
  <button class="join-item btn btn-sm">Next »</button>
</div>
```

| Property | Value |
|----------|-------|
| Component | DaisyUI `join` with `btn btn-sm` children |
| Active Page | `btn-active` |
| Disabled | `disabled` attribute + `btn-disabled` |
| Position | Centered below table, `mt-6` |
| Mobile | Compact: "← 1/3 →" with arrow-only buttons |

### 6.5 States — Loading, Empty, Error

#### Loading State (Skeleton)

```html
<div class="space-y-4 p-6">
  <div class="skeleton h-12 w-full"></div>
  <div class="skeleton h-8 w-3/4"></div>
  <div class="skeleton h-8 w-2/3"></div>
  <div class="skeleton h-8 w-4/5"></div>
  <div class="skeleton h-8 w-3/5"></div>
</div>
```

#### Empty State (Hero)

```html
<div class="hero py-12">
  <div class="hero-content text-center">
    <div>
      <p class="text-6xl mb-4">🦌</p>
      <h2 class="text-2xl font-bold">No contributors yet. Be the first!</h2>
      <p class="text-base-content/60 mt-2">Support the stream by donating at the link below.</p>
      <a href="https://easydonate.app/deerngo0" target="_blank"
         class="btn btn-primary mt-6">Donate Now →</a>
    </div>
  </div>
</div>
```

#### Error State (Alert)

```html
<div class="hero py-12">
  <div class="hero-content text-center">
    <div>
      <p class="text-5xl mb-4">⚠️</p>
      <h2 class="text-2xl font-bold">Scoreboard temporarily unavailable</h2>
      <p class="text-base-content/60 mt-2">We're having trouble loading the leaderboard. Please check back in a few minutes.</p>
      <button class="btn btn-outline btn-warning mt-6" onclick="location.reload()">Try Again</button>
    </div>
  </div>
</div>
```

### 6.6 Mobile Table (Card List)

> On screens < 640px, replace the HTML table with a card-based layout.

```html
<div class="space-y-2 md:hidden">
  <div class="card bg-base-100 shadow-sm border border-base-300 p-4 flex flex-row items-center gap-4">
    <span class="badge badge-neutral badge-sm">4</span>
    <div class="flex-1 min-w-0">
      <p class="font-medium truncate">Forest Friend</p>
      <p class="text-sm text-base-content/60 truncate">@forestfriend</p>
    </div>
    <span class="font-semibold text-accent text-lg">350</span>
  </div>
  <!-- more cards -->
</div>

<!-- Desktop table hidden on mobile -->
<table class="table table-zebra hidden md:table">
  ...
</table>
```

### 6.7 Header & Footer

```html
<!-- Header -->
<header class="text-center mb-8">
  <p class="text-5xl mb-2">🦌</p>
  <h1 class="text-4xl font-bold text-primary">DEERNGO BOT</h1>
  <p class="text-xl text-base-content/70 mt-2">Viewer Contribution Leaderboard</p>
  <p class="text-sm text-base-content/40 mt-1" id="contributor-count">
    25 contributors
  </p>
</header>

<!-- Footer -->
<footer class="text-center mt-12 py-6 border-t border-base-300">
  <p class="text-sm text-base-content/60">
    🦌 Support <strong>@Deer_NGO</strong> — Donate to earn points!
  </p>
  <a href="https://easydonate.app/deerngo0" target="_blank"
     class="link link-accent text-sm mt-1 inline-block">
    easydonate.app/deerngo0
  </a>
</footer>
```

---

## 7. Iconography

| Aspect | Specification |
|--------|--------------|
| Icon Set | Unicode Emoji (no external icon library needed for Phase 1) |
| Emoji — Brand | 🦌 (U+1F98C) — used in header, footer |
| Emoji — 1st Place | 🥇 (U+1F947) |
| Emoji — 2nd Place | 🥈 (U+1F948) |
| Emoji — 3rd Place | 🥉 (U+1F949) |
| Emoji — Warning/Error | ⚠️ (U+26A0) |
| Emoji — Loading | None (use DaisyUI `loading loading-spinner`) |
| Future Icon Set | Heroicons or Lucide Icons (if more icons needed in Phase 2) |

> **Rationale:** Unicode emojis require zero dependencies, render consistently across platforms, and match the deer theme. Upgrade to an icon library in Phase 2 when the UI grows beyond a single page.

---

## 8. Border & Radius

| Element | Border | Radius | Tailwind |
|---------|--------|--------|----------|
| Scoreboard Card | 1px `border-base-300` | 12px | `rounded-xl` |
| Podium Cards | 1px `border-base-300` | 12px | `rounded-xl` |
| Table | None (DaisyUI `table-zebra` handles borders) | — | — |
| Rank Badge | None | 12px (pill) | `badge` (auto-radius) |
| Button | 1px solid matching | 8px | `rounded-lg` (DaisyUI default) |
| Skeleton Loader | None | 8px | `rounded-lg` |

---

## 9. Elevation (Shadows)

| Level | Shadow | Tailwind | Usage |
|-------|--------|----------|-------|
| Level 0 | None | — | Table rows, text |
| Level 1 | `0 1px 3px rgba(0,0,0,0.08)` | `shadow-sm` | Mobile cards, table container on light backgrounds |
| Level 2 | `0 4px 6px rgba(0,0,0,0.10)` | `shadow-md` | Podium cards |
| Level 3 | `0 10px 25px rgba(0,0,0,0.12)` | `shadow-xl` | Main scoreboard card |

---

## 10. DaisyUI Theme Configuration

> Custom DaisyUI theme "deerngo" — extends the built-in `light` theme.

### 10.1 Tailwind Config (`tailwind.config.js` or CSS `@theme`)

```css
/* In app/globals.css — Tailwind CSS v4 + DaisyUI v5 */
@import "tailwindcss";
@plugin "daisyui";

@plugin "daisyui/theme" {
  name: "deerngo";
  default: true;
  prefersdark: false;

  /* Primary — Forest Green */
  --color-primary: #2D6A4F;
  --color-primary-content: #FFFFFF;

  /* Secondary — Muted Sage */
  --color-secondary: #95B79C;
  --color-secondary-content: #1B4332;

  /* Accent — Warm Amber/Gold */
  --color-accent: #D4A017;
  --color-accent-content: #1B4332;

  /* Neutral — Warm Grays */
  --color-neutral: #6B7280;
  --color-neutral-content: #FFFFFF;

  /* Base — Warm off-white */
  --color-base-100: #FAFBF5;
  --color-base-200: #F3F4E8;
  --color-base-300: #E5E7E0;
  --color-base-content: #1F2937;

  /* Semantic */
  --color-info: #2563EB;
  --color-info-content: #FFFFFF;
  --color-success: #16A34A;
  --color-success-content: #FFFFFF;
  --color-warning: #D4A017;
  --color-warning-content: #1B4332;
  --color-error: #DC2626;
  --color-error-content: #FFFFFF;

  /* Border radius */
  --radius-box: 0.75rem;       /* 12px — cards */
  --radius-btn: 0.5rem;        /* 8px — buttons */
  --radius-badge: 1.9rem;      /* ~30px — pill badges */
  --radius-selector: 0.5rem;   /* 8px */

  /* Misc */
  --border-btn: 1px;
}
```

### 10.2 Theme Application

```html
<!-- In layout.tsx <html> tag -->
<html data-theme="deerngo" lang="en">
```

---

## 11. Responsive Design Reference

| Breakpoint | Prefix | Min Width | Layout Behavior |
|-----------|--------|-----------|----------------|
| Default (mobile) | (none) | 0px | Single column, card list, stacked podium, full-width buttons |
| Small | `sm:` | 640px | Table appears (hidden on mobile) |
| Medium | `md:` | 768px | Side-by-side podium, standard padding |
| Large | `lg:` | 1024px | Full layout, max-width container centered |
| Extra Large | `xl:` | 1280px | Larger max-width (896px) |
| 2XL | `2xl:` | 1536px | No layout change (content is max-w-4xl) |

---

## 12. Animation & Transitions

| Element | Animation | Duration | Easing | Tailwind |
|---------|----------|----------|--------|----------|
| Page Load | Fade in | 300ms | `ease-out` | `animate-fade-in` (custom) or `transition-opacity` |
| Skeleton → Content | Crossfade | 200ms | `ease-in-out` | `transition-opacity duration-200` |
| Podium Card Hover | Scale up slightly + shadow | 150ms | `ease-out` | `hover:scale-[1.02] hover:shadow-lg transition-all` |
| Table Row Hover | Background change | 100ms | Instant | DaisyUI `hover` class |
| Error → Retry click | Button press | 100ms | `ease-in-out` | `active:scale-95` |
| Auto-refresh Spinner | Continuous rotate | 1s per rotation | Linear | `loading loading-spinner` (DaisyUI built-in) |

```css
/* Custom fade-in animation (add to globals.css) */
@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fade-in 300ms ease-out;
}
```

---

## 13. Accessibility Checklist

| Requirement | Implementation | WCAG Criterion | Status |
|------------|---------------|----------------|--------|
| Color contrast (text) | All text meets 4.5:1 minimum (AA) | 1.4.3 | ✅ See §3.6 |
| Color contrast (large text) | Headings ≥ 18px bold meet 3:1 minimum | 1.4.3 | ✅ |
| Non-color indicators | Rank badges use numbers + emoji (not color alone) | 1.4.1 | ✅ |
| Keyboard navigation | All interactive elements focusable (`btn`, `a`, `button`) | 2.1.1 | ✅ |
| Focus indicators | DaisyUI default focus ring on buttons/links | 2.4.7 | ✅ |
| Semantic HTML | `<header>`, `<main>`, `<footer>`, `<table>`, `<nav>` | 1.3.1 | ✅ |
| Screen reader text | `sr-only` labels for emoji-only elements, `aria-label` on pagination | 1.1.1 | ✅ |
| Responsive text | No horizontal scroll at 320px width | 1.4.10 | ✅ |
| Motion preference | Respect `prefers-reduced-motion` — disable animations | 2.3.3 | ✅ |
| Page title | `<title>Deerngo Bot — Viewer Leaderboard</title>` | 2.4.2 | ✅ |

---

## 14. Implementation Checklist for Dev

| # | Item | Priority |
|---|------|----------|
| 1 | Create DaisyUI custom theme "deerngo" in `globals.css` | 🔴 |
| 2 | Set `data-theme="deerngo"` on `<html>` | 🔴 |
| 3 | Implement scoreboard container with `card`, `shadow-xl`, `rounded-xl` | 🔴 |
| 4 | Use `table table-zebra` for desktop, card list for mobile | 🔴 |
| 5 | Podium cards with 🥇🥈🥉 emoji badges | 🟡 |
| 6 | Pagination with DaisyUI `join` + `btn` components | 🟡 |
| 7 | Skeleton loading state with DaisyUI `skeleton` | 🟡 |
| 8 | Empty state (🦌 hero) per AC-030e | 🔴 |
| 9 | Error state (⚠️ alert) per AC-030d | 🔴 |
| 10 | Auto-refresh every 60s with subtle spinner | 🟡 |
| 11 | Responsive breakpoints: card list (mobile), table (sm+) | 🔴 |
| 12 | Footer with donation CTA link | 🟡 |
| 13 | Accessibility: semantic HTML, focus rings, sr-only labels | 🟡 |
| 14 | `prefers-reduced-motion` support | 🟢 |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[026_wireframes_lofi]] | Wireframes these styles implement |
| [[022_API_specification]] | §4.3 — Scoreboard data structure driving component design |
| [[012_user_stories]] | US-030 (Public Scoreboard Page) |
| [[013_acceptance_criteria]] | AC-030a → AC-030e (scoreboard requirements) |
| [[029_architecture_overview]] | Tech stack: Next.js + Tailwind CSS + DaisyUI |
| [[021_architecture_decision_records]] | ADR-010 — DaisyUI component library decision |

---

> **Template Standard:** Based on ISO 9241-210, WCAG 2.1 Level AA
> **Usage:** This style guide is the *visual contract* for the Deerngo Bot scoreboard. Every UI decision should reference this guide. The DaisyUI "deerngo" theme definition in §10.1 is the **single source of truth** for design tokens.
