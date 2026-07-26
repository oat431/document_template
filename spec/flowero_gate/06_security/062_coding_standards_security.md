---
document_type: Coding Standards / Security
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Gate"
project_id: "flowero-gate"
classification: "Internal"
tags: [coding-standards, security, java, spring-boot, spring-cloud-gateway]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP API Security Top 10
  - Spring Security Best Practices
---

# Coding Standards — Security (Flowero Gate)

> **Project:** Flowero Gate (Spring Cloud Gateway)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Security coding standards for Flowero Gate. Covers JWT validation, rate limiting, CORS, security headers, and secure filter configuration.

## 2. JWT Validation

| Rule | Standard | Example |
|------|---------|---------|
| **JWKS Cache TTL** | Configure cache TTL (5 minutes) | `NimbusReactiveJwtDecoder.withJwkSetUri(...).cache(Duration.ofMinutes(5))` |
| **Algorithm Restriction** | Accept only RS256 | `.jwsAlgorithm(SignatureAlgorithm.RS256)` |
| **Issuer Validation** | Validate `iss` claim | Spring Security validates automatically |
| **Audience Validation** | Validate `aud` if present | Custom converter if needed |
| **Expiration** | Reject expired tokens | Spring Security handles automatically |

**Example (SecurityConfig):**
```java
@Bean
public ReactiveJwtDecoder jwtDecoder() {
    NimbusReactiveJwtDecoder decoder = NimbusReactiveJwtDecoder
        .withJwkSetUri("https://auth.panomete.com/realms/panomete/protocol/openid-connect/certs")
        .jwsAlgorithm(SignatureAlgorithm.RS256)
        .build();
    // Cache JWKS for 5 minutes
    decoder = ReactiveJwtDecoders.fromOidcIssuerLocation(
        "https://auth.panomete.com/realms/panomete"
    );
    return decoder;
}
```

## 3. CORS Configuration

| Rule | Standard | Example |
|------|---------|---------|
| **Allowed Origins** | Restrict to `*.panomete.com` | `config.setAllowedOrigins(List.of("https://*.panomete.com"))` |
| **Allowed Methods** | GET, POST, PUT, DELETE, OPTIONS | `config.setAllowedMethods(...)` |
| **Allowed Headers** | Authorization, Content-Type only | `config.setAllowedHeaders(...)` |
| **Credentials** | Allow for trusted origins | `config.setAllowCredentials(true)` |
| **Max Age** | 1 hour | `config.setMaxAge(3600L)` |
| **Preflight** | Handle OPTIONS before auth check | Order `CorsWebFilter` at `@Order(-1)` |

**Example (CorsConfig):**
```java
@Bean
@Order(-1)  // Before SecurityWebFilterChain
public CorsWebFilter corsWebFilter() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of(
        "https://blog.panomete.com",
        "https://todo.panomete.com",
        "https://short.panomete.com"
    ));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    config.setAllowCredentials(true);
    config.setMaxAge(3600L);
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return new CorsWebFilter(source);
}
```

## 4. Security Headers

| Rule | Standard | Example |
|------|---------|---------|
| **X-Content-Type-Options** | `nosniff` | `.header("X-Content-Type-Options", "nosniff")` |
| **X-Frame-Options** | `DENY` | `.header("X-Frame-Options", "DENY")` |
| **X-XSS-Protection** | `1; mode=block` | `.header("X-XSS-Protection", "1; mode=block")` |
| **Strict-Transport-Security** | `max-age=31536000` | `.header("Strict-Transport-Security", "max-age=31536000")` |
| **Content-Security-Policy** | Restrictive policy | `.header("Content-Security-Policy", "default-src 'self'")` |
| **Referrer-Policy** | `strict-origin-when-cross-origin` | `.header("Referrer-Policy", "strict-origin-when-cross-origin")` |

**Example (SecurityConfig):**
```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return http
        .headers(headers -> headers
            .contentTypeOptions(Customizer.withDefaults())
            .frameOptions(frame -> frame.mode(SAMEORIGIN))
            .xssProtection(Customizer.withDefaults())
            .httpStrictTransportSecurity(Customizer.withDefaults())
        )
        // ... other config
        .build();
}
```

## 5. Rate Limiting

| Rule | Standard | Example |
|------|---------|---------|
| **Per-IP Limit** | 100 req/min default | Configure in route definition |
| **Per-User Limit** | 200 req/min for authenticated | Use `principalKeyResolver` |
| **Burst Protection** | No burst above 2x limit | Token bucket algorithm |
| **Fail-Open** | Allow requests if Valkey unavailable | Circuit breaker around Redis |
| **Response Headers** | Include rate limit info | `X-RateLimit-Remaining`, `X-RateLimit-Limit` |

**Example (application.yaml):**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: todo-service
          uri: lb://tiny-mchwa
          predicates:
            - Path=/api/v1/todo/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
                redis-rate-limiter.requestedTokens: 1
                key-resolver: "#{@principalKeyResolver}"
```

## 6. Filter Security

| Rule | Standard | Example |
|------|---------|---------|
| **Filter Order** | Security filters run first | `@Order` annotation |
| **Immutable Requests** | Don't modify request after auth | Use `exchange.mutate()` |
| **Sensitive Data** | Mask sensitive data in logs | `SensitiveDataMasker` |
| **Trace Propagation** | Preserve upstream trace IDs | Check before generating |

**Example (JwtClaimHeaderFilter):**
```java
@Component
public class JwtClaimHeaderFilter implements GlobalFilter, Ordered {
    @Override
    public int getOrder() {
        return 20;  // After security (order 0) but before routing
    }
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return ReactiveSecurityContextHolder.getContext()
            .map(SecurityContext::getAuthentication)
            .flatMap(auth -> {
                if (auth instanceof JwtAuthenticationToken jwtAuth) {
                    Jwt jwt = jwtAuth.getToken();
                    var builder = exchange.getRequest().mutate()
                        .header("X-User-Id", jwt.getClaimAsString("sub"))
                        .header("X-User-Email", jwt.getClaimAsString("email"))
                        .header("X-User-Roles", extractRealmRoles(jwt))
                        .header("X-Client-Id", jwt.getClaimAsString("client_id"));
                    
                    return chain.filter(exchange.mutate().request(builder.build()).build());
                }
                return chain.filter(exchange);
            });
    }
}
```

## 7. Error Handling

| Rule | Standard | Example |
|------|---------|---------|
| **No Stack Traces** | Never expose stack traces | Return generic error messages |
| **JSON Errors** | All API errors return JSON | `GatewayExceptionHandler` |
| **401 vs 403** | Correct status codes | 401 = unauthenticated, 403 = unauthorized |
| **No Information Leakage** | Don't reveal internal details | Generic error messages |

**Example (GatewayExceptionHandler):**
```java
@Component
public class GatewayExceptionHandler implements ErrorWebExceptionHandler {
    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        exchange.getResponse().setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR);
        exchange.getResponse().getHeaders().setContentType(MediaType.APPLICATION_JSON);
        
        String body = """
            {
              "error": "Internal Server Error",
              "status": 500,
              "message": "An unexpected error occurred",
              "timestamp": "%s"
            }
            """.formatted(Instant.now().toString());
        
        DataBuffer buffer = exchange.getResponse().bufferFactory()
            .wrap(body.getBytes(StandardCharsets.UTF_8));
        return exchange.getResponse().writeWith(Mono.just(buffer));
    }
}
```

## 8. Dependency Security

| Rule | Standard | Example |
|------|---------|---------|
| **Latest Versions** | Use latest stable | Spring Boot 4.1.0, Spring Cloud 2025.1.2 |
| **Vulnerability Scanning** | Scan in CI | GitHub Actions dependency check |
| **Minimal Dependencies** | Only necessary deps | Review `build.gradle` regularly |
| **No SNAPSHOT** | Never use SNAPSHOT in production | Pin release versions |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[061_security_test_report]] | Security test results |
| [[043_defect_report]] | Security defects |
| [[../03_construction/035_coding_standards_development]] | General coding standards |
| [[../../panomete_platform/06_security/062_coding_standards_security]] | Platform-wide standards |

---

> **Template Standard:** Based on SWEBOK v4, OWASP API Security Top 10, Spring Security Best Practices
> **Usage:** These standards are mandatory. Review and update quarterly.
