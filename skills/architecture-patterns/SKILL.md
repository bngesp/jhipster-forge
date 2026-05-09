---
name: JHipster Architecture Patterns
description: Activates when working on a JHipster project that requires applying architectural patterns — including after entity generation, when creating validators, DTOs, mappers, or service layers. Also activates when the user mentions Sealed Interface, Strategy pattern, validation, MapStruct, or polymorphic types in a Spring Boot/JHipster context.
---

# JHipster Architecture Patterns

Apply these patterns systematically after JHipster code generation. They enhance generated code with production-grade architecture.

## Pattern 1 — Sealed Interface + Records (Java 17+)

Apply when an entity has multiple subtypes (discriminator column).

### Entity Side (Single Table Inheritance)
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "entity_type", discriminatorType = DiscriminatorType.STRING)
@Table(name = "my_entity")
public class MyEntity extends AbstractAuditingEntity<Long> { }

@Entity
@DiscriminatorValue("TYPE_A")
public class TypeAEntity extends MyEntity {
    @Column(name = "champ_specifique_a")
    private String champSpecifiqueA;
}
```

### DTO Side (Sealed Interface + Records)
```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.EXISTING_PROPERTY, property = "typeEntity", visible = true)
@JsonSubTypes({
    @JsonSubTypes.Type(value = TypeADTO.class, name = "TYPE_A"),
    @JsonSubTypes.Type(value = TypeBDTO.class, name = "TYPE_B")
})
public sealed interface EntityDTO extends Serializable
    permits TypeADTO, TypeBDTO { }

public record TypeADTO(
    Long id,
    @NotNull TypeEntity typeEntity,
    String champSpecifiqueA
) implements EntityDTO {
    public TypeADTO {
        if (typeEntity != null && typeEntity != TypeEntity.TYPE_A) {
            throw new IllegalArgumentException("Type invalide");
        }
    }
}
```

## Pattern 2 — Strategy + Registry (Polymorphic Validation)

Apply when different entity subtypes need different validation logic.

```java
// Functional interface
@FunctionalInterface
public interface EntityEventValidator {
    void validate(MyEntity entity, MessageList messages);
}

// Coordinator with Spring auto-injection
@Component
public class EntityValidator {
    private static final Logger log = LoggerFactory.getLogger(EntityValidator.class);

    @Autowired(required = false)
    private Map<String, EntityEventValidator> eventValidators;

    public void validate(MyEntity entity) throws BusinessException {
        MessageList messages = new MessageList();
        validateCommonRules(entity, messages);

        String key = "ENTITY_PREFIX_" + entity.getTypeEntity().name();
        EntityEventValidator specific = eventValidators.get(key);
        if (specific != null) {
            specific.validate(entity, messages);
        }

        if (!messages.isEmpty()) {
            throw new BusinessException(messages.getList());
        }
    }
}

// Specific validator — naming convention is critical
@Component("ENTITY_PREFIX_TYPE_A")
public class TypeAEventValidator implements EntityEventValidator {
    @Override
    public void validate(MyEntity entity, MessageList messages) {
        if (!(entity instanceof TypeAEntity typeA)) {
            throw new IllegalArgumentException("Type invalide");
        }
        // Type-specific validation
    }
}
```

## Pattern 3 — Collect-Then-Fail Validation

Never use fail-fast validation. Always collect all errors first.

### Style Monadique (Recommended)
```java
MessageList messages = new MessageList();

ValidationMonad.validateString(dto.getNom(), "nom")
    .notEmpty("Le nom est obligatoire")
    .maxLength(50, "Maximum 50 caractères")
    .accumulate(messages);

ValidationMonad.validate(dto.getAge(), "age")
    .asBusinessRule()
    .notNull("L'âge est obligatoire")
    .min(18, "Doit être majeur")
    .accumulate(messages);

if (!messages.isEmpty()) {
    throw new BusinessException(messages.getList());
}
```

### MessageList Usage (Correct Pattern)
```java
// CORRECT
messages.add(new Message("message", "field", Level.ERROR, ControlType.FORM_VALIDATION));
messages.add(new Message("message", "field", Level.WARNING, ControlType.BUSINESS_RULE));

// WRONG — do not use
messages.add("field", Level.WARNING, ControlType.BUSINESS_RULE, "message");
```

## Pattern 4 — MapStruct with Pattern Matching

Apply when JHipster generates a mapper and the entity has subtypes.

```java
@Mapper(componentModel = "spring")
public interface MyEntityMapper {

    default MyEntity toEntity(EntityDTO dto) {
        return switch (dto) {
            case TypeADTO a -> toEntity(a);
            case TypeBDTO b -> toEntity(b);
        };
    }

    TypeAEntity toEntity(TypeADTO dto);
    TypeADTO toDto(TypeAEntity entity);
    TypeBEntity toEntity(TypeBDTO dto);
    TypeBDTO toDto(TypeBEntity entity);
}
```

## Pattern 5 — RFC7807 Exception Translation (Zalando Problem)

```java
@ExceptionHandler(BusinessException.class)
public ResponseEntity<Problem> handleBusinessException(BusinessException ex) {
    return ResponseEntity
        .status(BAD_REQUEST)
        .body(Problem.builder()
            .withType("about:blank")
            .withTitle("Validation échouée: " + ex.countErrors() + " erreur(s)")
            .withStatus(BAD_REQUEST)
            .with("messages", ex.getMessageList())
            .with("errorCount", ex.countErrors())
            .with("warningCount", ex.countWarnings())
            .build());
}
```

## Package Structure Convention

```
com.{company}.{project}
├── domain/{domaine}/
│   ├── MyEntity.java
│   ├── TypeAEntity.java
│   └── TypeBEntity.java
├── service/
│   ├── dto/{domaine}/
│   │   ├── EntityDTO.java          (sealed interface)
│   │   ├── TypeADTO.java           (record)
│   │   └── TypeBDTO.java           (record)
│   ├── mapper/
│   │   └── MyEntityMapper.java
│   └── validator/{domaine}/
│       ├── EntityEventValidator.java
│       ├── EntityValidator.java
│       ├── TypeAEventValidator.java
│       └── TypeBEventValidator.java
└── repository/
    └── MyEntityRepository.java
```

## Logging Convention (Always)

```java
// CORRECT — use in every class
private static final Logger log = LoggerFactory.getLogger(MyClass.class);

// WRONG — never use @Slf4j annotation in this project
```

## Checklist After Entity Generation

- [ ] Does the entity have subtypes? → Apply Patterns 1 + 2
- [ ] Does validation need all errors at once? → Apply Pattern 3
- [ ] Does the mapper need polymorphism? → Apply Pattern 4
- [ ] Are exceptions returned as HTTP responses? → Apply Pattern 5
- [ ] Is logging present in service and validator? → Use LoggerFactory
