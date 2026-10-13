---
type: "practice"
difficulty: "Deep"
---

# Why is it important that the migration currently deployed to production always corresponds to a specific commit in Git?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Versioning SQL and Migrations](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that any issue or regression occurs during the development of new features, we need clear and reliable ways to identify and revert to previous states of our application. One key practice is ensuring that the migration currently deployed to production always corresponds to a specific commit in Git. This ensures that the exact codebase used for deployment is traceable and reproducible.

This question delves into why tracking migrations via specific commits is essential, particularly from the perspective of a fresher or mid-level database developer. The core idea revolves around maintaining a stable baseline of our application and enabling seamless rollback mechanisms in case something goes wrong.

**Comparison:**

```
Migration	Date	Commit SHA
Initial	2023-09-01	a1b2c3d4e5f6g7h8i
Email	2023-09-15	jk8l9m0n1o2p3q4r
```

**Code:**

```
Example migration code snippet

def upgrade(connection): with connection.cursor() as cursor: cursor.execute("ALTER TABLE users ADD COLUMN email_verified BOOLEAN")

CODE
Copy
```sql
-- Database schema change for adding a new column to the 'users' table
ALTER TABLE users ADD COLUMN email_verified BOOLEAN;
```

**Example input / output:**

```
Sample input for migration script:
PYTHON
Copy
# migration_20230915_v1.py
def upgrade(connection):
    with connection.cursor() as cursor:
        cursor.execute("ALTER TABLE users ADD COLUMN email_verified BOOLEAN")

def downgrade(connection):
    with connection.cursor() as cursor:
        cursor.execute("ALTER TABLE users DROP COLUMN email_verified")
Output for migration execution:
SHELL
Copy
$ python3 migration_20230915_v1.py
(Database successfully updated)
```

**Data:**

```
schema_changes.sql: Example of database schema changes.
SQL
Copy
-- Initial migration
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

-- Migration to add new column
ALTER TABLE users ADD COLUMN email_verified BOOLEAN;
commit_message.txt: Example of a commit message.
CODE
Copy
feat: Add email verification flag to the user table

This change adds an 'email_verified' field to the 'users' table, allowing us to track whether a user has verified their email address.
```

**Flow:**

1. Yes
2. No
3. Initial Migration
4. Is this commit deployed?
5. Deployed Successfully
6. Migrated Database Schema
7. Revert to Previous State
8. Identify Previous Commit
9. Migrate Back

