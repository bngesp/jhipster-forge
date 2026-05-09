---
name: JHipster JDL Modeling
description: Activates when a user wants to create or edit a JDL file, model a domain with multiple entities and relationships, import a JDL file into a JHipster project, or describe their data model in natural language for JHipster generation. Also activates for questions about JHipster Domain Language syntax.
---

# JHipster JDL Modeling

## JDL Overview

JDL (JHipster Domain Language) is JHipster's DSL for defining the full domain model in a single file. One JDL file can replace running `jhipster entity` multiple times.

Import command: `jhipster import-jdl model.jdl`

## Workflow A — Natural Language Description

When the user describes their domain in natural language:
1. Extract entities, fields, and relationships from the description
2. Identify cardinalities (one-to-many, many-to-many, etc.)
3. Generate the JDL file
4. Show it to the user for review
5. Ask: "Shall I import this JDL into your project?"
6. If yes: run `jhipster import-jdl <filename>.jdl`

When the domain is complex, ask clarifying questions one at a time:
- "Does [Entity] belong to a user, or is it standalone?"
- "Is the relationship between [A] and [B] bidirectional?"
- "Should [Entity] have pagination?"

## Workflow B — Direct JDL Editing

When the user provides or asks to modify a JDL file:
1. Read the existing JDL if it exists
2. Apply the requested changes
3. Validate syntax before importing
4. Run `jhipster import-jdl <filename>.jdl`

## JDL Syntax Reference

### Entity Definition
```jdl
entity UserProfile {
    firstName String required maxlength(50)
    lastName String required maxlength(50)
    email String required unique pattern(/^[^@]+@[^@]+\.[^@]+$/)
    birthDate LocalDate
    salary BigDecimal min(0)
    status UserStatus
    active Boolean required
}
```

### Enum Definition
```jdl
enum UserStatus {
    ACTIVE, INACTIVE, PENDING, SUSPENDED
}
```

### Relationships
```jdl
// ManyToOne (most common)
relationship ManyToOne {
    Order{customer required} to Customer
}

// OneToMany
relationship OneToMany {
    Customer{orders} to Order{customer}
}

// ManyToMany
relationship ManyToMany {
    Product{categories} to Category{products}
}

// OneToOne
relationship OneToOne {
    UserProfile{user(login)} to User
}
```

### Pagination
```jdl
paginate UserProfile, Order with pagination
paginate Product with infinite-scroll
paginate Category with no
```

### DTOs
```jdl
dto UserProfile, Order with mapstruct
```

### Service Layer
```jdl
service UserProfile with serviceClass
service Order with serviceImpl
```

### Search
```jdl
search UserProfile, Order with elasticsearch
```

### Filtering
```jdl
filter UserProfile, Order
```

### Microservices
```jdl
// Declare which microservice owns which entity
microservice Product, Category with productService
microservice Order with orderService

// Client-side gateway
clientRootFolder Product, Category with productService
```

## Validation Rules in JDL

| Validator | Applies To | Example |
|---|---|---|
| required | all | `name String required` |
| unique | all | `email String unique` |
| minlength(n) | String | `code String minlength(3)` |
| maxlength(n) | String | `name String maxlength(100)` |
| pattern(/regex/) | String | `phone String pattern(/^\d{10}$/)` |
| min(n) | numeric | `age Integer min(0)` |
| max(n) | numeric | `age Integer max(150)` |

## JDL Best Practices

1. **One JDL file per bounded context** — keeps models manageable
2. **Always declare DTOs with mapstruct** — generates MapStruct mappers automatically
3. **Declare service layer** — avoids direct repository calls from controllers
4. **Add pagination for all list views** — prevents performance issues in production
5. **Use filter** for entities that need dynamic querying (JHipster generates JPA Criteria API)
6. **Name relationships explicitly** — `Order{customer required}` is clearer than `Order to Customer`

## Post-Import Steps

After `jhipster import-jdl` completes:
1. Run `./mvnw compile` — verify no compilation errors
2. Check `src/main/resources/config/liquibase/changelog/` for new changesets
3. Run `./mvnw test` — all existing tests must pass
4. Apply architectural patterns where needed (see architecture-patterns skill)
