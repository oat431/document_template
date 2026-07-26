---
document_type: Test Cases
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Guard"
project_id: "flowero-guard"
classification: "Internal"
tags: [test-cases, oauth2, oidc, keycloak, token, sso, rbac, panomete]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
  - OAuth 2.0 (RFC 6749)
  - OpenID Connect Core 1.0
---

# Test Cases — Flowero Guard

> **Project:** Flowero Guard
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Detailed test cases for Flowero Guard covering OAuth2 flows (authorization code, client credentials, refresh token), user management, token validation, SSO, RBAC, and deployment verification. All test cases trace to user stories (US-001 through US-006) and acceptance criteria (AC-G001 through AC-G006).

## 2. Test Case Index

| Module | Total | Automated | Manual | Status |
|--------|-------|----------|--------|--------|
| Deployment & Configuration | 8 | 8 | 0 | ✅ |
| Authorization Code Flow | 7 | 5 | 2 | ✅ |
| Client Credentials Flow | 5 | 5 | 0 | ✅ |
| Token Refresh | 4 | 4 | 0 | ✅ |
| Token Introspection | 4 | 4 | 0 | ✅ |
| Single Sign-On (SSO) | 5 | 2 | 3 | ✅ |
| Role-Based Access Control | 5 | 3 | 2 | ✅ |
| User Management | 4 | 2 | 2 | ✅ |
| JWKS & Discovery | 4 | 4 | 0 | ✅ |
| Error Handling | 4 | 4 | 0 | ✅ |
| **Total** | **50** | **41** | **9** | |

## 3. Test Cases — Deployment & Configuration

### TC-G001: Guard Container Healthy Startup

| Field | Value |
|-------|-------|
| **ID** | TC-G001 |
| **Title** | Verify Guard starts healthy within 60 seconds |
| **Priority** | 🔴 Critical |
| **Type** | Deployment |
| **Automated** | Yes (CI/CD smoke test) |
| **Requirement** | US-001, AC-G001a |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | PostgreSQL 18 running with `keycloak` database and role created |
| 2 | `panomete-realm.json` exists at correct mount path |
| 3 | `.env` file contains valid `KC_DB_PASSWORD` and admin credentials |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Execute `docker compose up -d flowero-guard` | Container starts |
| 2 | Wait up to 60 seconds | Logs show "Keycloak 26.7.0 on JVM started in..." |
| 3 | `curl -sf http://localhost:8001/health/ready` | Returns HTTP 200 |
| 4 | `docker ps \| grep flowero-guard` | Shows "Up" status |

---

### TC-G002: Realm Pre-Configuration via JSON Import

| Field | Value |
|-------|-------|
| **ID** | TC-G002 |
| **Title** | Verify panomete realm imported with roles from JSON |
| **Priority** | 🔴 Critical |
| **Type** | Configuration |
| **Automated** | Yes |
| **Requirement** | US-001, AC-G001b, AC-G001c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard container is running and healthy |
| 2 | `panomete-realm.json` contains roles: admin, user, viewer |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login to Admin Console at `auth.panomete.com/admin` | Admin console loads |
| 2 | Navigate to Realms list | `panomete` realm is present |
| 3 | Open panomete realm → Realm Roles | Roles `admin`, `user`, `viewer` exist |
| 4 | Check realm settings → Tokens | Access Token Lifespan = 5 minutes |
| 5 | Check realm settings → Sessions | SSO Session Idle Timeout = 30 minutes |

---

### TC-G003: Data Persistence Across Restart

| Field | Value |
|-------|-------|
| **ID** | TC-G003 |
| **Title** | Verify all data persists across container restart |
| **Priority** | 🔴 Critical |
| **Type** | Configuration |
| **Automated** | Yes |
| **Requirement** | US-001, AC-G001d |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running with panomete realm configured |
| 2 | At least one user and one OAuth2 client created |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Note existing users and clients in Admin Console | Count recorded |
| 2 | `docker compose down flowero-guard` | Container stops |
| 3 | `docker compose up -d flowero-guard` | Container restarts |
| 4 | Wait for healthy status | Health endpoint returns 200 |
| 5 | Login to Admin Console, verify users and clients | All previously created entities intact |

---

### TC-G004: OIDC Discovery Endpoint

| Field | Value |
|-------|-------|
| **ID** | TC-G004 |
| **Title** | Verify OIDC discovery returns correct configuration |
| **Priority** | 🔴 Critical |
| **Type** | Configuration |
| **Automated** | Yes |
| **Requirement** | OBJ-GUARD-02 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running and healthy |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `curl -sf https://auth.panomete.com/realms/panomete/.well-known/openid-configuration` | Returns JSON 200 |
| 2 | Check `issuer` field | `"https://auth.panomete.com/realms/panomete"` |
| 3 | Check `authorization_endpoint` | Points to `/protocol/openid-connect/auth` |
| 4 | Check `token_endpoint` | Points to `/protocol/openid-connect/token` |
| 5 | Check `jwks_uri` | Points to `/protocol/openid-connect/certs` |
| 6 | Check `introspection_endpoint` | Points to `/protocol/openid-connect/token/introspect` |

---

### TC-G005: JWKS Endpoint Returns Valid RSA Keys

| Field | Value |
|-------|-------|
| **ID** | TC-G005 |
| **Title** | Verify JWKS endpoint returns valid RSA public keys |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | OBJ-GUARD-02 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running and healthy |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `curl -sf https://auth.panomete.com/realms/panomete/protocol/openid-connect/certs` | Returns JSON 200 |
| 2 | Parse response, check `keys` array | Array is non-empty |
| 3 | Check first key's `kty` field | Value is `"RSA"` |
| 4 | Check first key's `alg` field | Value is `"RS256"` |
| 5 | Check first key's `use` field | Value is `"sig"` |
| 6 | Verify no auth required (public endpoint) | 200 returned without any credentials |

---

### TC-G006: Admin Console Accessible

| Field | Value |
|-------|-------|
| **ID** | TC-G006 |
| **Title** | Verify admin console is accessible with configured credentials |
| **Priority** | 🔴 Critical |
| **Type** | Deployment |
| **Automated** | No (manual browser login) |
| **Requirement** | US-001, AC-G001f |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running and healthy |
| 2 | Admin credentials set in `.env` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to `https://auth.panomete.com/admin/` | Redirects to login page (302) |
| 2 | Enter admin username and password from `.env` | Login succeeds |
| 3 | Verify admin dashboard loads | Dashboard shows realm management options |
| 4 | Switch to `panomete` realm | Realm dashboard shows active users/sessions |

---

### TC-G007: Failed Startup — Missing Database

| Field | Value |
|-------|-------|
| **ID** | TC-G007 |
| **Title** | Verify clear error when PostgreSQL is unavailable |
| **Priority** | 🟡 High |
| **Type** | Configuration |
| **Automated** | Yes |
| **Requirement** | US-001, AC-G001e |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | PostgreSQL container stopped |
| 2 | Guard compose file configured |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Stop PostgreSQL: `docker stop local-postgres` | PostgreSQL stops |
| 2 | Start Guard: `docker compose up flowero-guard` | Guard attempts to start |
| 3 | Check logs: `docker logs flowero-guard 2>&1 \| tail -20` | Clear error about database connectivity; retries logged |
| 4 | Verify container does NOT silently succeed | Health endpoint returns non-200 |
| 5 | Restart PostgreSQL: `docker start local-postgres` | PostgreSQL running |
| 6 | Guard eventually connects | Health returns 200 within 120s |

---

### TC-G008: Nginx Proxy Routing

| Field | Value |
|-------|-------|
| **ID** | TC-G008 |
| **Title** | Verify Nginx correctly proxies auth.panomete.com to Guard |
| **Priority** | 🔴 Critical |
| **Type** | Deployment |
| **Automated** | Yes |
| **Requirement** | OBJ-GUARD-01 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running on port 8001 |
| 2 | Nginx configured with auth.panomete.com server block |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `curl -sf -H 'Host: auth.panomete.com' http://localhost:8001/health/ready` | 200 OK (direct) |
| 2 | `curl -sf https://auth.panomete.com/health/ready` | 200 OK (via Nginx) |
| 3 | `curl -sf -I https://auth.panomete.com/admin/` | 302 redirect to login (proper proxy) |
| 4 | Check response headers for X-Forwarded-Proto | `https` (hardcoded in Nginx) |

---

## 4. Test Cases — Authorization Code Flow

### TC-G009: Successful Authorization Code Flow

| Field | Value |
|-------|-------|
| **ID** | TC-G009 |
| **Title** | Complete authorization code flow with token exchange |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (curl-based) |
| **Requirement** | US-003, AC-G003b |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | OAuth2 client `cute-gufo` registered with confidential access type |
| 2 | Client has standard flow enabled |
| 3 | Valid redirect URI configured: `https://blog.panomete.com/*` |
| 4 | Test user `alice` with role `admin` exists |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to `GET /realms/panomete/protocol/openid-connect/auth?response_type=code&client_id=cute-gufo&redirect_uri=https://blog.panomete.com/callback&scope=openid+profile+email&state=test123` | Keycloak login page displayed |
| 2 | Enter credentials for user `alice` | Login succeeds |
| 3 | Verify redirect to `https://blog.panomete.com/callback?code=<auth_code>&state=test123` | Authorization code received; state matches |
| 4 | `POST /realms/panomete/protocol/openid-connect/token` with `grant_type=authorization_code&code=<auth_code>&client_id=cute-gufo&client_secret=<secret>&redirect_uri=https://blog.panomete.com/callback` | Returns 200 with JSON |
| 5 | Parse response | Contains `access_token`, `refresh_token`, `expires_in: 300`, `token_type: "Bearer"` |
| 6 | Decode access_token (JWT) | `sub` = alice's UUID, `realm_access.roles` includes `admin`, `iss` = `https://auth.panomete.com/realms/panomete` |

---

### TC-G010: Authorization Code Flow — Invalid Client Secret

| Field | Value |
|-------|-------|
| **ID** | TC-G010 |
| **Title** | Token exchange fails with wrong client secret |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-002, AC-G002c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Valid authorization code obtained |
| 2 | Client `cute-gufo` registered |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain valid auth code (steps 1-3 from TC-G009) | Auth code received |
| 2 | `POST /token` with correct code but wrong `client_secret=wrong_secret` | Returns 401 |
| 3 | Parse error response | `{"error": "invalid_client"}` |

---

### TC-G011: Authorization Code Flow — Invalid Redirect URI

| Field | Value |
|-------|-------|
| **ID** | TC-G011 |
| **Title** | Authorization request rejected for unregistered redirect URI |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-002 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client `cute-gufo` registered with redirect URI `https://blog.panomete.com/*` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /auth?response_type=code&client_id=cute-gufo&redirect_uri=https://evil.com/callback&scope=openid` | Returns 400 or error page |
| 2 | Verify error message | "Invalid redirect_uri" or similar |

---

### TC-G012: Authorization Code Flow — Failed Login

| Field | Value |
|-------|-------|
| **ID** | TC-G012 |
| **Title** | Login page shows error for invalid credentials |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | No (browser interaction) |
| **Requirement** | US-003, AC-G003c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Authorization endpoint initiated |
| 2 | Login page displayed |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Enter valid username `alice` but wrong password | Credentials submitted |
| 2 | Click Sign In | Error message: "Invalid username or password" |
| 3 | Verify no token issued | No redirect with code occurs |
| 4 | Verify login page remains displayed | User can retry |

---

### TC-G013: Authorization Code — Missing State Parameter

| Field | Value |
|-------|-------|
| **ID** | TC-G013 |
| **Title** | Authorization flow works without state (but state recommended) |
| **Priority** | 🟢 Medium |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client registered, user exists |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Initiate auth flow without `state` parameter | Redirect to login page |
| 2 | Authenticate successfully | Redirect back without state in callback URL |
| 3 | Exchange code for token | Token returned successfully |

---

### TC-G014: Authorization Code — Expired Code

| Field | Value |
|-------|-------|
| **ID** | TC-G014 |
| **Title** | Expired authorization code returns invalid_grant |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Valid authorization code obtained |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain auth code | Code received |
| 2 | Wait 60+ seconds (Keycloak auth code expiry ~60s) | Time passes |
| 3 | `POST /token` with expired code | Returns 400 |
| 4 | Parse error response | `{"error": "invalid_grant"}` |

---

### TC-G015: Authorization Code — Scope Validation

| Field | Value |
|-------|-------|
| **ID** | TC-G015 |
| **Title** | Token includes only requested and granted scopes |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client configured with `openid profile email` scopes |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Complete auth code flow with `scope=openid profile email` | Token received |
| 2 | Decode access token, check `scope` claim | Contains requested scopes |
| 3 | Check ID token contains `email` claim | Present when `email` scope granted |
| 4 | Check `preferred_username` present | Present when `profile` scope granted |

---

## 5. Test Cases — Client Credentials Flow

### TC-G016: Client Credentials — Successful Token Request

| Field | Value |
|-------|-------|
| **ID** | TC-G016 |
| **Title** | Service obtains token via client credentials grant |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-005, AC-G005a |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client `fluffy-mouton` registered with service accounts enabled |
| 2 | Client secret known |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /realms/panomete/protocol/openid-connect/token` with `grant_type=client_credentials&client_id=fluffy-mouton&client_secret=<secret>` | Returns 200 |
| 2 | Parse response | Contains `access_token`, `token_type: "Bearer"`, `expires_in: 300` |
| 3 | Decode JWT | `sub` = `service-account-fluffy-mouton`, `azp` = `fluffy-mouton` |
| 4 | Check `aud` claim | Contains `account` |

---

### TC-G017: Client Credentials — Invalid Secret

| Field | Value |
|-------|-------|
| **ID** | TC-G017 |
| **Title** | Client credentials rejected with wrong secret |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-002, AC-G002c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client `fluffy-mouton` registered |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /token` with `grant_type=client_credentials&client_id=fluffy-mouton&client_secret=WRONG` | Returns 401 |
| 2 | Parse response | `{"error": "invalid_client", "error_description": "..."}` |

---

### TC-G018: Client Credentials — Service Account Roles in JWT

| Field | Value |
|-------|-------|
| **ID** | TC-G018 |
| **Title** | Client credentials JWT contains service account roles |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-005, AC-G005a |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client `fluffy-mouton` with service account roles assigned |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain client credentials token | JWT received |
| 2 | Decode JWT payload | `realm_access.roles` present |
| 3 | Check roles contain assigned service roles | Client-specific roles present |

---

### TC-G019: Client Credentials — No Refresh Token Issued

| Field | Value |
|-------|-------|
| **ID** | TC-G019 |
| **Title** | Client credentials grant does not return refresh token |
| **Priority** | 🟢 Medium |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | RFC 6749 §4.4.3 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client registered with service accounts enabled |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain client credentials token | JWT received |
| 2 | Check response for `refresh_token` field | Field is absent (not returned for client_credentials) |

---

### TC-G020: Client Credentials — Unauthorized Grant Type

| Field | Value |
|-------|-------|
| **ID** | TC-G020 |
| **Title** | Client without service accounts enabled cannot use client_credentials |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-005, AC-G005d |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Client with service accounts DISABLED |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /token` with `grant_type=client_credentials` using disabled client | Returns 403 |
| 2 | Parse response | `{"error": "unauthorized_client"}` |

---

## 6. Test Cases — Token Refresh

### TC-G021: Successful Token Refresh

| Field | Value |
|-------|-------|
| **ID** | TC-G021 |
| **Title** | Expired access token renewed via refresh token |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003, AC-G003e |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User `alice` has completed auth code flow |
| 2 | `refresh_token` obtained (30 min expiry) |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Wait for access token to expire (5 minutes) OR use token immediately | Access token expired |
| 2 | `POST /token` with `grant_type=refresh_token&refresh_token=<token>&client_id=cute-gufo&client_secret=<secret>` | Returns 200 |
| 3 | Parse response | New `access_token` and new `refresh_token` returned |
| 4 | Verify new access token is valid | JWT decodes, `exp` is 5 minutes from now |
| 5 | Verify `expires_in: 300` | Correct lifespan |

---

### TC-G022: Refresh Token — Expired Refresh Token

| Field | Value |
|-------|-------|
| **ID** | TC-G022 |
| **Title** | Expired refresh token returns invalid_grant |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Refresh token obtained |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain refresh token | Token received |
| 2 | Wait 31+ minutes (SSO session idle timeout) | Token expires |
| 3 | `POST /token` with expired refresh token | Returns 400 |
| 4 | Parse response | `{"error": "invalid_grant"}` |

---

### TC-G023: Refresh Token — Revoked After Logout

| Field | Value |
|-------|-------|
| **ID** | TC-G023 |
| **Title** | Refresh token invalidated after user logout |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003, AC-G003f |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User logged in with active refresh token |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Obtain access + refresh token via auth code flow | Tokens received |
| 2 | Logout: `GET /realms/panomete/protocol/openid-connect/logout?id_token_hint=<id_token>` | Session terminated |
| 3 | Attempt refresh with the old refresh token | Returns 400 |
| 4 | Parse response | `{"error": "invalid_grant"}` |

---

### TC-G024: Refresh Token Rotation

| Field | Value |
|-------|-------|
| **ID** | TC-G024 |
| **Title** | Each refresh returns a new refresh token (rotation) |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User authenticated with tokens |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Use refresh token to get new tokens | New refresh_token received |
| 2 | Note the new refresh_token value | Different from original |
| 3 | Try to use the OLD refresh token | Returns 400 `invalid_grant` (rotated/invalidated) |
| 4 | Use the NEW refresh token | Returns 200 with fresh tokens |

---

## 7. Test Cases — Token Introspection

### TC-G025: Introspect Valid Token

| Field | Value |
|-------|-------|
| **ID** | TC-G025 |
| **Title** | Introspection returns active=true for valid token |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-006, AC-G006a |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Valid access token obtained |
| 2 | Client credentials for introspection caller available |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /realms/panomete/protocol/openid-connect/token/introspect` with `Authorization: Basic base64(client_id:secret)` and `token=<access_token>` | Returns 200 |
| 2 | Parse response | `{"active": true, "sub": "...", "realm_access": {"roles": [...]}, ...}` |
| 3 | Verify `sub` matches user UUID | Correct user identified |
| 4 | Verify roles present | User's roles in response |

---

### TC-G026: Introspect Expired Token

| Field | Value |
|-------|-------|
| **ID** | TC-G026 |
| **Title** | Introspection returns active=false for expired token |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-006, AC-G006b |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Expired access token available |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Wait for token to expire (>5 min) | Token expired |
| 2 | Call introspection endpoint with expired token | Returns 200 |
| 3 | Parse response | `{"active": false}` |

---

### TC-G027: Introspect Without Authentication

| Field | Value |
|-------|-------|
| **ID** | TC-G027 |
| **Title** | Introspection endpoint rejects unauthenticated requests |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-006, AC-G006c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Any access token available |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /token/introspect` without `Authorization` header | Returns 401 |
| 2 | Verify response | Unauthorized error returned |

---

### TC-G028: Introspect JWT Token

| Field | Value |
|-------|-------|
| **ID** | TC-G028 |
| **Title** | Introspection works for JWT (not just opaque tokens) |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-006, AC-G006d |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Valid JWT access token (default Keycloak output) |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Call introspection with JWT as token parameter | Returns 200 |
| 2 | Parse response | `{"active": true, ...claims}` with all parsed claims |

---

## 8. Test Cases — Single Sign-On (SSO)

### TC-G029: SSO Across Two Services

| Field | Value |
|-------|-------|
| **ID** | TC-G029 |
| **Title** | User logged into Service A is not re-prompted at Service B |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | No (browser) |
| **Requirement** | US-003, AC-G003d |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Two OAuth2 clients registered (e.g., `cute-gufo` for blog, `fluffy-mouton` for URL shortener) |
| 2 | User `alice` exists |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to Service A (blog) without auth | Redirected to `auth.panomete.com` login |
| 2 | Login as `alice` | Redirected back to Service A with token |
| 3 | Navigate to Service B (URL shortener) | NOT prompted to login; receives token via SSO |
| 4 | Verify Service B session | Valid token present, user identified as `alice` |

---

### TC-G030: Global Logout Terminates All Sessions

| Field | Value |
|-------|-------|
| **ID** | TC-G030 |
| **Title** | Logout from any service invalidates all sessions |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | No (browser) |
| **Requirement** | US-003, AC-G003f |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User logged into Service A and Service B via SSO |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Confirm both services have valid sessions | Both accessible |
| 2 | Click Logout on Service A | Keycloak end_session endpoint called |
| 3 | Attempt to access Service A | Redirected to login page |
| 4 | Attempt to access Service B | Redirected to login page |
| 5 | Verify refresh tokens for both services fail | `invalid_grant` returned |

---

### TC-G031: SSO Session Idle Timeout

| Field | Value |
|-------|-------|
| **ID** | TC-G031 |
| **Title** | Idle session expires after 30 minutes |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | No (timing-dependent) |
| **Requirement** | US-003, AC-G003g |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User logged in with active SSO session |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login and verify access | Access granted |
| 2 | Remain idle for 31 minutes (SSO Session Idle Timeout = 1800s) | Session expires |
| 3 | Access protected resource | Redirected to login page |

---

### TC-G032: Redirect to Login on 401

| Field | Value |
|-------|-------|
| **ID** | TC-G032 |
| **Title** | Unauthenticated request to protected API triggers login redirect |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | No (browser flow) |
| **Requirement** | US-003, AC-G003a |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User not authenticated (no token/session) |
| 2 | Gate running with JWT validation |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET https://api.panomete.com/api/blog/posts` without Authorization header | Gate returns 401 |
| 2 | Frontend detects 401 and redirects to `https://auth.panomete.com` | Keycloak login page displayed |

---

### TC-G033: Access Token Expiry Enforced by Gate

| Field | Value |
|-------|-------|
| **ID** | TC-G033 |
| **Title** | Expired access token rejected by Gate (local JWT validation) |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-003, AC-G003c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Gate running and caching JWKS |
| 2 | Expired JWT (exp in the past) |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Wait for access token to expire (>5 min) | Token expired |
| 2 | `GET /api/blog/posts` with expired `Authorization: Bearer <token>` | Gate returns 401 |
| 3 | Client uses refresh token to get new access token | New token received |
| 4 | `GET /api/blog/posts` with new token | Returns 200 |

---

## 9. Test Cases — Role-Based Access Control

### TC-G034: Admin Role Grants Access to Admin Endpoints

| Field | Value |
|-------|-------|
| **ID** | TC-G034 |
| **Title** | User with admin role can access admin-only endpoints |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | US-004, AC-G004a |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User `alice` with role `admin` |
| 2 | Service endpoint with `@PreAuthorize("hasRole('admin')")` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as `alice`, obtain JWT | Token with `admin` role |
| 2 | `GET /api/admin/users` with `Authorization: Bearer *** | Returns 200 |

---

### TC-G035: User Role Denied Admin Access

| Field | Value |
|-------|-------|
| **ID** | TC-G035 |
| **Title** | User with only 'user' role cannot access admin endpoints |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | US-004, AC-G004b |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User `bob` with role `user` only |
| 2 | Admin-only endpoint protected |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as `bob`, obtain JWT | Token with `user` role only |
| 2 | `GET /api/admin/users` with bob's token | Returns 403 |
| 3 | Parse response | `{"error": "Access Denied", "required_role": "admin"}` |

---

### TC-G036: JWT Contains Correct Roles Claim

| Field | Value |
|-------|-------|
| **ID** | TC-G036 |
| **Title** | JWT realm_access.roles claim matches user assignments |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes |
| **Requirement** | US-004, AC-G004c |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User `alice` assigned roles `admin` and `user` |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as `alice`, obtain access token | JWT received |
| 2 | Decode JWT payload (base64 decode middle section) | Claims visible |
| 3 | Check `realm_access.roles` | Contains `["admin", "user"]` |

---

### TC-G037: Viewer Role — Read-Only Access

| Field | Value |
|-------|-------|
| **ID** | TC-G037 |
| **Title** | Viewer role can read but not write |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | No |
| **Requirement** | US-004 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User with `viewer` role |
| 2 | Service with read (200) and write (403) endpoints |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as viewer, obtain JWT | Token with `viewer` role |
| 2 | `GET /api/blog/posts` | Returns 200 (read allowed) |
| 3 | `POST /api/blog/posts` with body | Returns 403 (write denied) |

---

### TC-G038: No Roles — Public Endpoint Access

| Field | Value |
|-------|-------|
| **ID** | TC-G038 |
| **Title** | User with no roles can access public (unprotected) endpoints |
| **Priority** | 🟢 Medium |
| **Type** | System |
| **Automated** | Yes |
| **Requirement** | US-004, AC-G004e |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User with no realm roles |
| 2 | Public endpoint (no `@PreAuthorize`) |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Login as user with no roles | JWT with empty roles array |
| 2 | Access public endpoint | Returns 200 |

---

## 10. Test Cases — User Management

### TC-G039: Admin Creates New User

| Field | Value |
|-------|-------|
| **ID** | TC-G039 |
| **Title** | Admin creates user via Admin Console |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | No (Admin Console) |
| **Requirement** | US-001, ADR-G004 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Admin logged into Admin Console |
| 2 | Panomete realm selected |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to Users → Add user | User creation form displayed |
| 2 | Fill: username=testuser, email=test@panomete.com | Fields populated |
| 3 | Save user | User created successfully |
| 4 | Set password (Credentials tab, temporary=true) | Password set |
| 5 | Assign role `user` (Role Mappings tab) | Role assigned |
| 6 | Login as testuser | Prompted to change password on first login |

---

### TC-G040: Self-Registration Disabled

| Field | Value |
|-------|-------|
| **ID** | TC-G040 |
| **Title** | User self-registration is not available |
| **Priority** | 🔴 Critical |
| **Type** | Configuration |
| **Automated** | Yes |
| **Requirement** | ADR-G004 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to login page | Login form displayed |
| 2 | Look for "Register" link | No registration link present |
| 3 | Check realm settings: `registrationAllowed` | Value is `false` |

---

### TC-G041: Brute Force Protection — Account Lockout

| Field | Value |
|-------|-------|
| **ID** | TC-G041 |
| **Title** | Account locked after repeated failed login attempts |
| **Priority** | 🔴 Critical |
| **Type** | Security |
| **Automated** | Yes |
| **Requirement** | OBJ-GUARD-03, realm config |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Test user `locktest` exists |
| 2 | `bruteForceProtected: true` in realm config |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Attempt login with wrong password (attempt 1) | "Invalid username or password" |
| 2 | Repeat attempts 2-30 (or configured threshold) | Same error |
| 3 | Attempt login with CORRECT password after threshold | Account temporarily disabled; login still fails |
| 4 | Check Admin Console → Users → locktest → Events | Failed login events logged |

---

### TC-G042: Admin Resets User Password

| Field | Value |
|-------|-------|
| **ID** | TC-G042 |
| **Title** | Admin can reset user password via Admin Console |
| **Priority** | 🟡 High |
| **Type** | System |
| **Automated** | No |
| **Requirement** | US-004 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | User exists, admin logged in |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to Users → select user → Credentials | Credentials tab shown |
| 2 | Set new password, set Temporary = On | Password updated |
| 3 | User logs in with new password | Prompted to change password (temporary) |
| 4 | User sets new password | Login succeeds with new password |

---

## 11. Test Cases — Error Handling

### TC-G043: Invalid Grant Type

| Field | Value |
|-------|-------|
| **ID** | TC-G043 |
| **Title** | Unsupported grant type returns error |
| **Priority** | 🟡 High |
| **Type** | Error Handling |
| **Automated** | Yes |
| **Requirement** | RFC 6749 §5.2 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /token` with `grant_type=implicit` | Returns 400 |
| 2 | Parse response | `{"error": "unsupported_grant_type"}` |

---

### TC-G044: Missing Required Parameters

| Field | Value |
|-------|-------|
| **ID** | TC-G044 |
| **Title** | Token request with missing grant_type returns error |
| **Priority** | 🟡 High |
| **Type** | Error Handling |
| **Automated** | Yes |
| **Requirement** | RFC 6749 §5.2 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /token` without `grant_type` parameter | Returns 400 |
| 2 | Parse response | `{"error": "invalid_request"}` |

---

### TC-G045: Unknown Client ID

| Field | Value |
|-------|-------|
| **ID** | TC-G045 |
| **Title** | Token request with unknown client_id returns error |
| **Priority** | 🟡 High |
| **Type** | Error Handling |
| **Automated** | Yes |
| **Requirement** | RFC 6749 §5.2 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Guard running |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `POST /token` with `client_id=nonexistent-client` | Returns 401 |
| 2 | Parse response | `{"error": "invalid_client"}` |

---

### TC-G046: Malformed Authorization Header

| Field | Value |
|-------|-------|
| **ID** | TC-G046 |
| **Title** | Gate rejects malformed Authorization header |
| **Priority** | 🟡 High |
| **Type** | Error Handling |
| **Automated** | Yes |
| **Requirement** | OBJ-GUARD-02 |

**Preconditions:**

| # | Condition |
|---|----------|
| 1 | Gate running |

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | `GET /api/blog/posts` with `Authorization: Bearer not.a.valid.jwt` | Gate returns 401 |
| 2 | `GET /api/blog/posts` with `Authorization: Basic dGVzdA==` | Gate returns 401 (Bearer expected) |
| 3 | `GET /api/blog/posts` with no Authorization header | Gate returns 401 |

---

## 12. Test Execution Summary

| Sprint | Executed | Passed | Failed | Blocked | Pass Rate |
|--------|---------|--------|--------|---------|----------|
| M1 — Core Infrastructure | 8 | — | — | — | — |
| M2 — End-to-End Auth | 16 | — | — | — | — |
| M3 — Production Hardening | 26 | — | — | — | — |
| **Total** | **50** | **—** | **—** | **—** | **—** |

> **Note:** Execution results to be populated during test runs.

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Plan governing these test cases |
| [[043_defect_report]] | Defects found during execution |
| [[044_regression_test_suite]] | Regression subset of these cases |
| [[013_acceptance_criteria]] | BDD criteria these cases verify |
| [[022_API_specification]] | Endpoints under test |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119
> **Usage:** Every user story has ≥1 test case. Every test case traces to a requirement. Keep in sync with realm changes.
