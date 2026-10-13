---
type: "practice"
difficulty: "Intermediate"
---

# Why does storing a UUID as varchar bloat indexes and slow joins compared to a native uuid/binary type, and how would you migrate the column type in place on a large table?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Storing UUIDs as VARCHAR can lead to significant bloat in indexes and slow down join operations due to increased storage size and loss of index efficiency. To mitigate these issues, you should migrate the column type from VARCHAR to a native UUID or BINARY(16) type. The migration involves careful planning and execution to ensure minimal downtime.

#### Triaging the Question
The question is about understanding why storing UUIDs as VARCHAR can cause problems in database performance, particularly with indexes and joins. It also asks for a strategy to migrate an existing column from VARCHAR to a native UUID type on a large table. The interviewer is probing your knowledge of SQL data types, indexing strategies, and database migration practices.

#### Assumptions

You are working with a database that supports both VARCHAR and UUID/BINARY(16) data types.
The table in question contains a significant number of rows (large table).
You have the necessary permissions to alter the schema.
You want to optimize for performance while ensuring minimal disruption.

#### First-Principles Solution

Why Storing UUIDs as VARCHAR Can Be Problematic

Index Bloat: When you store a UUID as VARCHAR, each row will take up more space in the index compared to storing it as a binary type. A 128-bit UUID (in hexadecimal format) takes 36 bytes (VARCHAR(36)), while a native UUID or BINARY(16) only takes 16 bytes.

SQL
-- VARCHAR(36): 36 bytes per UUID
-- BINARY(16): 16 bytes per UUID

Index Efficiency: Storing as VARCHAR means the index will contain more non-leaf nodes, increasing the height of the tree and thus slowing down query execution.

How Storing UUIDs as VARCHAR Slows Down Joins

When performing joins on columns that are stored as VARCHAR, the database must perform additional work to compare string values. This can significantly impact performance:

SQL
-- Example join with VARCHAR UUID
SELECT * FROM table1 JOIN table2 ON table1.uuid = table2.uuid;

In contrast, if both columns were of type UUID or BINARY(16), the comparison would be much faster.

Migrating from VARCHAR to UUID/BINARY

To migrate a column from VARCHAR to a native UUID or BINARY(16), you need to follow these steps:

Step 1: Create a New Column

First, add a new column of type UUID or BINARY(16).

SQL
ALTER TABLE large_table ADD COLUMN new_uuid UUID;
Step 2: Populate the New Column

Generate new UUIDs and populate the new column. You can use functions like UNHEX for BINARY types if your database supports it.

SQL
UPDATE large_table SET new_uuid = UNHEX(REPLACE(uuid_column, '-', '')) WHERE uuid_column IS NOT NULL;
Step 3: Verify Data Integrity

Ensure that the data in both columns is consistent and the new column contains valid UUIDs.

SQL
SELECT * FROM large_table WHERE new_uuid IS NULL OR LENGTH(new_uuid) != 16;
Step 4: Drop the Old Column (if safe)

Once you are confident that all data has been correctly migrated, drop the old VARCHAR column and rename the new one to replace it.

SQL
ALTER TABLE large_table DROP COLUMN uuid_column, CHANGE new_uuid uuid_column UUID;

#### Closing Summary

Migrating a VARCHAR column storing UUIDs to a native UUID or BINARY(16) type will reduce storage requirements, improve index efficiency, and speed up join operations. The key steps involve creating a temporary column, populating it with the correct data, verifying integrity, and then safely renaming or dropping the old column.

#### Common Mistakes/Follow-Ups
Not Verifying Data Integrity: Failing to check for nulls or incorrect UUIDs after migration can lead to inconsistent data.
Ignoring Indexes: Not rebuilding indexes after the migration can leave you with a performance bottleneck.
Failing to Handle Downtime: Significant operations on large tables can impact availability; consider using online schema change tools if available.

**Comparison:**

| UUID Type | VARCHAR (36 bytes) | BINARY(16) |
|-----------|--------------------|------------|
| Storage   | 36 bytes           | 16 bytes   |

**Code:**

```sql
-- Example code snippets for migration steps
ALTER TABLE large_table ADD COLUMN new_uuid UUID;
UPDATE large_table SET new_uuid = UNHEX(REPLACE(uuid_column, '-', '')) WHERE uuid_column IS NOT NULL;
SELECT * FROM large_table WHERE new_uuid IS NULL OR LENGTH(new_uuid) != 16;
ALTER TABLE large_table DROP COLUMN uuid_column, CHANGE new_uuid uuid_column UUID;
```

**Example input / output:**

```text
-- Example input-output of migration steps

Input:
UPDATE large_table SET new_uuid = UNHEX(REPLACE(uuid_column, '-', '')) WHERE uuid_column IS NOT NULL;

Output:
Rows updated: 123456

Input:
SELECT * FROM large_table WHERE new_uuid IS NULL OR LENGTH(new_uuid) != 16;

Output:
No rows returned
```

**Data:**

```sql
-- Sample data for demonstration
INSERT INTO large_table (uuid_column) VALUES ('123e4567-e89b-12d3-a456-426614174000');
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is UUID column stored asVARCHAR?
7. Add new UUID/BINARY(16)column
8. Populate new column withexisting data
9. Verify data integrity
10. Rename/replace old column
11. Fix issues and re-runverification
12. Drop VARCHAR column
13. Continue with databaseoperations

