---
name: forge-service
description: Generate a JHipster Spring service layer for an existing entity
argument-hint: "[EntityName]"
allowed-tools: [Bash, Read, Write, Edit]
---

Generate a Spring service for a JHipster entity.

## Steps

1. **Verify JHipster project** — check `.yo-rc.json` exists.

2. **Determine entity name**:
   - If `$ARGUMENTS` provided, use it
   - Otherwise ask: "Which entity needs a service layer?"

3. **Check if service already exists** — look in `src/main/java/**/service/`. Warn user if it already exists.

4. **Run generation**:
   ```bash
   jhipster spring-service <EntityName>
   ```
   Choose `serviceClass` (concrete class) over `serviceImpl` (interface + implementation) unless the user needs multiple implementations.

5. **Post-generation** — remind user to:
   - Move business logic from Controller to the new Service
   - Add `MessageList` + `ValidationMonad` pattern for validation
   - Use `LoggerFactory` (not `@Slf4j`)

## Notes

- `serviceClass`: single class — simpler, recommended for most cases
- `serviceImpl`: interface + implementation — use only if you need multiple service implementations
