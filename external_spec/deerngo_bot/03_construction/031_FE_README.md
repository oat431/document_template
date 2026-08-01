---
document_type: README (Frontend)
version: "0.2"
status: Draft
author: "SA / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "FE"
classification: "Internal"
tags: [readme, developer-guide, onboarding, nextjs, frontend, react, tailwind, daisyui, privacy]
standard_ref:
  - SWEBOK v4 — Construction
  - 12-Factor App Methodology
parent_project: "Deerngo Bot — VRM"
---

# README — Deerngo Bot Frontend (Next.js)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Repo:** `deerngo-web` (frontend)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> Phase 1 is a public read-only scoreboard for eligible registered members. It does not provide login or member self-service.

---

## Project Header

# Deerngo Bot — Frontend 🦌

> Next.js frontend for the public, privacy-filtered contributor scoreboard.

| Aspect | Detail |
|--------|--------|
| **Framework** | Next.js 15+ (App Router) |
| **UI Library** | React 19 |
| **CSS** | Tailwind CSS 4+ + DaisyUI 5+ |
| **Data Fetching** | SWR or fetch with controlled refresh |
| **Port** | 3008 |
| **Deployment** | Docker on homelab (`db-network`) |

## Public Data Contract

Consume `GET /api/v1/scoreboard` from `oat431/deerngo-bot`.

Allowed data:

```text
rank
youtube_handle
total_points
pagination metadata
```

Do not request, render, cache, or log:

- YouTube display names
- YouTube user IDs
- Raw EasyDonate donor names
- Donation messages
- Private/inactive/zero-point members

The backend is the source of truth for filtering. The frontend must still treat the response as untrusted input and render only the allowlisted fields.

## Quick Start

### Prerequisites

- Node.js 22 LTS
- Backend running on `:8008` (or set `NEXT_PUBLIC_API_URL`)

### Install & Run

```bash
npm install
npm run dev
# → http://localhost:3008
```

### Production Build

```bash
npm run build
npm start
```

### Verify

```bash
open http://localhost:3008
# Expected: scoreboard page with data, empty state, or unavailable state
```

## Configuration

| Variable | Required | Default | Description |
|----------|:--------:|---------|-------------|
| `NEXT_PUBLIC_API_URL` | ✅ | `http://localhost:8008` | Go backend base URL |

## Project Structure

```text
deerngo-web/
├── src/
│   ├── app/
│   │   ├── layout.tsx          # Root layout / Deer_NGO theme
│   │   ├── page.tsx            # Public scoreboard page
│   │   └── globals.css         # Tailwind + DaisyUI imports
│   ├── components/
│   │   ├── Scoreboard.tsx      # Allowlisted scoreboard table/list
│   │   ├── ScoreboardRow.tsx   # Handle + points row
│   │   ├── EmptyState.tsx      # "No contributors yet"
│   │   ├── ErrorState.tsx       # "Scoreboard temporarily unavailable"
│   │   └── LoadingState.tsx     # Skeleton/spinner
│   └── lib/
│       ├── api.ts              # Scoreboard fetcher
│       └── types.ts            # Public response types only
├── public/
├── package.json
├── tailwind.config.ts
├── tsconfig.json
├── next.config.ts
├── Dockerfile
└── README.md
```

## UI States

- **Loading:** accessible skeleton while the API request runs.
- **Happy path:** normalized handles and points, sorted by backend rank.
- **Empty:** `No contributors yet. Be the first!`.
- **Error:** `Scoreboard temporarily unavailable` with retry.
- **Responsive:** mobile card/list layout; no data hidden in inaccessible hover-only UI.
- **Refresh:** update within 60 seconds under normal backend/provider conditions.

## Privacy and Accessibility

- Do not add a display-name column back to the UI.
- Public scoreboard participation is controlled by the backend member visibility flag.
- Do not implement login/authentication in Phase 1.
- Use semantic headings/table or accessible list semantics, keyboard navigation, sufficient color contrast, and visible focus states.

## Testing

```bash
npm test
npm run lint
npm run type-check
npm run build
```

Tests must cover the API allowlist, private/inactive/zero-point exclusion, empty/error/loading states, pagination, responsive layout, and keyboard accessibility.

## Related Documents

| Document | Path |
|----------|------|
| User Stories | `01_requirement/012_user_stories.md` |
| Acceptance Criteria | `01_requirement/013_acceptance_criteria.md` |
| API Specification | `02_design/022_API_specification.md` |
| Architecture Overview | `02_design/029_architecture_overview.md` |
| Wireframes | `02_design/026_wireframes_lofi.md` |
| Style Guide | `02_design/028_style_guide.md` |
| Backend dependency | `https://github.com/oat431/deerngo-bot/issues/10` |

---

> **Template Standard:** Based on SWEBOK v4, 12-Factor App
> **Usage:** Frontend construction guide for the privacy-safe member scoreboard.
---
