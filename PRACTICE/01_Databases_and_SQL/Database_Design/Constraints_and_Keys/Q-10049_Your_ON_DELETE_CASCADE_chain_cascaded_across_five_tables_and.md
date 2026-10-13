---
type: "practice"
difficulty: "Deep"
---

# Your ON DELETE CASCADE chain cascaded across five tables and locked the app for 30 seconds. How would you redesign to avoid runaway cascade deletes?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Referential Actions](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a ON DELETE CASCADE chain locks an application for 30 seconds, it indicates a potential issue with the database schema or its implementation. To address this, we need to redesign the schema to prevent runaway cascade deletes while maintaining data integrity.

To start, let’s quickly summarize what the problem is: A cascading delete in your database locked the application for 30 seconds due to a large number of rows being deleted across multiple tables. This is likely because the ON DELETE CASCADE constraint was set up to propagate deletions through a chain of related tables.

Now, we need to triage the issue and make some assumptions:

The lock time suggests that a significant number of rows were being deleted.
There are five tables involved in this cascading deletion.
The application performance was significantly impacted during this operation.

#### Triage
and Assumptions

Assume that the schema is as follows:

Table A (Parent) with ON DELETE CASCADE to Table B, Table C, Table D, and Table E.
Each of these tables has their own cascading delete rules to other related tables.

Given this, we need to consider the following steps:

#### First-Principles Solution
Identify the Bottleneck: Understand which table(s) are causing the delay in the cascade process.
Optimize the Schema:
Use a ON DELETE SET NULL or ON DELETE NO ACTION on intermediate tables to prevent immediate cascading.
Implement a batch deletion strategy where rows can be marked for deletion without immediate cascading, and then run the deletions in batches.
Detailed Steps
Analyze the Impact: Identify which table is locking the application by monitoring the execution plan and identifying bottlenecks.
Modify the Schema:
Change ON DELETE CASCADE to ON DELETE SET NULL or ON DELETE NO ACTION on intermediate tables.
For example, if Table B has a foreign key referencing Table A, change it from ON DELETE CASCADE to ON DELETE SET NULL.
Implement Batch Processing:
Instead of deleting rows in one go, mark them as deleted and run batch deletions.
Use SQL transactions to ensure that the process is atomic but spread over multiple operations.
Sample Code
SQL
-- Example: Changing ON DELETE CASCADE to ON DELETE SET NULL
ALTER TABLE TableB
ADD CONSTRAINT fk_TableB_TableA FOREIGN KEY (TableAId) REFERENCES TableA(Id)
ON DELETE SET NULL;

-- Mark rows for deletion instead of immediate cascade
UPDATE TableB
SET Deleted = 1
WHERE SomeCondition;

-- Batch delete in a scheduled task or background process
DELETE FROM TableB
WHERE Deleted = 1 AND SomeOtherCondition;

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Overusing ON DELETE CASCADE without considering the implications.
Not monitoring performance when making schema changes.

#### Follow-Ups
Investigate if there are alternative designs that can reduce the need for cascading deletes, such as using junction tables or other normalization techniques.

#### Cons
ider implementing a retry mechanism to handle potential deadlocks in batch processing.

**Code:**

```sql
-- Example code for changing foreign key constraints
ALTER TABLE TableB
ADD CONSTRAINT fk_TableB_TableA FOREIGN KEY (TableAId) REFERENCES TableA(Id)
ON DELETE SET NULL;

-- Example of marking rows for deletion
UPDATE TableB
SET Deleted = 1
WHERE SomeCondition;
```

