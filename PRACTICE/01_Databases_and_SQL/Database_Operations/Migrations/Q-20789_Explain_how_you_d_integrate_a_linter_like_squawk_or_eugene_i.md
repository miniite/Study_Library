---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd integrate a linter like squawk or eugene into CI to statically reject unsafe Postgres migrations, and where static analysis falls short.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate a linter like Squawk or Eugene into our Continuous Integration (CI) pipeline to statically reject unsafe PostgreSQL migrations, we need to ensure that migration scripts are checked against predefined rules before they reach production. This approach leverages static analysis to catch issues early in the development cycle. The key steps involve setting up a CI job that runs the linter on migration files and failing the build if any violations are detected.

The static analysis provided by these tools is extremely valuable, as it can identify potential errors such as syntax issues, incorrect data types, or problematic SQL constructs that might fail at runtime. However, static analysis has its limitations and cannot catch every possible issue, particularly those related to database schema dependencies or complex logic.

**Code:**

```yaml
# Example CI YAML configuration for integrating Squawk linter with PostgreSQL migrations
stages:
  - lint

lint:
  stage: lint
  script:
    - docker run --rm squawk/postgres-lint < path/to/migrations/*.sql
```

**Example input / output:**

```
Example migration file (migrations/001_create_users.sql)

CREATE TABLE users ( id SERIAL PRIMARY KEY, email VARCHAR(255) NOT NULL UNIQUE, # Squawk may flag this as a potential issue due to lack of constraints password VARCHAR(60) NOT NULL, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP );

CODE
Copy

```plaintext
# Linter output (squawk output)
Linting migrations/001_create_users.sql...
Warning: The `email` column in the `users` table should have a length constraint to prevent excessively long emails.
Error: The `password` column in the `users` table should be hashed before storing, as plain text passwords are insecure.
```

**Flow:**

1. CI Job Starts
2. Run Squawk Linter onMigration Files
3. Fail Build if Violations Detected

