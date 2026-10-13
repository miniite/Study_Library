---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema migration workflow across a team so migrations are ordered, reversible, and safe to run in CI against a prod-shaped dataset?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust schema migration workflow that ensures migrations are ordered, reversible, and safe to run in CI against a prod-shaped dataset, we need to focus on several key aspects: versioning, dependency management, testing, and rollback capabilities. This approach will ensure that our migrations are well-organized, maintainable, and can be executed reliably.

#### Triage

The question is asking how to structure the migration process so it's both safe and efficient. The core of this involves:

Versioning: Each migration should have a unique identifier.
Dependency Management: Migrations must be ordered based on their dependencies.
Reversibility: Each migration should include an "undo" step to facilitate rollbacks.
Safety in CI: Ensuring migrations can be tested against a prod-like environment without disrupting live data.

#### Assumptions
We are using a relational database (e.g., PostgreSQL, MySQL).
Migrations are written as SQL scripts or a migration framework like Liquibase.
We have a CI/CD pipeline that supports running these migrations.
Step-by-Step Solution

Versioning and Naming Conventions Each migration script should be uniquely named to represent its version number and description. For example:

SQL
-- 20230901_01_create_users_table.sql
-- 20230905_02_add_email_column_to_users.sql

Dependency Management Migrations should be organized in a specific order that respects their dependencies. This can be enforced using version numbers or by defining explicit dependencies in each migration file.

Reversibility Each migration script should include an "undo" part, which is executed when the migration needs to be rolled back. For example:

SQL
-- 20230901_01_create_users_table.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);

-- 20230905_02_add_email_column_to_users.sql
ALTER TABLE users ADD COLUMN email VARCHAR(100);

-- Undo scripts can be stored in a separate file or included at the end of each migration.

Safety in CI To ensure migrations are safe to run in CI:

Use a staging environment that mirrors production as closely as possible.
Implement pre-flight checks, such as database health checks and validation scripts.
Use transactional support where possible to ensure atomicity.

#### Example
Workflow

Source Control Management Ensure all migrations are checked into version control (e.g., Git) with clear commit messages that describe the changes.

Migration Execution Script Write a script that runs migrations in order, respecting dependencies:

BASH
# run-migrations.sh
cd /path/to/migrations
./migrate --order=asc --env=staging

Rollback Mechanism Implement a rollback mechanism to revert changes if something goes wrong during CI execution or in production.

SQL
-- 20230905_02_add_email_column_to_users.sql
ALTER TABLE users ADD COLUMN email VARCHAR(100);

-- 20230906_01_drop_email_column_from_users.sql
DROP COLUMN IF EXISTS email;

#### Closing Summary
and Common Mistakes
Versioning: Forgetting to add version numbers or using inconsistent naming conventions.
Dependency Management: Overlooking dependencies, leading to errors during execution.
Reversibility: Not including undo scripts, making it difficult to roll back migrations if necessary.
Safety in CI: Running migrations against a live production database without proper safeguards.

By following these steps and ensuring thorough testing and safety measures, we can build a robust schema migration workflow that is both safe and efficient for the team.

