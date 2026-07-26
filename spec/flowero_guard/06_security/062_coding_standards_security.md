---
document_type: Coding Standards / Security
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Guard"
project_id: "flowero-guard"
classification: "Internal"
tags: [coding-standards, security, keycloak, oauth, configuration]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Top 10
  - OAuth 2.0 Security Best Current Practice
---

# Coding Standards — Security (Flowero Guard)

> **Project:** Flowero Guard (Keycloak IAM)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Security configuration standards for Flowero Guard. Covers Keycloak realm configuration, OAuth2/OIDC settings, token management, and operational security.

## 2. Realm Configuration

| Rule | Standard | Example |
|------|---------|---------|
| **Realm Name** | Use consistent realm name | `panomete` |
| **Realm Export** | Use partial export; exclude secrets | `Partial Export` with client scopes |
| **Version Control** | Commit realm JSON to git | `panomete-realm.json` |
| **CI Validation** | Validate JSON in CI pipeline | `jq . panomete-realm.json` |
| **Secret Scanning** | Scan for secrets before commit | `gitleaks` or `trufflehog` |

**Example (CI validation):**
```yaml
- name: Validate Realm JSON
  run: |
    jq . flowero-guard/panomete-realm.json > /dev/null
    REALM=$(jq -r '.realm' flowero-guard/panomete-realm.json)
    [ "$REALM" = "panomete" ] || exit 1
```

## 3. OAuth2 Client Configuration

| Rule | Standard | Example |
|------|---------|---------|
| **Client Type** | Confidential for server-side | `client_type: confidential` |
| **PKCE** | Require PKCE for public clients | `pkce_code_challenge_method: S256` |
| **Redirect URIs** | Restrict to exact URIs | No wildcards |
| **Grant Types** | Only enable needed grants | `authorization_code`, `client_credentials`, `refresh_token` |
| **Token Lifetimes** | Short access tokens; longer refresh | Access: 5 min; Refresh: 30 min |
| **Consent** | No consent for first-party clients | `consent_required: false` |

**Example (Client config):**
```json
{
  "clientId": "flowero-gate",
  "protocol": "openid-connect",
  "publicClient": false,
  "standardFlowEnabled": true,
  "directAccessGrantsEnabled": false,
  "serviceAccountsEnabled": true,
  "authorizationServicesEnabled": false,
  "redirectUris": ["https://api.panomete.com/*"],
  "attributes": {
    "pkce.code.challenge.method": "S256",
    "access.token.lifespan": "300",
    "client.offline.session.idle.timeout": "1800"
  }
}
```

## 4. Token Security

| Rule | Standard | Example |
|------|---------|---------|
| **Signing Algorithm** | RS256 (asymmetric) | `signature_algorithm: RS256` |
| **Access Token TTL** | 5 minutes | `access_token_lifespan: 300` |
| **Refresh Token TTL** | 30 minutes | `sso_session_idle_timeout: 1800` |
| **Token Rotation** | Rotate refresh tokens on use | `refresh_token_rotation: true` |
| **Revoke on Password Change** | Invalidate sessions | `revoke_refresh_token: true` |
| **Audience Restriction** | Include `aud` claim | `audience: true` |

## 5. Password & Credential Security

| Rule | Standard | Example |
|------|---------|---------|
| **Password Hashing** | bcrypt or Argon2 | Keycloak default: PBKDF2-SHA512 |
| **Min Password Length** | 12 characters | `min_length: 12` |
| **Password Complexity** | Upper + lower + digit + special | `password_policy` |
| **Account Lockout** | Lock after 5 failed attempts | `failure_factor: 5` |
| **Lockout Duration** | 15 minutes | `wait_increment: 900` |

## 6. Session Security

| Rule | Standard | Example |
|------|---------|---------|
| **SSO Session Idle** | 30 minutes | `sso_session_idle_timeout: 1800` |
| **SSO Session Max** | 10 hours | `sso_session_max_lifespan: 36000` |
| **Offline Session Idle** | 30 days | `offline_session_idle_timeout: 2592000` |
| **Remember Me** | Disabled by default | `remember_me: false` |
| **Concurrent Sessions** | Limit per user | `max_concurrent_sessions: 3` |

## 7. Admin Console Security

| Rule | Standard | Example |
|------|---------|---------|
| **MFA Required** | TOTP for all admin users | Configure OTP policy |
| **IP Whitelist** | Restrict to Tailscale IPs | Nginx `allow`/`deny` |
| **Strong Password** | 20+ characters for admin | Password policy |
| **Audit Logging** | Log all admin actions | Keycloak events |

**Example (Nginx IP whitelist):**
```nginx
server {
    server_name auth.panomete.com;
    
    location /admin {
        allow 100.73.0.0/16;  # Tailscale
        deny all;
        proxy_pass http://127.0.0.1:8001;
    }
}
```

## 8. Network & Deployment Security

| Rule | Standard | Example |
|------|---------|---------|
| **Internal Port** | Bind to `127.0.0.1:8001` | Docker port mapping |
| **HTTPS** | TLS at edge (Cloudflare) | `KC_PROXY: edge` |
| **Database** | Shared PostgreSQL with dedicated DB | `keycloak` database |
| **Resource Limits** | 1GB memory limit | Docker `deploy.resources.limits` |
| **Health Check** | `/health/ready` endpoint | Docker healthcheck |

**Example (docker-compose.yml):**
```yaml
services:
  flowero-guard:
    image: quay.io/keycloak/keycloak:latest
    ports:
      - "127.0.0.1:8001:8080"
    environment:
      KC_PROXY: edge
      KC_HOSTNAME: auth.panomete.com
      KC_HTTP_ENABLED: "true"
    deploy:
      resources:
        limits:
          memory: 1G
    healthcheck:
      test: ["CMD", "curl", "-sf", "http://localhost:8080/health/ready"]
      interval: 15s
      timeout: 5s
      retries: 3
```

## 9. Monitoring & Alerting

| Rule | Standard | Example |
|------|---------|---------|
| **Event Logging** | Log LOGIN, LOGOUT, CODE_TO_TOKEN events | Keycloak Events |
| **Admin Events** | Log all admin operations | Admin Events |
| **Failed Logins** | Alert on brute force patterns | Fail2ban + monitoring |
| **Token Errors** | Log invalid/expired token attempts | Application logs |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[061_security_test_report]] | Security test results |
| [[043_defect_report]] | Security defects |
| [[../03_construction/035_coding_standards_development]] | General configuration standards |
| [[../../panomete_platform/06_security/062_coding_standards_security]] | Platform-wide standards |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Top 10, OAuth 2.0 Security BCP
> **Usage:** These standards are mandatory. Review and update quarterly.
