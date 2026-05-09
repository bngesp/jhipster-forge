---
name: forge-liquibase
description: Generate a Liquibase diff changelog after manual database schema changes in a JHipster project
argument-hint: ""
allowed-tools: [Bash, Read]
---

Generate a Liquibase diff changelog for schema changes in a JHipster project.

## Steps

1. **Verify JHipster project** — check `.yo-rc.json` and `src/main/resources/config/liquibase/` exist.

2. **Check database is running** — the diff command requires a live database connection:
   ```bash
   # Check if Docker is running the DB
   docker ps | grep postgres
   ```
   If not running, ask user to start the database first.

3. **Run Liquibase diff**:
   ```bash
   ./mvnw liquibase:diff
   ```

4. **Review generated changelog** — show the user the new file in `src/main/resources/config/liquibase/changelog/` and explain what changed.

5. **Verify master changelog** — confirm the new file is referenced in `src/main/resources/config/liquibase/master.xml`.

6. **Run a test** to verify the changelog applies cleanly:
   ```bash
   ./mvnw liquibase:update -Dspring.profiles.active=dev
   ```

## Common Scenarios

- **Added a column manually**: Run diff to capture it in a changeset
- **Renamed a table/column**: Diff won't detect renames automatically — create changeset manually
- **Added an index**: Run diff to capture it

## Notes

- Never edit existing changesets — Liquibase checksums will fail
- Always add new changesets, never modify old ones
- Use `runOnChange="true"` only for views and stored procedures
