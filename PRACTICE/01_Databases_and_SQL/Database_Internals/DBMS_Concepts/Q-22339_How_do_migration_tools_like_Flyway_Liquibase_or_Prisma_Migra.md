---
type: "practice"
difficulty: "Intermediate"
---

# How do migration tools like Flyway, Liquibase, or Prisma Migrate keep schema changes ordered and repeatable?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases in a team environment, it's crucial to maintain an ordered and repeatable process for applying schema changes. Tools like Flyway, Liquibase, and Prisma Migrate help manage these changes effectively by ensuring that each migration is applied in the correct order and can be rolled back if necessary.

Flyway and Liquibase both rely on versioning to keep track of migrations, but they differ slightly in their implementation and features. Let's dive into how Flyway ensures ordered and repeatable schema changes from first principles.

