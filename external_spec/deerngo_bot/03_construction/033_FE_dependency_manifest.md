---
document_type: Dependency Manifest (Frontend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "FE"
classification: "Internal"
tags: [dependencies, npm, nextjs, frontend, sbom]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Top 10 (A06:2021 — Vulnerable Dependencies)
parent_project: "Deerngo Bot — VRM"
---

# Dependency Manifest — Deerngo Bot Frontend (Next.js)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-web` (frontend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## package.json

```json
{
  "name": "deerngo-web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --port 3008",
    "build": "next build",
    "start": "next start --port 3008",
    "lint": "next lint",
    "test": "vitest"
  },
  "dependencies": {
    "next": "^15.1.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "swr": "^2.3.0"
  },
  "devDependencies": {
    "typescript": "^5.7.0",
    "@types/react": "^19.0.0",
    "@types/node": "^22.0.0",
    "tailwindcss": "^4.0.0",
    "daisyui": "^5.0.0",
    "vitest": "^3.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.5.0"
  }
}
```

---

## Direct Dependencies

| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| `next` | ^15.1.0 | MIT | React framework (SSR/SSG) |
| `react` | ^19.0.0 | MIT | UI library |
| `react-dom` | ^19.0.0 | MIT | React DOM renderer |
| `swr` | ^2.3.0 | MIT | Data fetching + caching |

---

## Dev Dependencies

| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| `typescript` | ^5.7.0 | Apache-2.0 | Type checking |
| `tailwindcss` | ^4.0.0 | MIT | CSS framework |
| `daisyui` | ^5.0.0 | MIT | Component library |
| `vitest` | ^3.0.0 | MIT | Test framework |
| `eslint` | ^9.0.0 | MIT | Linter |
| `prettier` | ^3.5.0 | MIT | Formatter |

---

## Dependency Notes

| Dependency | Notes |
|-----------|-------|
| `next` 15+ | App Router. Server Components by default. |
| `react` 19+ | Server Components support. |
| `swr` | Revalidate on focus, polling for scoreboard refresh. |
| `tailwindcss` 4+ | CSS-first config (`@theme`), no `tailwind.config.js` needed. |
| `daisyui` 5+ | Custom "deerngo" theme from style guide (028). |

---

## Dependency Policy

| Rule | Tool |
|------|------|
| No Critical/High vulnerabilities | `npm audit` |
| Lock file committed | `package-lock.json` |
| Monthly audit | `npm audit` |

---

## Related Documents

| Document | Path |
|----------|------|
| Build Scripts | `03_construction/034_FE_build_scripts.md` |
| Style Guide | `02_design/028_FE_style_guide.md` |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Top 10
