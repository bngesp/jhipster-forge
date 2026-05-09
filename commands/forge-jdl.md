---
name: forge-jdl
description: Create or import a JDL file to generate multiple JHipster entities and relationships at once. Supports natural language description or direct JDL editing.
argument-hint: "[jdl-file.jdl | natural language description]"
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep]
---

Create and import a JDL model into the current JHipster project.

## Steps

1. **Verify JHipster project** — check `.yo-rc.json` exists at project root.

2. **Determine input mode**:
   - If `$ARGUMENTS` ends with `.jdl`: read and import the specified file
   - If `$ARGUMENTS` is a description: use it as the domain description (natural language mode)
   - If no argument: ask "Describe your domain (entities, fields, relationships) or provide a .jdl filename"

3. **Natural language mode** (when user describes domain):
   - Extract entities and their fields
   - Identify relationships and cardinalities
   - For complex domains, ask one clarifying question at a time:
     - "Is [EntityA]–[EntityB] relationship bidirectional?"
     - "Should [Entity] have pagination?"
   - Generate JDL file content
   - Save as `model.jdl` (or ask user for filename)
   - Show full JDL to user for review
   - Ask: "Shall I import this JDL now?"

4. **Import JDL**:
   ```bash
   jhipster import-jdl <filename>.jdl
   ```

5. **Post-import**:
   - Run `./mvnw compile`
   - Report generated entities and any errors
   - Ask: "Should I apply architectural patterns (Sealed Interface, Strategy+Registry) to any of these entities?"

## JDL Template

When generating JDL from natural language, always include:
```jdl
dto * with mapstruct
service * with serviceClass
paginate <Entity> with pagination
```

Unless the user specifies otherwise.

## Notes

- JDL import replaces running `jhipster entity` N times — much faster for multiple entities
- Relationships in JDL automatically generate the correct JPA annotations
- Always show generated JDL to user before importing
