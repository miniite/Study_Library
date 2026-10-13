---
type: "practice"
difficulty: "Intermediate"
---

# A UNIQUE index rebuild during an online DDL failed and left a duplicate. How do you find and resolve duplicate keys before re-adding the constraint?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Unique and Deferred Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to identify duplicates in the column that was supposed to be unique but now has a duplicate value due to an unsuccessful index rebuild. We'll then remove or correct these duplicates before re-applying the unique constraint.

#### Triage
and Assumptions:
The failure during the UNIQUE index rebuild likely caused a violation, leaving behind one or more duplicate values.
The database is online, meaning operations must be performed without significant downtime.
There's no existing backup of the data to restore from. We will have to find and correct the issue in place.
Step-by-Step Solution:
Identify Duplicate Values:
Use SQL queries to find rows with duplicate values in the column that should be unique.
Mark or Delete Duplicates:
Decide on a strategy for handling duplicates (e.g., delete one of them, update the data).
Reapply the Constraint:
Once duplicates are resolved, re-add the UNIQUE constraint to ensure data integrity.
Finding and Resolving Duplicates

To find duplicate values in an SQL database, you can use a query that groups by the column in question and filters for those with more than one occurrence. Here’s how:

SQL
SELECT column_name, COUNT(*) as count
FROM your_table
GROUP BY column_name
HAVING count > 1;

This will give you a list of all values that appear more than once.

Handling Duplicates
Option 1: Delete One of Each Duplicate

If the application logic allows, you can delete one instance of each duplicate value. Here's an example:

SQL
DELETE FROM your_table
WHERE column_name IN (
    SELECT column_name
    FROM (
        SELECT column_name
        FROM your_table
        GROUP BY column_name
        HAVING COUNT(*) > 1
    ) AS duplicates
);
Option 2: Update Data to Resolve Duplicates

Alternatively, if the application can tolerate a temporary change in data integrity (e.g., for correcting an entry), you might update one of the duplicate values to be unique. For example:

SQL
UPDATE your_table
SET column_name = 'new_value'
WHERE id IN (
    SELECT id
    FROM (
        SELECT id, ROW_NUMBER() OVER(PARTITION BY column_name ORDER BY id) as rn
        FROM your_table
    ) AS duplicates
    WHERE rn > 1;
);
Reapplying the Unique Constraint

Once you've resolved the duplicates, you can safely reapply the unique constraint. If the table was in a transaction that failed, you might need to commit any pending changes first.

SQL
ALTER TABLE your_table ADD CONSTRAINT unique_column_name UNIQUE (column_name);

#### Closing Summary
and Common Mistakes
Mistake: Not handling duplicates correctly could lead to data integrity issues or application crashes.
Mistake: Failing to mark the constraint as NOT NULL if required, leading to future violations.
Common Follow-Up: Interviewers might ask about the impact on performance and suggest alternative strategies like using a temporary table for deduplication.

**Comparison:**

```
id	column_name
1	valueA
2	valueB
3	valueC
id	column_name
1	valueA
4	valueB_new
5	valueC
```

**Code:**

```sql
-- Finding duplicates query
SELECT column_name, COUNT(*) as count
FROM your_table
GROUP BY column_name
HAVING count > 1;

-- Deleting duplicate rows
DELETE FROM your_table
WHERE column_name IN (
    SELECT column_name
    FROM (
        SELECT column_name
        FROM your_table
        GROUP BY column_name
        HAVING COUNT(*) > 1
    ) AS duplicates
);

-- Updating data to resolve duplicates
UPDATE your_table
SET column_name = 'new_value'
WHERE id IN (
    SELECT id
    FROM (
        SELECT id, ROW_NUMBER() OVER(PARTITION BY column_name ORDER BY id) as rn
        FROM your_table
    ) AS duplicates
    WHERE rn > 1;
);

-- Reapplying the unique constraint
ALTER TABLE your_table ADD CONSTRAINT unique_column_name UNIQUE (column_name);
```

**Example input / output:**

```text
Input:
- SQL table with column 'column_name' having duplicates

Output:
1. Identifying duplicate values in the 'column_name' column.
2. Deleting or updating one of each duplicate value to resolve them.
3. Reapplying the unique constraint on the 'column_name' column.
```

**Data:**

```json
{
  "table": {
    "id": ["int", "Primary Key"],
    "column_name": ["string"]
  },
  "duplicates": [
    {"id": 1, "column_name": "valueA"},
    {"id": 2, "column_name": "valueB"},
    {"id": 3, "column_name": "valueC"}
  ]
}
```

**Flow:**

1. Delete
2. Update
3. Identify Duplicates
4. Delete or Update?
5. Mark as Deleted
6. Update Data
7. Reapply Unique Constraint
8. Reapply Unique Constraint

