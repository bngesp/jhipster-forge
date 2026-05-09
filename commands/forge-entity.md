---
name: forge-entity
description: Generate a JHipster entity with fields, relationships, and architectural patterns applied automatically
argument-hint: "[EntityName]"
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep]
---

Generate a JHipster entity in the current project.

## Steps

1. **Verify JHipster project** — run `jhipster info` or check `.yo-rc.json` exists. If not found, tell the user this is not a JHipster project.

2. **Determine entity name**:
   - If `$ARGUMENTS` is provided, use it as the entity name (ensure PascalCase, singular)
   - If not provided, ask: "What is the entity name? (PascalCase, singular — e.g. `UserProfile`)"

3. **Collect entity definition** by asking:
   - Fields: "List your fields in format `name:type:validations` (one per line, or comma-separated)"
     - Offer examples: `firstName:String:required,maxlength(50)`, `birthDate:LocalDate`, `status:StatusEnum`
   - Relationships: "Any relationships with other entities? (e.g. ManyToOne to Customer)"
   - Pagination: "Pagination type? (pagination / infinite-scroll / no)"
   - DTO: "Generate DTO with MapStruct? (yes/no)"

4. **Confirm** — show a summary and ask for confirmation before running.

5. **Run generation**:
   ```bash
   cd <project-root> && jhipster entity <EntityName>
   ```
   Respond to JHipster prompts using collected information.

6. **Post-generation** — ask the user:
   - "Does this entity have subtypes (discriminator)? I can apply the Sealed Interface + Strategy pattern automatically."
   - If yes: apply architecture-patterns skill

7. **Verify** — run `./mvnw compile` and report any errors.

## Notes

- Entity name must be PascalCase and singular (`UserProfile`, not `userprofiles`)
- JHipster generates: Entity, Repository, Service, Controller, DTO, Mapper, Liquibase changelog, Angular CRUD
- Always run compile after generation to catch issues early
