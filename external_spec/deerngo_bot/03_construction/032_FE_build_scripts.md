---
document_type: Build Scripts (Frontend)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
repo_type: "FE"
classification: "Internal"
tags: [build-scripts, npm, docker, nextjs, frontend]
standard_ref:
  - SWEBOK v4 — Construction
  - 12-Factor App (Build, release, run)
parent_project: "Deerngo Bot — VRM"
---

# Build Scripts — Deerngo Bot Frontend (Next.js)

> **Project:** Deerngo Bot — VRM | **Repo:** `deerngo-web` (frontend)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. package.json scripts

```json
{
  "scripts": {
    "dev": "next dev --port 3008",
    "build": "next build",
    "start": "next start --port 3008",
    "lint": "next lint",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "docker:build": "docker build -t deerngo-web:latest ."
  }
}
```

---

## 2. Dockerfile (Multi-Stage)

```dockerfile
# ---- Stage 1: Dependencies ----
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# ---- Stage 2: Build ----
FROM node:22-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# ---- Stage 3: Runtime ----
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=build /app/public ./public
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static

EXPOSE 3008
USER nextjs

CMD ["node", "server.js"]
```

---

## 3. Build Commands Summary

| Command | Purpose |
|---------|---------|
| `npm run dev` | Dev server on :3008 (hot reload) |
| `npm run build` | Production build |
| `npm run start` | Start production server |
| `npm run lint` | ESLint |
| `npm run format` | Prettier |
| `npm run type-check` | TypeScript check |
| `npm test` | Vitest |
| `npm run docker:build` | Build Docker image |

---

## Related Documents

| Document | Path |
|----------|------|
| README | `03_construction/032_FE_README.md` |
| Dependency Manifest | `03_construction/036_FE_dependency_manifest.md` |
| Style Guide | `02_design/028_FE_style_guide.md` |

---

> **Template Standard:** Based on SWEBOK v4, 12-Factor App
