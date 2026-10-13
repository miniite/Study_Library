---
type: "practice"
difficulty: "Deep"
---

# How would you design a deploy so that a failed migration can be detected and the migration rolled back before the new app version starts receiving traffic, rather than after?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Database Migrations in Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Database_Migrations_in_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that a database migration failure is caught early in the deployment pipeline, allowing us to rollback changes and prevent the new application version from receiving traffic until the issue is resolved. This involves designing a robust CI/CD pipeline with specific checks for migrations. Here’s how you can achieve it:

#### Overview
We will design a CI/CD pipeline that includes automated tests for migrations at various stages, ensuring they are run in a non-production environment before deployment. If any migration fails, the pipeline should automatically roll back to the previous version.

#### Triage
The core of this question is about ensuring data integrity and rollback capability during deployments. We need to assume a multi-stage deployment process with clear separation between development and production environments. Additionally, we must consider how migrations are handled in our CI/CD pipeline.

#### First-Principles Solution

Environment Staging: Set up separate staging environments for each environment (e.g., dev, qa, staging) that mimic the production setup as closely as possible.
Migration Testing: Implement automated migration tests that run against these staging databases before promoting to the next stage. If a migration fails, it should not proceed further in the pipeline.
Rollback Mechanism: Design a rollback strategy that can be triggered automatically if any migration step fails. This could involve restoring the database from a backup or reverting to the previous version of the application.

Detailed Steps:

Environment Setup: Ensure each environment has its own database, which mirrors the production setup.
Pipeline Stages: Define clear stages in your CI/CD pipeline (e.g., Build, Test, Deploy) and integrate migration testing into the "Test" stage.
Migration Testing Script: Write a script that automates running migrations against the staging database. This script should check for errors or failures during execution.
Failure Handling: If any part of the migration fails, immediately trigger a rollback process. For example, restore the previous version’s database dump and revert to the last stable application state.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Summary: By integrating automated tests for migrations at each stage, we can catch issues early and ensure that only fully validated code reaches production.
Mistakes to Avoid: Failing to set up proper staging environments or skipping automated migration testing can lead to critical failures once the application is live. Ensure thorough planning and testing before deployment.

**Comparison:**

```
Environment	Database URL
Dev	localhost:5432/dev_db
QA	localhost:5432/qa_db
Staging	localhost:5432/staging_db
```

**Code:**

```python
# Example of a simple migration test script in Python
def run_migrations(database_url):
    # Code to connect to database using URL
    with DatabaseConnection(database_url) as db:
        try:
            # Run migrations
            execute_migrations(db)
            return True
        except MigrationError:
            return False

def execute_migrations(db_connection):
    # Implement the logic to run migrations here
    pass
```

**Example input / output:**

```text
Input:
- Migrations run against staging database URL

Output:
- True if migrations succeed
- False and rollback triggered if any step fails
```

**Data:**

```yaml
environments:
  - name: dev
    database_url: "localhost:5432/dev_db"
  - name: qa
    database_url: "localhost:5432/qa_db"
  - name: staging
    database_url: "localhost:5432/staging_db"
```

**Flow:**

1. Pipeline Starts
2. Build
3. Test Migrations
4. Deploy to Staging
5. Promote to QA
6. Staging Tests Pass
7. Promote to Production
8. Migrations Fail
9. Rollback to Previous Version

