---
type: "practice"
difficulty: "Deep"
---

# How would you create an index on a huge production table without blocking writes, and what are the caveats of doing so?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create an index on a huge production table without blocking writes, we'll leverage SQL Server's  ONLINE = ON feature, which allows DDL operations to proceed in parallel with transactions. This is crucial for minimizing downtime and ensuring that the database remains available for read and write operations during the index creation process.

#### Triaging the Question

The question essentially asks how to add an index to a large table without causing significant performance degradation or blocking writes. The interviewer is probing your understanding of SQL Server's transactional behavior, concurrency control mechanisms, and best practices in database administration.

#### Assumptions
We are working with a production database where minimal downtime is critical.
The table has existing data that needs to remain accessible during the index creation.
We need to ensure that writes can still proceed without being blocked or significantly delayed.

#### First-Principles Solution

To add an index without blocking writes, we will use the CREATE INDEX statement with the ONLINE = ON option. This feature allows the database engine to create the index in phases while allowing reads and writes to continue.

Backup the Existing Table (Optional but Recommended): Before making any structural changes, it’s a good practice to back up the existing table.
Create a New Index: Use the CREATE INDEX statement with the ONLINE = ON option.
Analyze Performance and Adjust if Necessary: Monitor the performance during the index creation process.

Here is an example of how you might write this query:

SQL
-- Example: Creating an index on a large table without blocking writes
BEGIN TRANSACTION;

CREATE INDEX idx_column_name
ON your_table (column_name)
WITH (ONLINE = ON);

COMMIT;

#### Closing Summary
and Common Mistakes/Follow-Ups

While ONLINE is a powerful feature, it’s important to monitor the performance closely. The operation might still cause some degree of locking or blocking, especially if there are concurrent heavy transactions.

#### Common Mistakes
Insufficient Monitoring: Not keeping an eye on the performance can lead to unexpected issues.
Failing to Back Up Data: In case of a failure, having no backup means data loss.
Incorrect Indexing Strategy: Creating indexes that do not align with query patterns may still degrade read performance.
Triaging Follow-Ups
What if the ONLINE Option is Not Available?: Discuss alternatives like using CREATE INDEX ... ON TABLE... to create the index on a temporary table and then swapping tables.
How Do You Ensure Data Consistency During Index Creation?: Explain how WITH (CHECK_CONSTRAINTS, ONLINE = ON) ensures that constraints are enforced during the creation process.
What Are the Performance Impacts of Creating an Index on a Large Table?: Discuss potential bottlenecks and how to mitigate them.

**Code:**

```sql
-- Example query for creating an index online
BEGIN TRANSACTION;

CREATE INDEX idx_column_name
ON your_table (column_name)
WITH (ONLINE = ON);

COMMIT;
```

