---
document_type: Coding Standards (Frontend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "FE"
classification: "Internal"
tags: [coding-standards, typescript, nextjs, react, tailwind, daisyui, frontend]
standard_ref:
  - SWEBOK v4 — Construction
  - Airbnb JavaScript Style Guide
  - Next.js Documentation
parent_project: "Deerngo Bot — VRM"
---

# Coding Standards — Deerngo Bot Frontend (Next.js)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-web` (frontend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

> Coding standards for the Deerngo Bot Next.js frontend. Standards are **enforced** — if it doesn't pass ESLint and TypeScript check, it doesn't merge. Dev agents should follow these rules to produce consistent, readable code.

---

## 2. Language Standards

| Aspect | Standard | Tool |
|--------|---------|------|
| TypeScript | Strict mode (`strict: true`) | `tsc --noEmit` |
| Runtime | Node.js 22 LTS | `.nvmrc` |
| Framework | Next.js 15+ (App Router) | — |
| CSS | Tailwind CSS 4+ + DaisyUI 5+ | — |
| Formatting | 2 spaces, single quotes, trailing commas | Prettier |
| Linting | ESLint (Next.js config) | `next lint` |

---

## 3. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files — Components | PascalCase | `Scoreboard.tsx`, `ScoreboardRow.tsx` |
| Files — Utils/Hooks | camelCase | `useScoreboard.ts`, `fetchData.ts` |
| Files — Pages | lowercase (App Router) | `page.tsx`, `layout.tsx` |
| Components | PascalCase | `function Scoreboard() {}` |
| Hooks | `use` prefix | `useScoreboard`, `useViewerPoints` |
| Functions/Variables | camelCase | `fetchScoreboard`, `totalPoints` |
| Constants | UPPER_SNAKE_CASE | `API_BASE_URL`, `MAX_PAGE_SIZE` |
| Types/Interfaces | PascalCase | `interface ViewerPoints`, `type ScoreboardEntry` |
| CSS Classes | Tailwind utilities | No custom CSS unless necessary |

---

## 4. Project Structure

```
deerngo-web/
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout (DaisyUI theme, fonts)
│   │   ├── page.tsx                # Scoreboard page (server component)
│   │   ├── loading.tsx             # Loading skeleton
│   │   ├── error.tsx               # Error boundary
│   │   └── globals.css             # Tailwind + DaisyUI imports
│   ├── components/
│   │   ├── Scoreboard.tsx          # Main scoreboard (client component)
│   │   ├── ScoreboardRow.tsx       # Individual row
│   │   ├── EmptyState.tsx          # "No contributors yet"
│   │   ├── ErrorState.tsx          # "Temporarily unavailable"
│   │   ├── LoadingState.tsx        # Skeleton/spinner
│   │   └── Header.tsx              # 🦌 DEERNGO BOT header
│   ├── lib/
│   │   ├── api.ts                  # SWR fetcher + API client
│   │   ├── types.ts                # TypeScript types (API responses)
│   │   └── constants.ts            # API URLs, page sizes
│   └── hooks/
│       └── useScoreboard.ts        # SWR hook for scoreboard data
├── public/
│   └── favicon.ico
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── Dockerfile
└── README.md
```

---

## 5. Code Patterns

### 5.1 Server Component (Page)

```tsx
// ✅ Good — Server Component for initial data fetch
// src/app/page.tsx
import { Scoreboard } from '@/components/Scoreboard'
import { Header } from '@/components/Header'

export default async function ScoreboardPage() {
  return (
    <main className="min-h-screen bg-base-200">
      <Header />
      <Scoreboard />
    </main>
  )
}
```

### 5.2 Client Component (Data Fetching)

```tsx
// ✅ Good — Client Component with SWR for auto-refresh
// src/components/Scoreboard.tsx
'use client'

import useSWR from 'swr'
import { fetcher } from '@/lib/api'
import type { ScoreboardResponse } from '@/lib/types'
import { ScoreboardRow } from './ScoreboardRow'
import { EmptyState } from './EmptyState'
import { ErrorState } from './ErrorState'
import { LoadingState } from './LoadingState'

export function Scoreboard() {
  const { data, error, isLoading } = useSWR<ScoreboardResponse>(
    '/api/v1/scoreboard',
    fetcher,
    { refreshInterval: 30000 } // auto-refresh every 30s
  )

  if (isLoading) return <LoadingState />
  if (error) return <ErrorState />
  if (!data?.data.length) return <EmptyState />

  return (
    <div className="overflow-x-auto">
      <table className="table table-zebra">
        <thead>
          <tr>
            <th>Rank</th>
            <th>Viewer</th>
            <th>Points</th>
          </tr>
        </thead>
        <tbody>
          {data.data.map((entry) => (
            <ScoreboardRow key={entry.youtube_handle} entry={entry} />
          ))}
        </tbody>
      </table>
    </div>
  )
}

// ❌ Bad — fetch in useEffect, no caching, no auto-refresh
export function ScoreboardBad() {
  const [data, setData] = useState(null)
  useEffect(() => {
    fetch('/api/v1/scoreboard').then(r => r.json()).then(setData)
  }, [])
  // ...
}
```

### 5.3 Component Pattern

```tsx
// ✅ Good — typed props, DaisyUI classes
// src/components/ScoreboardRow.tsx
import type { ScoreboardEntry } from '@/lib/types'

interface ScoreboardRowProps {
  entry: ScoreboardEntry
}

export function ScoreboardRow({ entry }: ScoreboardRowProps) {
  return (
    <tr>
      <td>
        <span className="font-bold">{entry.rank}</span>
      </td>
      <td>
        <div>
          <div className="font-semibold">{entry.display_name}</div>
          <div className="text-sm opacity-50">@{entry.youtube_handle}</div>
        </div>
      </td>
      <td>
        <span className="badge badge-primary">
          {entry.total_points.toLocaleString()} pts
        </span>
      </td>
    </tr>
  )
}
```

### 5.4 API Client

```tsx
// ✅ Good — typed fetcher, error handling
// src/lib/api.ts
const API_BASE = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8008'

export const fetcher = async (path: string) => {
  const res = await fetch(`${API_BASE}${path}`)
  if (!res.ok) {
    const error = new Error('API error')
    error.status = res.status
    throw error
  }
  return res.json()
}
```

### 5.5 Types

```tsx
// ✅ Good — explicit types for API responses
// src/lib/types.ts
export interface ScoreboardEntry {
  rank: number
  display_name: string
  youtube_handle: string
  total_points: number
  donation_count: number
}

export interface ScoreboardResponse {
  data: ScoreboardEntry[]
  meta: {
    total: number
    page: number
    limit: number
    pages: number
    hasNext: boolean
    hasPrev: boolean
  }
}

export interface ViewerPoints {
  youtube_handle: string
  display_name: string
  total_points: number
  donation_count: number
  last_donation: string | null
}
```

---

## 6. DaisyUI / Tailwind Rules

| Rule | Rationale |
|------|----------|
| Use DaisyUI components (`table`, `badge`, `alert`) | Pre-built, accessible, consistent |
| Use Tailwind utilities for spacing/layout | No custom CSS files |
| Use `@theme` in CSS for custom theme | Tailwind 4+ pattern |
| Responsive: mobile-first | `sm:`, `md:`, `lg:` breakpoints |
| Use semantic HTML | `<main>`, `<table>`, `<thead>`, `<tbody>` |
| Use `data-theme="deerngo"` | Custom theme from style guide (028) |

### DaisyUI Theme (from Style Guide)

```css
/* src/app/globals.css */
@import "tailwindcss";
@plugin "daisyui" {
  themes: deerngo --default;
}

@theme {
  --color-primary: #2D5016;    /* Forest Green */
  --color-secondary: #D4A017;  /* Warm Amber */
  --color-accent: #8B4513;     /* Earth Brown */
  --color-base-100: #FEFCE8;   /* Cream */
  --color-base-200: #F5F0E1;   /* Light Beige */
  --color-base-content: #1A1A1A;
}
```

---

## 7. Testing Standards

| Rule | Standard |
|------|----------|
| Framework | Vitest |
| File naming | `*.test.tsx` next to component |
| Component testing | `@testing-library/react` |
| Mocking | `vi.fn()`, `vi.mock()` |
| Coverage | ≥ 80% on components |

```tsx
// ✅ Good — component test
import { render, screen } from '@testing-library/react'
import { ScoreboardRow } from './ScoreboardRow'

test('renders viewer name and points', () => {
  const entry = {
    rank: 1,
    display_name: 'Top Donor',
    youtube_handle: 'topdonor',
    total_points: 1500,
    donation_count: 10,
  }

  render(<ScoreboardRow entry={entry} />)

  expect(screen.getByText('Top Donor')).toBeInTheDocument()
  expect(screen.getByText('1,500 pts')).toBeInTheDocument()
})
```

---

## 8. Universal Rules

| Rule | Rationale |
|------|----------|
| No `any` type | Use `unknown` or proper types |
| No inline styles | Use Tailwind utilities |
| No `useEffect` for data fetching | Use SWR instead |
| Explicit return types on functions | TypeScript enforces |
| Meaningful names | `fetchScoreboard` not `fs` |
| Early returns | Flatten conditionals |
| One component per file | Except tiny related components |

---

## 9. Linting & CI

| Tool | Command | Config |
|------|---------|--------|
| ESLint | `next lint` | `.eslintrc.json` |
| Prettier | `prettier --write .` | `.prettierrc` |
| TypeScript | `tsc --noEmit` | `tsconfig.json` |
| Vitest | `vitest` | `vitest.config.ts` |

All linting **must pass** before merge.

---

## Related Documents

| Document | Path |
|----------|------|
| README | `03_construction/032_FE_README.md` |
| Build Scripts | `03_construction/034_FE_build_scripts.md` |
| Dependency Manifest | `03_construction/036_FE_dependency_manifest.md` |
| Wireframes | `02_design/026_FE_wireframes_lofi.md` |
| Style Guide | `02_design/028_FE_style_guide.md` |
| API Specification | `02_design/022_SHARED_API_specification.md` |

---

> **Template Standard:** Based on SWEBOK v4, Airbnb JS Style Guide, Next.js Docs
> **Usage:** Dev agents should follow these patterns exactly. Standards are enforced in CI.
