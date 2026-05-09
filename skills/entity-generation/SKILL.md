---
name: JHipster Entity Generation
description: Activates when a user wants to generate a JHipster entity, create domain objects, add fields to an existing entity, or scaffold database models in a JHipster 8.x project. Also activates when the user mentions entity creation, domain modeling, or JPA entity generation in a Spring Boot/JHipster context.
---

# JHipster Entity Generation

## Pre-Generation Checklist

Before running any JHipster entity command, verify:
1. Run `jhipster info` to confirm JHipster version (must be 8.x)
2. Check if a `.yo-rc.json` exists at project root — if not, this is not a JHipster project
3. Identify the application type from `.yo-rc.json`: `monolith`, `microservice`, or `gateway`
4. Check existing entities in `src/main/java/**/domain/` to avoid naming conflicts

## Workflow A — Argument Provided (`/forge-entity EntityName`)

When the user provides an entity name directly:
1. Ask for fields: name, type, validation, nullable
2. Ask for relationships: type (ManyToOne, OneToMany, ManyToMany, OneToOne), target entity
3. Ask for pagination: no, pagination, infinite-scroll
4. Confirm summary before running
5. Run: `jhipster entity EntityName`

## Workflow B — No Argument (interactive)

When no entity name is provided, ask progressively:
1. "What is the entity name?" (PascalCase, singular — e.g. `UserProfile`)
2. "List the fields (name:type:validation)" — offer examples:
   - `firstName:String:required,maxlength(50)`
   - `birthDate:LocalDate`
   - `salary:BigDecimal:min(0)`
3. "Any relationships with other entities?"
4. "Pagination type?"
5. Confirm then run `jhipster entity EntityName`

## Supported Field Types

| JHipster Type | Java Type | Example |
|---|---|---|
| String | String | `name:String:required` |
| Integer / Long | Integer / Long | `age:Integer:min(0),max(150)` |
| BigDecimal | BigDecimal | `price:BigDecimal` |
| LocalDate | LocalDate | `birthDate:LocalDate` |
| ZonedDateTime | ZonedDateTime | `createdAt:ZonedDateTime` |
| Boolean | Boolean | `active:Boolean` |
| Enum | Custom Enum | `status:StatusEnum` |
| Blob / AnyBlob | byte[] | `document:AnyBlob` |

## Architectural Patterns to Apply Post-Generation

After JHipster generates the entity, apply the project's architectural patterns:

### 1. Sealed Interface + Records (Java 17+)
If the entity has subtypes (discriminator column), generate:
- `EntityDTO.java` — sealed interface with `@JsonTypeInfo` / `@JsonSubTypes`
- `TypeADTO.java`, `TypeBDTO.java` — records implementing the sealed interface
- Compact constructors with type validation

### 2. Strategy + Registry Pattern
If multiple entity subtypes need specific validation:
- `EntityEventValidator.java` — functional interface
- `EntityValidator.java` — coordinator with `Map<String, EntityEventValidator>`
- `TypeAEventValidator.java` — annotated `@Component("ENTITY_PREFIX_TYPE_A")`

### 3. Collect-Then-Fail Validation
Always use `MessageList` + `ValidationMonad` pattern, never fail-fast:
```java
ValidationMonad.validateString(dto.getNom(), "nom")
    .notEmpty("Le nom est obligatoire")
    .maxLength(50, "Maximum 50 caractères")
    .accumulate(messages);
```

### 4. Single Table Inheritance
When entity has subtypes, use `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)` with `@DiscriminatorColumn`.

## Naming Conventions

- Entity class: PascalCase singular (`UserProfile`, not `UserProfiles`)
- Table name: snake_case plural (`user_profile`)
- Repository: `UserProfileRepository extends JpaRepository<UserProfile, Long>`
- Service: `UserProfileService`
- Controller: `UserProfileResource` (JHipster convention)

## Logging Convention

Always use `LoggerFactory`, never `@Slf4j`:
```java
private static final Logger log = LoggerFactory.getLogger(UserProfileService.class);
```

## Post-Generation Steps

After `jhipster entity` completes:
1. Run `./mvnw compile` to verify no compilation errors
2. Check generated liquibase changelog in `src/main/resources/config/liquibase/changelog/`
3. Apply architectural patterns if entity has subtypes
4. Run `./mvnw test` to verify existing tests still pass
