---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you'd add a NOT NULL column with a default to a 200M-row table in Postgres without locking writes for minutes.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a NOT NULL column with a default value to a large table like a 200M-row table in PostgreSQL, the primary goal is to ensure minimal downtime and zero lock contention on write operations. The key steps involve:

Preparing the schema update.
Adding the new column with a default value temporarily.
Updating existing rows to set non-null values.
Dropping the old columns if necessary.

This process ensures that writes remain available while updates are performed in a controlled manner, minimizing disruption and downtime.

**Code:**

```sql
-- Step 1: Prepare the schema update
ALTER TABLE my_table ADD COLUMN new_column VARCHAR(255) DEFAULT 'default_value';
```

**Example input / output:**

```text
-- Initial state of the table
SELECT * FROM my_table LIMIT 10;

-- After adding the column with default value
ALTER TABLE my_table ADD COLUMN new_column VARCHAR(255) DEFAULT 'default_value';
SELECT * FROM information_schema.columns WHERE table_name = 'my_table';

-- Update existing rows to non-null values (example for one row)
UPDATE my_table SET new_column = 'non_null_value' WHERE id = 1;

-- Final state of the table
SELECT * FROM my_table LIMIT 10;
```

**Flow:**

1. Preparation
2. Add column with default
3. Update rows
4. Verify schema update
5. Dropping old columns(optional)

