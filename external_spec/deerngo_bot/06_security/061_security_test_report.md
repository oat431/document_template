---
document_type: Security Test Report
version: "0.1"
status: Draft
author: "QA Engineer"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [security-testing, vulnerability, owasp, swebok, vrm, deerngo-bot]
standard_ref:
  - SWEBOK v4 — Testing
  - OWASP Testing Guide v4
  - OWASP Top 10 (2021)
  - ISO/IEC 25010 — Security
---

# Security Test Report

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## 1. Purpose

Reports security assessment for Deerngo Bot Phase 1 — OWASP Top 10 evaluation, vulnerability analysis, and penetration test plan. This is a **pre-code assessment** derived from spec documents (API Specification, ADRs, Database Schema, Architecture). Findings will be validated after code exists.

---

## 2. Security Test Summary

| Field | Detail |
|-------|--------|
| Test Date | 2026-07-30 (spec-based assessment) |
| Test Type | Spec review + threat modeling (pre-code) |
| Tools | Manual review (no code to scan yet) |
| Tester | QA Engineer |
| Scope | Go backend API, webhook endpoint, database, public scoreboard |
| Overall Risk | 🟡 Medium — internal system with limited attack surface, but webhook + public scoreboard need hardening |

---

## 3. Threat Model

### 3.1 Attack Surface

| Surface | Exposure | Entry Point | Risk |
|---------|----------|-------------|:----:|
| **Webhook endpoint** | Public (via Cloudflare Tunnel) | `POST /api/v1/webhooks/easydonate` | 🟠 High |
| **Scoreboard page** | Public (via Cloudflare Tunnel) | `GET /` (Next.js frontend) | 🟡 Medium |
| **Scoreboard API** | Public (via Next.js → Go) | `GET /api/v1/scoreboard` | 🟡 Medium |
| **Subscriber API** | LAN only | `POST /api/v1/subscribers` | 🟢 Low |
| **Points API** | LAN only | `GET /api/v1/points/{handle}` | 🟢 Low |
| **PostgreSQL** | Docker network only | `localhost:5432` | 🟢 Low |

### 3.2 Trust Boundaries

```
Internet ──→ Cloudflare Tunnel ──→ Next.js (:3008) ──→ Go Backend (:8008) ──→ PostgreSQL (:5432)
                                        │                     ↑
                                        │                     │
LAN ──→ streamer.bot (Windows PC) ─────┘                     │
                                                              │
EasyDonate API ──→ Webhook ──→ Go Backend (HMAC verified) ────┘
```

| Boundary | Trust Level | Security Control |
|----------|------------|-----------------|
| Internet → Cloudflare | Untrusted | Cloudflare DDoS, TLS, WAF |
| Cloudflare → Next.js | Semi-trusted | Tunnel auth, CORS |
| Next.js → Go Backend | Trusted | Docker network, no auth (Phase 1) |
| streamer.bot → Go Backend | Trusted | LAN only, no auth (Phase 1) |
| EasyDonate → Go Backend | Semi-trusted | HMAC-SHA256 signature verification |

---

## 4. OWASP Top 10 Assessment (2021)

### 4.1 Assessment Matrix

| # | Category | Status | Notes |
|---|----------|:------:|-------|
| A01 | Broken Access Control | 🟡 Partial | No auth on internal endpoints (acceptable for Phase 1 LAN-only). Webhook uses HMAC. Scoreboard is public read-only. |
| A02 | Cryptographic Failures | ✅ Pass | HMAC-SHA256 for webhook verification. Cloudflare handles TLS. No passwords stored. |
| A03 | Injection | ✅ Pass | sqlx with named parameters — no string concatenation in SQL. `pg_trgm` queries use parameterized inputs. |
| A04 | Insecure Design | 🟡 Partial | No rate limiting on LAN endpoints. Webhook has rate limiting. No request size limits documented. |
| A05 | Security Misconfiguration | 🟡 Partial | No security headers documented. Debug mode must be disabled in production. CORS not yet configured. |
| A06 | Vulnerable Components | ⬜ Pending | No code yet — `govulncheck` scan required after implementation. |
| A07 | Auth Failures | ✅ Pass | No auth system (Phase 1 design decision). HMAC on webhook prevents unauthorized donations. LAN-only for internal endpoints. |
| A08 | Data Integrity Failures | 🟡 Partial | HMAC verifies webhook payload integrity. No CSRF protection needed (no forms). Idempotent sync prevents duplicate donations. |
| A09 | Logging Failures | 🟡 Partial | Request logging middleware exists (coding standards). No audit logging for security events documented. |
| A10 | SSRF | ✅ Pass | No server-side request functionality except outbound calls to YouTube API and EasyDonate API (controlled URLs). |

### 4.2 Summary

| Status | Count |
|--------|:-----:|
| ✅ Pass | 4 |
| 🟡 Partial | 5 |
| ⬜ Pending | 1 |
| 🔴 Fail | 0 |

---

## 5. Security Findings

### SEC-001: No Authentication on Internal API Endpoints

| Field | Detail |
|-------|--------|
| **ID** | SEC-001 |
| **Severity** | 🟡 Medium |
| **Category** | A01 — Broken Access Control |
| **Component** | `POST /api/v1/subscribers`, `GET /api/v1/points/{handle}` |
| **Description** | Internal API endpoints have no authentication. Anyone on the LAN can create subscribers or query points. |
| **Impact** | Low — LAN-only exposure. A compromised device on the same network could inject fake subscribers or query viewer data. |
| **Remediation** | Phase 1: Accept risk (LAN is trusted). Phase 2: Add API key or mTLS for internal endpoints. |
| **Status** | ⬜ Accepted (Phase 1 design decision) |
| **Reference** | ADR-004 (Homelab Deployment), API Spec §2 |

---

### SEC-002: Webhook Replay Attack Vulnerability

| Field | Detail |
|-------|--------|
| **ID** | SEC-002 |
| **Severity** | 🟡 Medium |
| **Category** | A08 — Data Integrity Failures |
| **Component** | `POST /api/v1/webhooks/easydonate` |
| **Description** | HMAC-SHA256 verifies payload integrity but does not prevent replay attacks. An attacker who captures a valid webhook request could replay it to create duplicate donations. The `easydonate_id` idempotency key mitigates this partially — duplicates are skipped. However, if the attacker modifies the `easydonate_id` while keeping a valid signature, the signature check would fail. |
| **Impact** | Low — idempotent sync prevents duplicate donations. Replay of the same payload results in a skip. |
| **Remediation** | Phase 1: Accept risk (idempotency key provides sufficient protection). Phase 2: Add timestamp validation (reject webhooks older than 5 minutes). |
| **Status** | ⬜ Accepted (Phase 1) |
| **Reference** | ADR-012 (HMAC-SHA256), API Spec §4.4 |

---

### SEC-003: HMAC Secret Key Stored in Environment Variable

| Field | Detail |
|-------|--------|
| **ID** | SEC-003 |
| **Severity** | 🟢 Low |
| **Category** | A02 — Cryptographic Failures |
| **Component** | Webhook configuration |
| **Description** | The HMAC secret key is stored in an environment variable. If the Docker container or host is compromised, the key is exposed. |
| **Impact** | Low — homelab environment, single operator. Key exposure would allow fake webhook requests. |
| **Remediation** | Phase 1: Use `.env` file with restricted permissions (`chmod 600`). Phase 2: Use a secrets manager (e.g., Docker secrets, Vault). |
| **Status** | ⬜ Open |
| **Reference** | ADR-012, DEF-S005 (key rotation) |

---

### SEC-004: No Security Headers on API Responses

| Field | Detail |
|-------|--------|
| **ID** | SEC-004 |
| **Severity** | 🟢 Low |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Fiber middleware |
| **Description** | No security headers are documented in the API spec or coding standards: `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`, `Content-Security-Policy`. |
| **Impact** | Low — API is mostly LAN-only. Scoreboard via Cloudflare may inherit some headers. Missing headers could enable MIME sniffing or clickjacking on the scoreboard. |
| **Remediation** | Add Fiber middleware to set security headers. Cloudflare Tunnel provides HSTS automatically. |
| **Status** | ⬜ Open |

---

### SEC-005: No CORS Configuration Documented

| Field | Detail |
|-------|--------|
| **ID** | SEC-005 |
| **Severity** | 🟡 Medium |
| **Category** | A05 — Security Misconfiguration |
| **Component** | Fiber middleware, API Spec §6 |
| **Description** | API Spec §6 mentions "Allow origin from scoreboard frontend URL (Cloudflare hostname)" but no specific CORS configuration is documented. If CORS is too permissive (`*`), any website could call the scoreboard API. |
| **Impact** | Medium — scoreboard is public read-only, so data exposure risk is low. But permissive CORS on internal endpoints would be a concern. |
| **Remediation** | Configure CORS middleware: allow only the scoreboard frontend origin for `/api/v1/scoreboard`. Block cross-origin requests to all other endpoints. |
| **Status** | ⬜ Open |
| **Reference** | API Spec §6, Coding Standards §9 (middleware/cors.go) |

---

### SEC-006: No Request Size Limit on Webhook Endpoint

| Field | Detail |
|-------|--------|
| **ID** | SEC-006 |
| **Severity** | 🟢 Low |
| **Category** | A04 — Insecure Design |
| **Component** | `POST /api/v1/webhooks/easydonate` |
| **Description** | No documented maximum request body size for the webhook endpoint. An attacker could send an extremely large payload to consume memory. |
| **Impact** | Low — Fiber has a default body limit (4MB). But explicitly configuring a smaller limit for the webhook (e.g., 1KB) would be more secure. |
| **Remediation** | Set `app.BodyLimit(1 * 1024 * 1024)` globally or per-route for webhook. |
| **Status** | ⬜ Open |

---

### SEC-007: SQL Injection Protection (Verified)

| Field | Detail |
|-------|--------|
| **ID** | SEC-007 |
| **Severity** | ✅ Info (Positive Finding) |
| **Category** | A03 — Injection |
| **Component** | All repository layer (sqlx) |
| **Description** | sqlx with named parameters (`:field`) and `NamedQueryContext` prevents SQL injection. No raw string concatenation in queries per coding standards. |
| **Impact** | None — properly mitigated. |
| **Status** | ✅ Verified |
| **Reference** | Coding Standards §5.3, ADR-008 |

---

### SEC-008: Public Scoreboard Data Exposure

| Field | Detail |
|-------|-------|
| **ID** | SEC-008 |
| **Severity** | 🟢 Low |
| **Category** | A01 — Broken Access Control |
| **Component** | `GET /api/v1/scoreboard`, Next.js frontend |
| **Description** | The scoreboard publicly exposes YouTube handles and display names. This is intentional (OBJ-04) but should be noted — viewer PII is visible to anyone. |
| **Impact** | Low — YouTube handles are already public. No email, IP, or financial data exposed. |
| **Remediation** | Document in privacy notice. Consider allowing viewers to opt out of scoreboard. |
| **Status** | ⬜ Accepted (by design) |

---

## 6. Penetration Test Plan

> To be executed after code exists. Scope and test cases defined here.

### 6.1 Test Scope

| Test | Target | Method | Priority |
|------|--------|--------|:--------:|
| **Webhook signature bypass** | `POST /api/v1/webhooks/easydonate` | Send requests without signature, with wrong signature, with empty signature | 🔴 |
| **Webhook replay** | `POST /api/v1/webhooks/easydonate` | Replay a valid webhook request | 🔴 |
| **SQL injection** | All endpoints with input | Fuzz `youtube_handle`, `donor_name`, `limit`, `page` with SQL payloads | 🔴 |
| **Input validation** | `POST /api/v1/subscribers` | Send oversized handles (>100 chars), special chars, unicode | 🟡 |
| **Rate limiting** | All endpoints | Exceed 100 req/min, verify 429 response | 🟡 |
| **CORS** | All endpoints | Send cross-origin requests from unauthorized origins | 🟡 |
| **Error leakage** | All endpoints | Trigger 500 errors, check for stack traces in response | 🟡 |
| **Path traversal** | Scoreboard frontend | Attempt directory traversal in URL paths | 🟢 |
| **Header injection** | All endpoints | Inject newlines in header values | 🟢 |

### 6.2 Tools

| Tool | Purpose |
|------|---------|
| `curl` + manual scripts | Webhook signature tests, replay tests |
| `sqlmap` | Automated SQL injection scanning |
| `OWASP ZAP` | Automated vulnerability scan (scoreboard) |
| `govulncheck` | Go dependency vulnerability scan |
| `golangci-lint` | Static analysis (security rules) |

---

## 7. Security Recommendations

| # | Recommendation | Priority | Status | Owner |
|---|---------------|:--------:|:------:|:-----:|
| 1 | Configure CORS middleware with explicit allowed origins | 🟡 | ⬜ Open | Dev |
| 2 | Add security headers middleware (X-Content-Type-Options, X-Frame-Options) | 🟢 | ⬜ Open | Dev |
| 3 | Set explicit request body size limit on webhook endpoint | 🟢 | ⬜ Open | Dev |
| 4 | Restrict `.env` file permissions (chmod 600) | 🟢 | ⬜ Open | DevOps |
| 5 | Run `govulncheck` after dependencies are installed | 🔴 | ⬜ Pending | Dev |
| 6 | Add webhook timestamp validation for replay protection (Phase 2) | 🟢 | Deferred | — |
| 7 | Add API key authentication for internal endpoints (Phase 2) | 🟢 | Deferred | — |
| 8 | Document HMAC key rotation procedure | 🟡 | ⬜ Open | Dev+DevOps |

---

## 8. Compliance Summary

| Standard | Status | Notes |
|----------|:------:|-------|
| OWASP Top 10 (2021) | 🟡 4/10 Pass, 5/10 Partial, 1 Pending | See §4 |
| ISO/IEC 25010 Security | 🟡 Acceptable for Phase 1 | Internal system, limited exposure |
| OWASP API Security Top 10 | 🟡 Partial | API1 (Broken Auth) accepted for Phase 1 |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[022_API_specification]] | API contracts being assessed |
| [[021_architecture_decision_records]] | ADR-012 (HMAC), ADR-011 (Cloudflare) |
| [[023_database_schema_DDL]] | SQL injection surface (parameterized queries) |
| [[041_test_plan]] | Security testing integrated into test plan |
| [[043_defect_report]] | DEF-S005 (HMAC key rotation) |
| [[062_coding_standards_security]] | Security coding rules |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Testing Guide v4
> **Usage:** This is a *pre-code assessment*. Security findings must be validated after code exists. Run automated scans in CI/CD.
