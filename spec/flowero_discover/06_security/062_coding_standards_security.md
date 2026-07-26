---
document_type: Coding Standards / Security
version: "1.0"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Discover"
project_id: "flowero-discover"
classification: "Internal"
tags: [coding-standards, security, java, spring-boot, eureka]
standard_ref:
  - SWEBOK v4 — Construction
  - OWASP Top 10
  - Spring Security Best Practices
---

# Coding Standards — Security (Flowero Discover)

> **Project:** Flowero Discover (Spring Cloud Netflix Eureka)
> **Version:** 1.0 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Security coding standards for Flowero Discover. Covers Eureka Server configuration, network isolation, dashboard access, and registration security.

## 2. Network Security

| Rule | Standard | Example |
|------|---------|---------|
| **Internal Only** | Eureka only accessible on Docker network | Bind to `127.0.0.1:8999` |
| **No External Access** | Block direct access from internet | UFW firewall rules |
| **Nginx Proxy** | Dashboard accessible via Nginx only | `discovery.panomete.com` |
| **Docker Network** | Use shared Docker network | `db-network` |

**Example (docker-compose.yml):**
```yaml
services:
  flowero-discover:
    ports:
      - "127.0.0.1:8999:8999"  # Internal only
      - "127.0.0.1:3999:3999"  # Dashboard internal only
    networks:
      - db-network
```

## 3. Dashboard Security

| Rule | Standard | Example |
|------|---------|---------|
| **Authentication** | Add basic auth to dashboard | Spring Security basic auth |
| **IP Whitelist** | Restrict dashboard access by IP | Nginx `allow`/`deny` |
| **HTTPS** | Dashboard accessible via HTTPS only | Cloudflare TLS |

**Example (application.yaml):**
```yaml
spring:
  security:
    user:
      name: ${EUREKA_USERNAME}
      password: ${EUREKA_PASSWORD}
```

**Example (Nginx):**
```nginx
server {
    server_name discovery.panomete.com;
    
    # IP whitelist
    allow 100.73.0.0/16;  # Tailscale
    deny all;
    
    location / {
        proxy_pass http://127.0.0.1:3999;
    }
}
```

## 4. Registration Security

| Rule | Standard | Example |
|------|---------|---------|
| **Standalone Mode** | Disable peer replication | `register-with-eureka: false` |
| **Self-Preservation** | Disable for single-instance | `enable-self-preservation: false` |
| **Eviction Timeout** | Configure appropriate timeout | `expiration-time-multiplier: 0.9` |
| **Audit Logging** | Log registration events | Enable Eureka audit log |

**Example (application.yaml):**
```yaml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 5000
```

## 5. Actuator Security

| Rule | Standard | Example |
|------|---------|---------|
| **Endpoint Exposure** | Expose only necessary endpoints | `health`, `prometheus` |
| **Health Endpoint** | Publicly accessible (for health checks) | No auth required |
| **Prometheus Endpoint** | Internal access only | Restrict to Prometheus server |

**Example (application.yaml):**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,prometheus
  endpoint:
    health:
      show-details: when-authorized
```

## 6. Dependency Security

| Rule | Standard | Example |
|------|---------|---------|
| **Latest Versions** | Use latest stable | Spring Boot 4.1.0, Spring Cloud 2025.1.2 |
| **Vulnerability Scanning** | Scan in CI | GitHub Actions dependency check |
| **Minimal Dependencies** | Only Eureka Server + Actuator | Review `build.gradle` |

**Example (build.gradle):**
```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
}
```

## 7. Logging & Auditing

| Rule | Standard | Example |
|------|---------|---------|
| **Structured Logs** | JSON format | Logback JSON encoder |
| **Registration Events** | Log all registration/deregistration | Eureka event listener |
| **No Sensitive Data** | Don't log service credentials | Mask sensitive fields |

**Example (Custom Event Listener):**
```java
@Component
public class EurekaAuditListener {
    @EventListener
    public void onRegistration(EurekaInstanceRegisteredEvent event) {
        log.info("Service registered: instanceId={}, app={}", 
            event.getInstanceInfo().getInstanceId(),
            event.getInstanceInfo().getAppName());
    }
    
    @EventListener
    public void onCancellation(EurekaInstanceCanceledEvent event) {
        log.info("Service deregistered: appName={}", event.getAppName());
    }
}
```

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[061_security_test_report]] | Security test results |
| [[../03_construction/035_coding_standards_development]] | General coding standards |
| [[../../panomete_platform/06_security/062_coding_standards_security]] | Platform-wide standards |

---

> **Template Standard:** Based on SWEBOK v4, OWASP Top 10, Spring Security Best Practices
> **Usage:** These standards are mandatory. Review and update quarterly.
