# jhipster-forge

> The first dedicated Claude Code plugin for JHipster 8.x

Generate JHipster entities, DTOs, JDL models, services and more — directly from Claude Code, with production-grade architectural patterns built-in.

## Features

- **Entity generation** with guided field/relationship collection
- **JDL modeling** from natural language descriptions
- **Architectural patterns** auto-applied: Sealed Interface + Records, Strategy+Registry, Collect-Then-Fail validation, MapStruct polymorphic mapping
- **Microservices/Gateway** support with Keycloak/OAuth2 guidance
- **Auto-detection** of JHipster projects at session start

## Prerequisites

- [JHipster 8.x](https://www.jhipster.tech/) (`npm install -g generator-jhipster`)
- Java 17+
- Node.js 18+
- Maven or Gradle

## Installation

```bash
claude plugin install https://github.com/bassiroungom/jhipster-forge
```

## Commands

| Command | Description |
|---|---|
| `/jhipster-forge:forge-new [monolith\|microservice\|gateway]` | Scaffold a new JHipster project |
| `/jhipster-forge:forge-entity [EntityName]` | Generate an entity with fields and relationships |
| `/jhipster-forge:forge-jdl [description\|file.jdl]` | Create/import a JDL domain model |
| `/jhipster-forge:forge-service [EntityName]` | Generate a Spring service layer |
| `/jhipster-forge:forge-controller [EntityName]` | Generate a Spring REST controller |
| `/jhipster-forge:forge-liquibase` | Generate a Liquibase diff changelog |

## Agents

### `jdl-architect`
Designs complete JDL models from natural language. Describe your domain, get a production-ready `.jdl` file.

**Example:**
> "I need a JDL for an e-commerce app with products, categories, orders and customers"

### `forge-reviewer`
Reviews JHipster-generated code for architectural pattern compliance. Triggered automatically after entity generation.

## Skills (Auto-activated)

| Skill | Triggers on |
|---|---|
| `entity-generation` | Entity creation, domain modeling |
| `jdl-modeling` | JDL files, domain language questions |
| `architecture-patterns` | Sealed Interface, Strategy, validation, MapStruct |
| `microservices-gateway` | Microservice, gateway, Keycloak, Eureka |

## Hook

On session start, if a `.yo-rc.json` is detected in the project root, Claude displays:
```
⚡ JHipster project detected: myApp (monolith, oauth2, postgresql) — use /jhipster-forge:forge-entity, /jhipster-forge:forge-jdl and more.
```

## Architectural Patterns

This plugin is opinionated and applies these patterns from the Spring Boot 3.x + JHipster 8.x stack:

- **Sealed Interface + Records** — type-safe polymorphic DTOs (Java 17+)
- **Strategy + Registry** — polymorphic validation via Spring-injected `Map<String, Validator>`
- **Collect-Then-Fail** — `ValidationMonad` / `MessageList` for accumulating all errors
- **Single Table Inheritance** — JPA entity hierarchy with `@DiscriminatorColumn`
- **MapStruct Pattern Matching** — exhaustive mapping with Java 17 switch expressions
- **RFC7807** — standardized HTTP error responses via Zalando Problem

## Compatibility

- JHipster 8.x
- Spring Boot 3.x
- Java 17+
- Angular (frontend)
- PostgreSQL (production), H2 (dev)
- Monolith + Microservices/Gateway

## Author

Bassir Oungom — bassiroungom26@gmail.com

## License

MIT
