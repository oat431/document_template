---
document_type: Coding Standards (Frontend)
version: "0.2"
status: Draft
author: "SA / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "FE"
classification: "Internal"
tags: [coding-standards, typescript, nextjs, react, tailwind, daisyui, frontend, privacy]
standard_ref:
  - SWEBOK v4 — Construction
  - Airbnb JavaScript Style Guide
  - Next.js Documentation
parent_project: "Deerngo Bot — VRM"
---

# Coding Standards — Deerngo Bot Frontend (Next.js)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Repo:** `deerngo-web` (frontend)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> Phase 1 frontend renders only the privacy-safe scoreboard projection. No display names, stable user IDs, raw donor fields, login, or admin behavior belongs in this repository.

## 1. Language Standards

| Aspect | Standard | Tool |
|--------|---------|------|
| TypeScript | Strict mode | `tsc --noEmit` |
| Runtime | Node.js 22 LTS | `.nvmrc` |
| Framework | Next.js 15+ App Router | — |
| CSS | Tailwind CSS 4+ + DaisyUI 5+ | — |
| Formatting | 2 spaces, single quotes, trailing commas | Prettier |
| Linting | ESLint | CI |

## 2. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Components | PascalCase | `ScoreboardRow.tsx` |
| Utils/hooks | camelCase | `useScoreboard.ts` |
| App pages | lowercase | `page.tsx` |
| Functions/variables | camelCase | `fetchScoreboard` |
| Constants | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| Types | PascalCase | `ScoreboardEntry` |

## 3. Project Structure

```text
deerngo-web/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── Scoreboard.tsx
│   │   ├── ScoreboardRow.tsx
│   │   ├── EmptyState.tsx
│   │   ├── ErrorState.tsx
│   │   ├── LoadingState.tsx
│   │   └── Header.tsx
│   └── lib/
│       ├── api.ts
│       ├── types.ts
│       └── constants.ts
├── public/
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── Dockerfile
└── README.md
```

## 4. Public API Types — Allowlist Only

```tsx
export interface ScoreboardEntry {
  rank: number
  youtube_handle: string
  total_points: number
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
```

Do not add `display_name`, `youtube_user_id`, `donation_count`, raw donor fields, registration time, or private member fields to the frontend type model.

## 5. Component Pattern

```tsx
interface ScoreboardRowProps {
  entry: ScoreboardEntry
}

export function ScoreboardRow({ entry }: ScoreboardRowProps) {
  return (
    <tr>
      <td className="text-center">{entry.rank}</td>
      <td className="font-medium">@{entry.youtube_handle}</td>
      <td className="text-right font-semibold">
        {entry.total_points.toLocaleString()} pts
      </td>
    </tr>
  )
}
```

Render only backend-allowlisted fields. Do not reconstruct display names from browser metadata.

## 6. Data Fetching

Use a typed fetcher/SWR with controlled refresh (target ≤60 seconds) and explicit error handling. Do not use raw `any` or log full responses.

```tsx
const API_BASE = process.env.NEXT_PUBLIC_API_URL ?? 'http://localhost:8008'

export async function fetchScoreboard(page = 1, limit = 50): Promise<ScoreboardResponse> {
  const res = await fetch(`${API_BASE}/api/v1/scoreboard?page=${page}&limit=${limit}`)
  if (!res.ok) throw new Error('Scoreboard temporarily unavailable')
  return res.json() as Promise<ScoreboardResponse>
}
```

## 7. UI and Accessibility

- Semantic table/list markup.
- Keyboard-accessible pagination/retry.
- Loading, empty, and error states are always implemented.
- Mobile-first responsive layout.
- No hover-only information.
- Use approved DaisyUI theme and WCAG AA contrast.
- Scoreboard empty text: `No contributors yet. Be the first!`.
- Error text: `Scoreboard temporarily unavailable`.

## 8. Testing Standards

| Rule | Standard |
|------|----------|
| Framework | Vitest + Testing Library |
| Coverage | ≥80% components |
| API tests | Allowlist, filtering, pagination, empty/error |
| Privacy tests | No display-name/user-ID/donor fields rendered |
| Accessibility | Keyboard, semantic roles, visible focus |
| Build gate | lint + type-check + test + build |

## 9. Universal Rules

- No `any` type.
- No secrets in client code.
- No login/authentication in Phase 1.
- No direct database/provider calls from the frontend.
- No client-side reimplementation of member filters or point calculations.
- No real donor data in fixtures.

## Related Documents

| Document | Path |
|----------|------|
| Frontend README | `03_construction/031_FE_README.md` |
| Wireframes | `02_design/026_wireframes_lofi.md` |
| Style Guide | `02_design/028_style_guide.md` |
| API Specification | `02_design/022_API_specification.md` |
| User Stories | `01_requirement/012_user_stories.md` |
| Acceptance Criteria | `01_requirement/013_acceptance_criteria.md` |

---

> **Template Standard:** Based on SWEBOK v4, Airbnb JS Style Guide, Next.js Docs
> **Usage:** Mandatory frontend construction standard for the privacy-safe scoreboard.
---
