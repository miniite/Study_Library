---
type: "practice"
difficulty: "Deep"
---

# What is a database migration and how do you handle schema changes safely in production?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Database migrations are essential when you need to change your database schema without breaking existing functionality, especially during deployments. At its core, a migration involves altering the database schema—adding or removing columns, creating new tables, or modifying indexes—to match changes in your application's data model. Handling these safely is crucial to avoid downtime and ensure data integrity.

To handle schema changes safely, we need to consider various aspects such as backwards compatibility, data validation, and transactional integrity. We'll start by outlining the steps involved and then delve into each one from first principles.

**Code:**

```sql
-- Example migration script for adding a new column in PostgreSQL
ALTER TABLE users ADD COLUMN last_login TIMESTAMP;
```

**Example input / output:**

```text
-- Initial state (before migration)
users table:
id | name | email
1  | Alice| alice@example.com
2  | Bob  | bob@example.com

-- After adding last_login column
users table:
id | name | email       | last_login
1  | Alice| alice@example.com   |
2  | Bob  | bob@example.com     |
```

**Data:**

```

None needed here, but we might have initial data:

JSON
Copy
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com"
  }
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a migration?
5. Migrate using migrationsscripts
6. Verify the new schema anddata integrity
7. Deploy to production ifverified successfully
8. Continue with normaloperations
9. Monitor for issuespost-deployment

