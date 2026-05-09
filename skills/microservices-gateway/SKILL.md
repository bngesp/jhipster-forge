---
name: JHipster Microservices & Gateway
description: Activates when working on a JHipster microservices architecture — including gateway configuration, inter-service communication, OAuth2/Keycloak setup, service registry, or when the user mentions microservice, gateway, Eureka, Consul, or Keycloak in a JHipster context.
---

# JHipster Microservices & Gateway

## Architecture Overview

```
Browser / Mobile
      │
      ▼
┌─────────────┐
│   Gateway   │  ← OAuth2 Resource Server (Keycloak)
│  (Angular)  │  ← Spring Cloud Gateway
└──────┬──────┘
       │
  ┌────┴────┐
  │ Eureka  │  ← Service Registry
  └────┬────┘
  ┌────┴──────────────┐
  │                   │
  ▼                   ▼
┌──────────┐    ┌──────────┐
│ Service A│    │ Service B│  ← Microservices
└──────────┘    └──────────┘
```

## JHipster Generation Commands

### Create a Gateway
```bash
jhipster
# Choose: Gateway
# Auth: OAuth2 / OIDC (Keycloak)
# Service discovery: Eureka
```

### Create a Microservice
```bash
jhipster
# Choose: Microservice application
# Auth: OAuth2 / OIDC (same Keycloak realm)
# Service discovery: Eureka (same registry)
# Port: unique per service (8081, 8082...)
```

### Register a Microservice Entity on the Gateway
```bash
# In the gateway project:
jhipster entity EntityName --microservice-name serviceA --microservice-url http://localhost:8081
```

### Import JDL for Microservices
```jdl
// In JDL, declare microservice ownership
microservice Product, Category with productService
microservice Order, OrderItem with orderService

// Gateway displays all
clientRootFolder Product, Category with productService
clientRootFolder Order with orderService

paginate Product with infinite-scroll
paginate Order with pagination
dto * with mapstruct
service * with serviceClass
```

## Keycloak / OAuth2 Configuration

### application.yml (microservice)
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9080/realms/jhipster
      client:
        provider:
          oidc:
            issuer-uri: http://localhost:9080/realms/jhipster
        registration:
          oidc:
            client-id: internal
            client-secret: internal
```

### Keycloak Setup Checklist
- [ ] Realm `jhipster` created
- [ ] Client `web_app` for gateway (Authorization Code flow)
- [ ] Client `internal` for service-to-service (Client Credentials flow)
- [ ] Roles: `ROLE_USER`, `ROLE_ADMIN` mapped
- [ ] Users created with roles assigned

## Inter-Service Communication

### Using Spring Cloud OpenFeign (JHipster default)
```java
@FeignClient(name = "productService")
public interface ProductServiceClient {
    @GetMapping("/api/products/{id}")
    ProductDTO getProduct(@PathVariable Long id);
}
```

### Security Context Propagation
JHipster auto-configures JWT propagation between services. Verify `AuthorizationHeaderUtil` is used if calling services manually.

## Service Registry (Eureka)

### Start Eureka locally
```bash
# JHipster generates a registry project
cd jhipster-registry
./mvnw spring-boot:run
# Dashboard: http://localhost:8761
```

### Check service registration
All microservices and gateway must appear in Eureka dashboard before testing inter-service calls.

## Docker Compose (Full Stack)

JHipster generates Docker Compose for the full stack:
```bash
# In gateway project
jhipster docker-compose
# Select all services to include

# Start everything
docker-compose up -d
```

Services started:
- Keycloak (port 9080)
- Eureka Registry (port 8761)
- PostgreSQL per service
- All microservices
- Gateway

## Common Issues & Fixes

| Issue | Cause | Fix |
|---|---|---|
| 401 on inter-service call | Missing JWT propagation | Add `AuthorizationHeaderUtil` to Feign config |
| Service not found | Not registered in Eureka | Check `eureka.client.service-url` in config |
| CORS error | Gateway CORS config | Check `jhipster.cors` in gateway `application.yml` |
| Token expired | Short Keycloak TTL | Increase `Access Token Lifespan` in Keycloak realm |
| Port conflict | Two services same port | Each service needs unique `server.port` |

## Multi-Service JDL Workflow

When user describes a multi-service domain:
1. Identify bounded contexts → one per microservice
2. Generate JDL with `microservice` declarations
3. Create each microservice project first
4. Import JDL into each microservice
5. Import JDL into gateway (for Angular UI)
6. Configure Docker Compose
