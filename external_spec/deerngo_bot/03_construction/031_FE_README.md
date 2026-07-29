---
document_type: README (Frontend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "FE"
classification: "Internal"
tags: [readme, developer-guide, onboarding, nextjs, frontend, react, tailwind, daisyui]
standard_ref:
  - SWEBOK v4 — Construction
  - 12-Factor App Methodology
parent_project: "Deerngo Bot — VRM"
---

# README — Deerngo Bot Frontend (Next.js)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Repo:** `deerngo-web` (frontend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## Project Header

# Deerngo Bot — Frontend 🦌

> Next.js frontend for the Deerngo Bot VRM system — public scoreboard displaying viewer contribution rankings.

| Aspect | Detail |
|--------|--------|
| **Framework** | Next.js 15+ (App Router) |
| **UI Library** | React 19 |
| **CSS** | Tailwind CSS 4+ + DaisyUI 5+ |
| **Data Fetching** | SWR |
| **Port** | 3008 |
| **Deployment** | Docker on homelab (`db-network`) |

---

## Quick Start

### Prerequisites

- **Node.js** 22 LTS
- **Backend** running on `:8008` (or set `NEXT_PUBLIC_API_URL`)

### Install & Run

```bash
# Install dependencies
npm install

# Development mode (hot reload)
npm run dev
# → Frontend starts on http://localhost:3008
```

### Production Build

```bash
npm run build
npm start
```

### Docker

```bash
npm run docker:build
docker run -p 3008:3008 deerngo-web:latest
```

### Verify

```bash
open http://localhost:3008
# Expected: Scoreboard page (may be empty initially)
```

---

## Configuration

| Variable | Required | Default | Description |
|----------|:--------:|---------|------------|
| `NEXT_PUBLIC_API_URL` | ✅ | `http://localhost:8008` | Go backend URL |

---

## Project Structure

```
deerngo-web/
├── src/
│   ├── app/
│   │   ├── layout.tsx          # Root layout (DaisyUI theme)
│   │   ├── page.tsx            # Scoreboard page
│   │   └── globals.css         # Tailwind + DaisyUI imports
│   ├── components/
│   │   ├── Scoreboard.tsx      # Scoreboard table component
│   │   ├── ScoreboardRow.tsx   # Individual row
│   │   ├── EmptyState.tsx      # "No contributors yet"
│   │   ├── ErrorState.tsx      # "Temporarily unavailable"
│   │   └── LoadingState.tsx    # Skeleton/spinner
│   └── lib/
│       ├── api.ts              # SWR fetcher for backend API
│       └── types.ts            # TypeScript types
├── public/
│   └── favicon.ico             # 🦌 deer emoji
├── package.json
├── tailwind.config.ts          # DaisyUI "deerngo" theme
├── tsconfig.json
├── next.config.ts
├── Dockerfile
└── README.md
```

---

## Design References

| Document | Path | Purpose |
|----------|------|---------|
| Wireframes | `02_design/026_FE_wireframes_lofi.md` | UI layout (5 screens) |
| Style Guide | `02_design/028_FE_style_guide.md` | Colors, typography, DaisyUI theme |
| API Specification | `02_design/022_SHARED_API_specification.md` | Scoreboard response structure |

---

## Testing

```bash
npm test              # Run Vitest
npm run lint          # ESLint
npm run type-check    # TypeScript check
```

---

## Related Documents

| Document | Path | Purpose |
|----------|------|---------|
| Wireframes | `02_design/026_FE_wireframes_lofi.md` | UI structure |
| Style Guide | `02_design/028_FE_style_guide.md` | Visual design |
| API Specification | `02_design/022_SHARED_API_specification.md` | Backend API contract |
| Build Scripts | `03_construction/034_FE_build_scripts.md` | Build pipeline |
| Dependency Manifest | `03_construction/036_FE_dependency_manifest.md` | Node dependencies |

---

> **Template Standard:** Based on SWEBOK v4, 12-Factor App
