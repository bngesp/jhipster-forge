---
name: forge-new
description: Scaffold a new JHipster 8.x project interactively — monolith or microservices/gateway
argument-hint: "[monolith | microservice | gateway]"
allowed-tools: [Bash, Read, Write]
---

Create a new JHipster 8.x project in the current directory.

## Steps

1. **Check prerequisites**:
   ```bash
   node --version    # Must be >= 18
   java --version    # Must be >= 17
   jhipster --version # Must be >= 8.x
   ```
   If any prerequisite fails, tell the user what to install.

2. **Determine application type**:
   - If `$ARGUMENTS` is `monolith`, `microservice`, or `gateway`: use it directly
   - Otherwise ask: "What type of application? (monolith / microservice / gateway)"

3. **Guide configuration** based on type:

   **Monolith**:
   - Build tool: Maven
   - Auth: JWT or OAuth2/OIDC (Keycloak)
   - DB: PostgreSQL (production), H2 (dev)
   - Cache: Ehcache
   - Frontend: Angular

   **Microservice**:
   - Build tool: Maven
   - Auth: OAuth2/OIDC (same Keycloak realm as gateway)
   - DB: PostgreSQL
   - Service discovery: Eureka
   - No frontend

   **Gateway**:
   - Build tool: Maven
   - Auth: OAuth2/OIDC (Keycloak)
   - Service discovery: Eureka
   - Frontend: Angular

4. **Run JHipster**:
   ```bash
   jhipster
   ```
   Guide the user through the interactive prompts based on collected answers.

5. **Post-creation checklist**:
   - [ ] Run `./mvnw verify` — all tests must pass
   - [ ] Run `npm install` in project root for Angular
   - [ ] Start dev server: `./mvnw` + `npm start`
   - [ ] Verify app loads at `http://localhost:8080`

## Notes

- Always use PostgreSQL for production profile, H2 for dev
- OAuth2/OIDC requires Keycloak running — use `docker-compose -f src/main/docker/keycloak.yml up -d`
- For microservices: start Eureka registry first (`jhipster-registry`)
