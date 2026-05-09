---
name: forge-controller
description: Generate a JHipster Spring REST controller (resource) for an existing entity
argument-hint: "[EntityName]"
allowed-tools: [Bash, Read, Write, Edit]
---

Generate a Spring REST controller for a JHipster entity.

## Steps

1. **Verify JHipster project** — check `.yo-rc.json` exists.

2. **Determine entity name**:
   - If `$ARGUMENTS` provided, use it
   - Otherwise ask: "Which entity needs a REST controller?"

3. **Check existing controller** — look in `src/main/java/**/web/rest/`. Warn if `<EntityName>Resource.java` already exists.

4. **Run generation**:
   ```bash
   jhipster spring-controller <EntityName>
   ```

5. **Post-generation** — remind user:
   - JHipster names controllers `<EntityName>Resource.java` (not `Controller`)
   - Controller should delegate to Service, not contain business logic
   - Use `ResponseEntity<?>` as return type
   - Catch `BusinessException` and return RFC7807 Problem response

## Notes

- JHipster convention: controllers are named `*Resource.java`
- Generated controller includes CRUD endpoints: GET (list + byId), POST, PUT, PATCH, DELETE
