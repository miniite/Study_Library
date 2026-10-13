---
type: "practice"
difficulty: "Intermediate"
---

# How do you design a schema-versioned migration workflow across a team of many engineers so migrations are ordered, reversible, and safe to run against production with automated CI?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To design a robust schema-versioned migration workflow in a multi-engineer environment, we need a systematic approach that ensures migrations are ordered, reversible, and safe to run against production. This involves using version-controlled SQL scripts, a clear versioning strategy, CI/CD integrations, and a migration management tool.

#### Triage

The question is asking for a comprehensive schema migration workflow that can be safely applied across multiple engineers in a development team. The core elements the interviewer is probing are:

Version Control: Ensuring migrations are tracked using version numbers.
Ordering: Automating the order of migrations to avoid conflicts and ensure dependencies are respected.
Reversibility: Making each migration reversible, allowing rollback if necessary.
Safety in Production: Ensuring that only safe and tested migrations are run on production environments.

#### First Principles Solution

Versioned SQL Scripts:

Use a versioning strategy like v1, v2, etc., or follow semver (e.g., 0.1.0).
Store migration scripts in a source control system like Git, ensuring all team members can access and modify them.

Migration Management Tool:

Implement a tool that manages migrations based on version numbers.
The tool should be able to detect dependencies between migrations and ensure they are applied in the correct order.

CI/CD Integration:

Integrate the migration management tool with CI pipelines, so every time new changes are committed, the latest migration scripts are tested.
Run unit tests (e.g., schema validation) for each migration script before it is deployed to production.

Rollback Mechanism:

For each migration, create a corresponding rollback script that can undo the change.
The rollback scripts should be versioned and applied in reverse order if necessary.

Production Environment Safety:

Before deploying migrations to production, set up staging environments to test them first.
Use a blue-green deployment strategy to minimize downtime during migration.
Detailed Steps:

Define the Versioning Strategy:

PLAINTEXT
v0.1.0
|--- create_users_table.sql
|--- add_user_email_column.sql
|--- remove_old_sign_up_flow.sql
Use semantic versioning or a simpler versioning strategy based on your needs.

Create Migration Scripts:

SQL
-- v1:create_users_table.sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  password_hash BYTEA NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- v2:add_user_email_column.sql
ALTER TABLE users ADD COLUMN name VARCHAR(100);

Migration Management Tool:

Use a tool like Flyway, Liquibase, or create a custom script.
BASH
flyway migrate # Applies all missing migrations in order
flyway repair  # Checks and repairs the database schema
flyway rollback # Rolls back to a specific version if needed

CI/CD Integration:

Set up CI pipelines (e.g., using GitHub Actions, Jenkins) that run tests on migration scripts.
YAML
jobs:
  - name: Test Migrations
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Dependencies
        run: npm install --save-dev @flyway/flyway-cli
      - name: Run Migrations
        run: flyway migrate --url=jdbc:mysql://localhost:3306/dev --user=root --password=root

Rollback Mechanism:

SQL
-- v1:rollback_create_users_table.sql
DROP TABLE users;

-- v2:rollback_add_user_email_column.sql
ALTER TABLE users DROP COLUMN name;

Staging Environments:

Deploy the latest migrations to a staging environment.
Test thoroughly in the staging environment before moving to production.

#### Closing Summary

By following these steps, we can ensure that schema migrations are ordered, reversible, and safe to run against production. The key is using version control, automated tools, CI/CD integrations, and thorough testing.

#### Common Mistakes/Follow-Ups
Migrating without proper rollback scripts: Always have a plan for reverting changes.
Not testing in staging environments: Running migrations directly on production can lead to downtime or data loss.
Ignoring dependencies between migrations: This can cause conflicts and incomplete schema updates.

**Code:**

```yaml
jobs:
  - name: Test Migrations
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Dependencies
        run: npm install --save-dev @flyway/flyway-cli
      - name: Run Migrations
        run: flyway migrate --url=jdbc:mysql://localhost:3306/dev --user=root --password=root
```

**Flow:**

1. Source Control
2. CI Pipeline Trigger
3. Test Migrations
4. Migrate to Staging
5. Migrate to Production

