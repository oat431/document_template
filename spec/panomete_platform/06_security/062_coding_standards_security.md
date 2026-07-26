---
document_type: Coding Standards / Security
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Panomete Platform"
project_id: "PAN-PLAT-001"
classification: "Internal"
tags: [coding-standards, security, java, spring-boot, owasp]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Top 10
  - Spring Security Best Practices
---

# Coding Standards — Security (Panomete Platform)

> **Project:** Panomete Platform
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Platform-wide security coding standards for the Panomete Platform. These standards apply to all foundation services (Guard, Discover, Gate) and future business services.

## 2. Authentication & Authorization

### 2.1 JWT Validation

| Rule | Standard | Example |
|------|---------|---------|
| **JWT Validation** | Validate JWT signature locally using cached JWKS | `SecurityConfig.java` uses `oauth2ResourceServer().jwt()` |
| **JWKS Cache** | Configure JWKS cache TTL (e.g., 5 minutes) | `JwkSetUriReactiveJwtDecoder` with cache |
| **Issuer Validation** | Validate JWT `iss` claim matches expected issuer | `https://auth.panomete.com/realms/panomete` |
| **Audience Validation** | Validate JWT `aud` claim if present | Check `aud` contains service client ID |
| **Expiration Check** | Reject expired tokens | Spring Security handles this automatically |

**Example (Gate):**
```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        return http
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtDecoder(jwkSetUriReactiveJwtDecoder())
                )
            )
            .build();
    }
    
    @Bean
    public ReactiveJwtDecoder jwkSetUriReactiveJwtDecoder() {
        return NimbusReactiveJwtDecoder.withJwkSetUri(
            "https://auth.panomete.com/realms/panomete/protocol/openid-connect/certs"
        ).jwsAlgorithm(SignatureAlgorithm.RS256).build();
    }
}
```

### 2.2 Claim Forwarding

| Rule | Standard | Example |
|------|---------|---------|
| **User ID** | Forward `sub` claim as `X-User-Id` header | `JwtClaimHeaderFilter.java` |
| **User Email** | Forward `email` claim as `X-User-Email` header | `JwtClaimHeaderFilter.java` |
| **User Roles** | Forward `realm_access.roles` as `X-User-Roles` header | `JwtClaimHeaderFilter.java` |
| **User Scope** | Forward `scope` claim as `X-User-Scope` header | `JwtClaimHeaderFilter.java` |
| **Client ID** | Forward `client_id` claim as `X-Client-Id` header (for S2S) | Extend `JwtClaimHeaderFilter` |

**Example (JwtClaimHeaderFilter):**
```java
@Component
public class JwtClaimHeaderFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return ReactiveSecurityContextHolder.getContext()
            .map(SecurityContext::getAuthentication)
            .flatMap(auth -> {
                if (auth instanceof JwtAuthenticationToken jwtAuth) {
                    Jwt jwt = jwtAuth.getToken();
                    var builder = exchange.getRequest().mutate();
                    
                    builder.header("X-User-Id", jwt.getClaimAsString("sub"));
                    builder.header("X-User-Email", jwt.getClaimAsString("email"));
                    builder.header("X-User-Roles", extractRealmRoles(jwt));
                    builder.header("X-User-Scope", jwt.getClaimAsString("scope"));
                    
                    return chain.filter(exchange.mutate().request(builder.build()).build());
                }
                return chain.filter(exchange);
            });
    }
}
```

### 2.3 RBAC

| Rule | Standard | Example |
|------|---------|---------|
| **Role-Based Access** | Enforce RBAC at Gate and service level | `@PreAuthorize("hasRole('admin')")` |
| **Role Hierarchy** | Define role hierarchy if needed | `admin > editor > viewer` |
| **Default Deny** | Deny access by default; explicitly permit | `anyExchange().authenticated()` |

## 3. Network Security

### 3.1 Firewall Rules

| Rule | Standard | Example |
|------|---------|---------|
| **UFW** | Allow only ports 22 (SSH), 80 (HTTP), 443 (HTTPS) | `sudo ufw allow 22,80,443/tcp` |
| **Internal Ports** | Block direct access to internal ports (8000, 8001, 8999) | Not in UFW rules |
| **Fail2ban** | Enable Fail2ban for SSH protection | `sudo fail2ban-client status sshd` |

### 3.2 Docker Network

| Rule | Standard | Example |
|------|---------|---------|
| **Network Isolation** | Use shared Docker network for inter-service communication | `db-network` |
| **Port Binding** | Bind to `127.0.0.1` only (not `0.0.0.0`) | `127.0.0.1:8000:8000` |
| **No External Access** | Internal services not accessible from outside | Nginx reverse proxy only |

### 3.3 TLS

| Rule | Standard | Example |
|------|---------|---------|
| **External TLS** | TLS 1.3 via Cloudflare | Cloudflare handles TLS termination |
| **Internal HTTP** | Plain HTTP on trusted Docker network | No TLS between Nginx and services |
| **Certificate Management** | Cloudflare manages certificates | Automatic renewal |

## 4. Secret Management

| Rule | Standard | Example |
|------|---------|---------|
| **No Secrets in Git** | Never commit secrets to version control | `.env` in `.gitignore` |
| **Environment Variables** | Use environment variables for secrets | `POSTGRES_PASSWORD`, `KC_DB_PASSWORD` |
| **Secret Rotation** | Rotate secrets quarterly | `POSTGRES_PASSWORD`, `VALKEY_PASSWORD` |
| **GitHub Secrets** | Use GitHub Actions secrets for CI/CD | `TS_AUTH_KEY`, `HOMELAB_SSH_KEY` |

**Example (.gitignore):**
```
.env
*.env.local
```

**Example (docker-compose.yml):**
```yaml
services:
  flowero-guard:
    environment:
      KC_DB_PASSWORD: ${KC_DB_PASSWORD}
```

## 5. Input Validation

| Rule | Standard | Example |
|------|---------|---------|
| **Parameterized Queries** | Use parameterized queries to prevent SQL injection | Spring Data JPA handles this |
| **Input Sanitization** | Sanitize user input at Gate | Validate request body size, content type |
| **Rate Limiting** | Enforce rate limits at Gate | Valkey-backed rate limiter |
| **Request Size Limit** | Limit request body size | `client_max_body_size 10M` in Nginx |

## 6. Logging & Auditing

| Rule | Standard | Example |
|------|---------|---------|
| **Structured Logs** | Emit structured JSON logs | `timestamp`, `level`, `service`, `trace_id` |
| **Audit Trail** | Log authentication events | Login, logout, token validation |
| **No Sensitive Data** | Never log passwords, tokens, or PII | Mask sensitive fields |
| **Log Rotation** | Rotate logs to prevent disk exhaustion | Docker log driver configuration |

**Example (RequestLoggingFilter):**
```java
@Component
public class RequestLoggingFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        long startTime = System.currentTimeMillis();
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            long duration = System.currentTimeMillis() - startTime;
            log.info("method={} path={} status={} duration_ms={}",
                exchange.getRequest().getMethod(),
                exchange.getRequest().getPath(),
                exchange.getResponse().getStatusCode(),
                duration);
        }));
    }
}
```

## 7. CORS Configuration

| Rule | Standard | Example |
|------|---------|---------|
| **Allowed Origins** | Restrict to trusted origins only | `*.panomete.com` |
| **Allowed Methods** | Allow only necessary HTTP methods | GET, POST, PUT, DELETE, OPTIONS |
| **Allowed Headers** | Allow only necessary headers | Authorization, Content-Type |
| **Credentials** | Allow credentials for cross-origin requests | `allowCredentials(true)` |
| **Preflight** | Handle OPTIONS preflight correctly | Ensure CORS filter ordered before security |

**Example (CorsConfig):**
```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("https://*.panomete.com"));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
        config.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}
```

## 8. Rate Limiting

| Rule | Standard | Example |
|------|---------|---------|
| **Per-IP Limit** | Rate limit by IP address | `ipKeyResolver()` |
| **Per-User Limit** | Rate limit by authenticated user | `principalKeyResolver()` |
| **Per-Route Limit** | Rate limit by API route | Configure per-route limits |
| **Valkey-Backed** | Use Valkey for persistent rate limits | Survives Gate restarts |
| **Fail-Open** | Fail open if Valkey unavailable | Allow requests if Valkey down |

**Example (RateLimiterConfig):**
```java
@Configuration
public class RateLimiterConfig {
    @Bean
    @Primary
    public KeyResolver principalKeyResolver() {
        return exchange -> exchange.getPrincipal()
            .map(Principal::getName)
            .defaultIfEmpty("anonymous");
    }
}
```

## 9. Error Handling

| Rule | Standard | Example |
|------|---------|---------|
| **No Stack Traces** | Never expose stack traces to clients | Return generic error messages |
| **Standardized Errors** | Use consistent error response format | JSON with `error`, `status`, `message`, `timestamp` |
| **401 vs 403** | Return 401 for unauthenticated, 403 for unauthorized | `SecurityConfig.java` |
| **Circuit Breaker** | Use circuit breaker for downstream failures | Resilience4j |

**Example (Error Response):**
```json
{
  "error": "Unauthorized",
  "status": 401,
  "message": "Authentication required",
  "timestamp": "2026-07-26T12:00:00Z"
}
```

## 10. Dependency Management

| Rule | Standard | Example |
|------|---------|---------|
| **Latest Versions** | Use latest stable versions | Spring Boot 4.1.0, Spring Cloud 2025.1.2 |
| **Vulnerability Scanning** | Scan dependencies for CVEs | GitHub Actions dependency check |
| **Minimal Dependencies** | Include only necessary dependencies | Review `build.gradle` regularly |
| **Pin Versions** | Pin dependency versions | Use Gradle version catalog |

**Example (build.gradle):**
```gradle
plugins {
    id 'org.springframework.boot' version '4.1.0'
    id 'io.spring.dependency-management' version '1.1.7'
}

ext {
    set('springCloudVersion', "2025.1.2")
}
```

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[061_security_test_report]] | Security test results |
| [[../03_construction/031_README_developer_guide]] | Developer guide |
| [[../flowero_gate/06_security/062_coding_standards_security]] | Gate-specific standards |
| [[../flowero_discover/06_security/062_coding_standards_security]] | Discover-specific standards |
| [[../flowero_guard/06_security/062_coding_standards_security]] | Guard-specific standards |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Top 10, Spring Security Best Practices
> **Usage:** These standards are mandatory for all platform services. Review and update quarterly.
