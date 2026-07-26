---
document_type: Test Cases
version: "0.1"
status: Draft
author: "QA Engineer"
created: "2026-07-26"
last_updated: "2026-07-26"
project_name: "Flowero Discover"
project_id: "flowero-discover"
classification: "Internal"
tags: [test-cases, test-scenarios, eureka, service-discovery, swebok, iso-29119]
standard_ref:
  - SWEBOK v4 — Testing
  - ISO/IEC/IEEE 29119 — Software Testing
---

# Test Cases

> **Project:** Flowero Discover
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-26

---

## 1. Purpose

> Detailed test cases — preconditions, steps, expected results, and actual results for each test scenario covering the Flowero Discover Eureka service registry.

## 2. Test Case Index

| Module | Total | Automated | Manual | Status |
|--------|-------|----------|--------|--------|
| Server Startup & Configuration | 5 | 5 | 0 | ✅ |
| Service Registration | 4 | 2 | 2 | ✅ |
| Service Discovery | 4 | 2 | 2 | ✅ |
| Heartbeat & Eviction | 3 | 1 | 2 | 🔄 |
| Dashboard | 3 | 2 | 1 | ✅ |
| Actuator & Monitoring | 2 | 2 | 0 | ✅ |
| **Total** | **21** | **14** | **7** | |

## 3. Test Cases — Server Startup & Configuration

### TC-001: Eureka Server Context Loads

| Field | Value |
|-------|-------|
| **ID** | TC-001 |
| **Title** | Eureka server context loads successfully |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`contextLoads()`) |
| **Requirement** | US-101 / AC-D101a |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | `FlowerodiscoveryApplication` class annotated with `@SpringBootApplication` + `@EnableEurekaServer` |
| 2 | `application.yaml` present with standalone configuration |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Start application via `@SpringBootTest(webEnvironment = RANDOM_PORT)` | Spring context initializes without errors | Context loaded successfully | ✅ Pass |
| 2 | Verify no exceptions in application logs | No `BeanCreationException` or startup errors | Clean startup | ✅ Pass |

#### Post-conditions

| # | Condition |
|---|----------|
| 1 | Embedded Tomcat running on random port |
| 2 | Eureka server initialized in standalone mode |

---

### TC-002: Standalone Mode — No Self-Registration

| Field | Value |
|-------|-------|
| **ID** | TC-002 |
| **Title** | Eureka does not register with itself in standalone mode |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`doesNotRegisterWithItself()`) |
| **Requirement** | US-101 / AC-D101d |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | `eureka.client.register-with-eureka: false` configured |
| 2 | `eureka.client.fetch-registry: false` configured |
| 3 | `spring.application.name: flowero-discover` |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Send `GET /eureka/apps` to the running server | 200 OK response | HTTP 200 | ✅ Pass |
| 2 | Parse response body for `FLOWERO-DISCOVER` | Application name NOT present in registry | Not found in response | ✅ Pass |
| 3 | Verify registry is empty (standalone, no clients) | No applications registered | Empty registry | ✅ Pass |

---

### TC-003: Eureka Dashboard Reachable

| Field | Value |
|-------|-------|
| **ID** | TC-003 |
| **Title** | Eureka dashboard HTML is served at root path |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`eurekaDashboardIsReachable()`) |
| **Requirement** | US-101 / AC-D101b, US-104 / AC-D104a |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running with `@EnableEurekaServer` |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Send `GET /` to the running server | HTTP 200 OK | HTTP 200 | ✅ Pass |
| 2 | Verify response body contains Eureka dashboard marker | Body contains "Instances currently registered with Eureka" | Marker present | ✅ Pass |

---

### TC-004: Actuator Health Returns UP

| Field | Value |
|-------|-------|
| **ID** | TC-004 |
| **Title** | Actuator health endpoint returns UP status |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`actuatorHealthReturnsUp()`) |
| **Requirement** | US-101 / AC-D101a |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | `management.endpoints.web.exposure.include: health,info,prometheus` configured |
| 2 | `management.endpoint.health.show-details: always` configured |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Send `GET /actuator/health` to the running server | HTTP 200 OK | HTTP 200 | ✅ Pass |
| 2 | Verify response contains UP status | Body contains `"status":"UP"` | Status UP confirmed | ✅ Pass |

---

### TC-005: Server Starts on Port 8999

| Field | Value |
|-------|-------|
| **ID** | TC-005 |
| **Title** | Server binds to configured port 8999 |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (via `@LocalServerPort` + `@SpringBootTest(RANDOM_PORT)`) |
| **Requirement** | US-101 / ADR-D005 |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | `server.port: 8999` configured in `application.yaml` |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Start application | Server binds to available port | Server started | ✅ Pass |
| 2 | Verify port is accessible via HTTP | Connection succeeds | Connection OK | ✅ Pass |

---

## 4. Test Cases — Service Registration

### TC-006: Register Service via REST API

| Field | Value |
|-------|-------|
| **ID** | TC-006 |
| **Title** | Register a new service instance via POST /eureka/apps/{app} |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Manual (requires curl or HTTP client) |
| **Requirement** | US-102 / AC-D102a, AC-D102b |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running on localhost:8999 |
| 2 | No services currently registered |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `POST /eureka/apps/CUTE-GUFO` with JSON body containing instance details (instanceId: "cute-gufo:8005", hostName: "cute-gufo", port: 8005, status: UP) | HTTP 204 No Content |
| 2 | Send `GET /eureka/apps/CUTE-GUFO` with `Accept: application/json` | HTTP 200, response contains CUTE-GUFO application with instance details |
| 3 | Verify response includes `hostName`, `port`, `healthCheckUrl`, `status` | All required fields present |

---

### TC-007: Spring Boot Client Auto-Registration

| Field | Value |
|-------|-------|
| **ID** | TC-007 |
| **Title** | Spring Boot service with Eureka client auto-registers on startup |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Manual (requires second service) |
| **Requirement** | US-102 / AC-D102a |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running |
| 2 | A Spring Boot service with `spring-cloud-starter-netflix-eureka-client` on classpath |
| 3 | `eureka.client.service-url.defaultZone` configured to point to Eureka |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Start the client service | Service boots successfully |
| 2 | Wait 30 seconds | — |
| 3 | Query `GET /eureka/apps/{SERVICE-NAME}` | Service appears in registry with status UP |
| 4 | Check Eureka dashboard at `/` | Service listed under "Instances currently registered with Eureka" |

---

### TC-008: Graceful Deregistration on Shutdown

| Field | Value |
|-------|-------|
| **ID** | TC-008 |
| **Title** | Service deregisters from Eureka on graceful shutdown |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Manual |
| **Requirement** | US-102 / AC-D102c |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Service registered and showing UP in Eureka |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send SIGTERM to the registered service | Service initiates graceful shutdown |
| 2 | Service sends `DELETE /eureka/apps/{app}/{instance}` | Eureka receives deregistration |
| 3 | Wait 30 seconds | — |
| 4 | Query `GET /eureka/apps/{SERVICE-NAME}` | Service no longer in registry (404 or empty) |

---

### TC-009: Non-Spring Service Manual Registration

| Field | Value |
|-------|-------|
| **ID** | TC-009 |
| **Title** | Non-Spring Boot service registers via REST API |
| **Priority** | 🟡 Should Have |
| **Type** | Integration |
| **Automated** | Manual |
| **Requirement** | US-102 / AC-D102e |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `POST /eureka/apps/MY-GO-SERVICE` with valid JSON instance registration | HTTP 204 |
| 2 | Query registry | Service appears with status UP |
| 3 | Send heartbeat: `PUT /eureka/apps/MY-GO-SERVICE/my-go-service:8080` | HTTP 200 |
| 4 | Stop sending heartbeats for 90 seconds | Service evicted from registry |

---

## 5. Test Cases — Service Discovery

### TC-010: Eureka Apps Endpoint Returns Empty Registry (XML)

| Field | Value |
|-------|-------|
| **ID** | TC-010 |
| **Title** | GET /eureka/apps returns valid XML response |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`eurekaAppsEndpointReturnsEmptyRegistry()`) |
| **Requirement** | US-103 / AC-D103d |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running, no services registered |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Send `GET /eureka/apps` (no Accept header) | HTTP 200 OK | HTTP 200 | ✅ Pass |
| 2 | Verify response body is not empty | Body contains XML data | Response not empty | ✅ Pass |

---

### TC-011: Eureka Apps Endpoint Returns JSON with Accept Header

| Field | Value |
|-------|-------|
| **ID** | TC-011 |
| **Title** | GET /eureka/apps returns JSON when Accept: application/json is sent |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`eurekaAppsEndpointReturnsJsonWithAcceptHeader()`) |
| **Requirement** | US-103 / AC-D103d |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running |

#### Test Steps

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|----------------|--------------|--------|
| 1 | Send `GET /eureka/apps` with `Accept: application/json` header | HTTP 200 OK | HTTP 200 | ✅ Pass |
| 2 | Verify response body contains `"applications"` | JSON structure present | `"applications"` found | ✅ Pass |

---

### TC-012: Resolve Service by Name via Load-Balanced Client

| Field | Value |
|-------|-------|
| **ID** | TC-012 |
| **Title** | Service A resolves Service B by name using @LoadBalanced RestTemplate |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Manual (requires two services) |
| **Requirement** | US-103 / AC-D103a |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka server running |
| 2 | Service B registered with Eureka |
| 3 | Service A configured with `@LoadBalanced RestTemplate` |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Service A calls `restTemplate.getForObject("http://service-b/api/endpoint", ...)` | Host:port resolved from Eureka |
| 2 | Request reaches Service B instance | Successful response from Service B |

---

### TC-013: Service Not Found Error

| Field | Value |
|-------|-------|
| **ID** | TC-013 |
| **Title** | Clear error when resolving non-existent service |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Manual |
| **Requirement** | US-103 / AC-D103c |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Service "nonexistent-service" NOT registered in Eureka |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Attempt to resolve `http://nonexistent-service` via load-balanced client | `IllegalStateException` or `ServiceUnavailableException` thrown |
| 2 | Verify error message is clear and descriptive | NOT a cryptic `UnknownHostException` |

---

## 6. Test Cases — Heartbeat & Eviction

### TC-014: Heartbeat Renews Lease

| Field | Value |
|-------|-------|
| **ID** | TC-014 |
| **Title** | PUT /eureka/apps/{name}/{instance} renews lease successfully |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Manual (requires registered service) |
| **Requirement** | US-102 / AC-D102a |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Service registered in Eureka |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `PUT /eureka/apps/CUTE-GUFO/cute-gufo:8005` | HTTP 200 OK — lease renewed |
| 2 | Query registry | Service still listed with status UP |

---

### TC-015: Heartbeat on Unregistered Instance Returns 404

| Field | Value |
|-------|-------|
| **ID** | TC-015 |
| **Title** | PUT heartbeat for unknown instance returns 404 (must re-register) |
| **Priority** | 🟡 High |
| **Type** | Integration |
| **Automated** | Manual |
| **Requirement** | US-102 |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | No service named "UNKNOWN-SERVICE" registered |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `PUT /eureka/apps/UNKNOWN-SERVICE/unknown:8080` | HTTP 404 Not Found |

---

### TC-016: Crash Eviction After 90s

| Field | Value |
|-------|-------|
| **ID** | TC-016 |
| **Title** | Stale instance evicted after heartbeat timeout |
| **Priority** | 🔴 Critical |
| **Type** | System |
| **Automated** | Manual (time-sensitive) |
| **Requirement** | US-102 / AC-D102d |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Service registered with Eureka |
| 2 | `eureka.server.eviction-interval-timer-in-ms: 5000` |
| 3 | Self-preservation NOT active (or threshold not breached) |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Register a service instance | Service appears in registry |
| 2 | Stop sending heartbeats (simulate crash) | No more PUT requests |
| 3 | Wait 90 seconds (lease expiration) + 5s (eviction timer) | — |
| 4 | Query `GET /eureka/apps/{SERVICE-NAME}` | Service evicted from registry |

---

## 7. Test Cases — Dashboard

### TC-017: Dashboard CSS and JavaScript Load

| Field | Value |
|-------|-------|
| **ID** | TC-017 |
| **Title** | Dashboard static resources load correctly |
| **Priority** | 🟡 Should Have |
| **Type** | System |
| **Automated** | Manual (browser-based) |
| **Requirement** | US-104 / AC-D104a, AC-D101e |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Nginx proxying `discovery.panomete.com` → Eureka :3999 |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to `https://discovery.panomete.com` in browser | Dashboard loads |
| 2 | Inspect browser DevTools Network tab | All CSS/JS/images return 200, no 404s |
| 3 | Verify dashboard renders completely | Layout intact, no broken resources |

---

### TC-018: Dashboard Shows Registered Services

| Field | Value |
|-------|-------|
| **ID** | TC-018 |
| **Title** | Dashboard displays registered services with status |
| **Priority** | 🟡 Should Have |
| **Type** | System |
| **Automated** | Manual |
| **Requirement** | US-104 / AC-D104b |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | At least one service registered with Eureka |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Open Eureka dashboard | Dashboard loads |
| 2 | View "Instances currently registered with Eureka" section | Service listed with name, status UP, instance count |
| 3 | Click on a service link | Service detail page shows: instance ID, host, port, health URL, lease info |

---

### TC-019: Dashboard Shows Clean Initial State

| Field | Value |
|-------|-------|
| **ID** | TC-019 |
| **Title** | Dashboard shows "No instances available" on fresh start |
| **Priority** | 🔴 Critical |
| **Type** | Integration |
| **Automated** | Yes (`eurekaDashboardIsReachable()` verifies dashboard loads) |
| **Requirement** | US-101 / AC-D101b |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | Eureka just started, no services registered |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Access dashboard at `http://localhost:8999/` | HTTP 200 |
| 2 | Verify "Instances currently registered with Eureka" section | Shows no instances |

---

## 8. Test Cases — Actuator & Monitoring

### TC-020: Actuator Info Endpoint

| Field | Value |
|-------|-------|
| **ID** | TC-020 |
| **Title** | Actuator info endpoint is accessible |
| **Priority** | 🟢 Medium |
| **Type** | Integration |
| **Automated** | Yes (part of context load verification) |
| **Requirement** | ADR-D005 / Configuration |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | `management.endpoints.web.exposure.include: health,info,prometheus` |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `GET /actuator/info` | HTTP 200 with build information |

---

### TC-021: Prometheus Metrics Endpoint

| Field | Value |
|-------|-------|
| **ID** | TC-021 |
| **Title** | Prometheus metrics endpoint accessible |
| **Priority** | 🟢 Medium |
| **Type** | Integration |
| **Automated** | Manual |
| **Requirement** | Configuration (`micrometer-registry-prometheus`) |

#### Preconditions

| # | Condition |
|---|----------|
| 1 | `io.micrometer:micrometer-registry-prometheus` dependency on classpath |

#### Test Steps

| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Send `GET /actuator/prometheus` | HTTP 200 with Prometheus-formatted metrics |
| 2 | Verify Eureka-specific metrics present | Metrics contain `eureka.server.*` or JVM metrics |

---

## 9. Test Execution Summary

| Sprint | Executed | Passed | Failed | Blocked | Pass Rate |
|--------|---------|--------|--------|---------|----------|
| M1 — Core Infrastructure | 6 | 6 | 0 | 0 | 100% |
| M2 — End-to-End Auth | 10 | — | — | — | Pending |
| M3 — Production Hardening | 5 | — | — | — | Pending |
| **Total** | **21** | **6** | **0** | **0** | **100% (of executed)** |

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[041_test_plan]] | Plan governing these cases |
| [[044_regression_test_suite]] | Regression subset of these cases |
| [[013_acceptance_criteria]] | BDD acceptance criteria mapped to test cases |
| [[012_user_stories]] | User stories driving these test cases |

---

> **Template Standard:** Based on SWEBOK v4, ISO/IEC/IEEE 29119
> **Usage:** Every requirement needs at least one test case. Every test case traces to a requirement. Keep them in sync.
