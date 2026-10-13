---
type: "practice"
difficulty: "Expert"
---

# A backfill of 500M rows in one UPDATE statement blew up the WAL and replication lag spiked. How would you have chunked it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a massive backfill operation without overwhelming the Write-Ahead Log (WAL) and causing excessive replication lag, we need to break down the task into smaller, manageable chunks. This approach ensures that the database system can process the updates more efficiently, maintaining both performance and data integrity.

#### Triaging the Question

The question is asking how to handle a large-scale database update operation in a way that minimizes its impact on the database's write performance and replication lag. The key aspects are understanding the current state of the database, identifying the potential bottlenecks (WAL and replication), and devising a strategy to mitigate these issues.

#### Assumptions
We are using PostgreSQL or a similar relational database system.
The backfill involves updating 500 million rows in one table.
Replication is set up with streaming replication, which is common for production databases.
There are no constraints on the database that would complicate this operation further.

#### First Principles Solution
Identify the Problem: A single UPDATE statement can overwhelm the WAL and cause significant write delays, leading to increased replication lag.
Segment the Update: Break down the 500 million rows into smaller batches for processing.
Use Transactions Wisely: Ensure each batch of updates is committed in a transaction to maintain data integrity.
Monitor Performance: Continuously monitor the database's performance metrics (WAL size, replication lag) during the operation.
Step-by-Step Solution

Determine Batch Size:

The optimal batch size depends on several factors, including system capacity and available resources.
A practical approach is to start with a small batch size, say 50K rows, and increase it based on observed performance metrics.

Partition the Data:

Use SQL WHERE clauses or additional indices to partition the data into smaller chunks.
For example, if you have a date column, partition by date ranges (e.g., every week).

Batch Processing:

Execute an UPDATE statement for each batch in a transaction.
Use a loop or a procedural block to automate this process.

Monitor and Adjust:

Continuously monitor the WAL size and replication lag.
Adjust the batch size if necessary, possibly decreasing it if the database is struggling under the load.

Optimize Indexes and Statistics:

Ensure that indexes are up-to-date to improve query performance.
Run ANALYZE commands on relevant tables to update statistics, which can help the planner optimize queries better.

#### Example
Code
SQL
-- Step 1: Determine batch size (example: 50K rows)
DO $$
DECLARE
    chunk_size INT := 50000;
    start_id INT := (SELECT MIN(id) FROM large_table);
BEGIN
    -- Loop through the data in batches
    WHILE start_id IS NOT NULL LOOP
        -- Execute an UPDATE statement for a batch of rows
        EXECUTE format('UPDATE large_table SET column1 = new_value WHERE id >= %s AND id < %s',
                       start_id, start_id + chunk_size);

        -- Commit the transaction to ensure data integrity
        COMMIT;

        -- Move to the next batch
        start_id := (SELECT MIN(id) FROM large_table WHERE id > start_id + chunk_size);
    END LOOP;
END $$;

#### Closing Summary
and Common Mistakes/Follow-Ups

By breaking down the update into smaller batches, we can manage the write load more effectively. This approach ensures that the database system can handle the updates without excessive resource consumption or performance degradation.

#### Common Mistakes
to Avoid:

Failing to monitor performance metrics during the operation.
Setting the batch size too large, leading to significant write delays and replication lag.
Not adjusting the batch size based on observed performance.

If asked for follow-ups:

How would you handle a situation where the database is already under heavy load?
What about concurrent transactions affecting the table being updated?

**Comparison:**

| Batch Size | Monitoring Metrics |
|------------|--------------------|
| Initial    | WAL: 1.5GB, Lag: 2 min |
| Adjusted   | WAL: 1.3GB, Lag: 1.5 min |

**Code:**

```sql
DO $$
DECLARE
    chunk_size INT := 50000;
    start_id INT := (SELECT MIN(id) FROM large_table);
BEGIN
    -- Loop through the data in batches
    WHILE start_id IS NOT NULL LOOP
        -- Execute an UPDATE statement for a batch of rows
        EXECUTE format('UPDATE large_table SET column1 = new_value WHERE id >= %s AND id < %s',
                       start_id, start_id + chunk_size);

        -- Commit the transaction to ensure data integrity
        COMMIT;

        -- Move to the next batch
        start_id := (SELECT MIN(id) FROM large_table WHERE id > start_id + chunk_size);
    END LOOP;
END $$;
```

**Example input / output:**

```text
Initial Setup:
- Database: PostgreSQL
- Batch size: 50K rows

Monitoring Metrics During Operation:
- WAL Size: 2GB
- Replication Lag: 3 minutes

Adjustments Made:
- Batch size increased to 100K rows after monitoring showed no issues.
```

**Data:**

```sql
-- Sample table schema for reference
CREATE TABLE large_table (
    id SERIAL PRIMARY KEY,
    column1 INT,
    -- other columns...
);

-- Example data insertion (for demonstration purposes)
INSERT INTO large_table (column1) SELECT generate_series(1, 50000000);
```

**Flow:**

1. Yes
2. No
3. Initial Setup
4. Partition Data
5. Set Batch Size (50K)
6. Monitor Performance Metrics
7. Is Performance Satisfactory?
8. Proceed with Batches
9. Adjust Batch Size and Retry
10. Execute UPDATE Statements inTransactions
11. Maintain Data Integrity byCommitting Each Batch

