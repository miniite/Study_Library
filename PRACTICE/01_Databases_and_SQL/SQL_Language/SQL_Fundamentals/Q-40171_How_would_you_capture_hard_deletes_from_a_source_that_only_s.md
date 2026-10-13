---
type: "practice"
difficulty: "Intermediate"
---

# How would you capture hard deletes from a source that only sends a daily full snapshot, using an EXCEPT/anti-join between yesterday's and today's keys, and mark them soft-deleted downstream?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of capturing hard deletes from a source that only sends daily full snapshots, we can use an EXCEPT/anti-join between yesterday’s and today’s datasets to identify records no longer present in today's snapshot. These missing records will be marked as soft-deleted in our downstream system. This approach ensures we accurately track deletions without requiring incremental updates from the source.

#### Triaging the Question

The core of this question is about identifying deleted records efficiently using SQL techniques. The key points are:

Daily Full Snapshots: The data source sends a complete set of records every day.
EXCEPT/ANTI-JOIN: We need to use these SQL operations to find records that exist in yesterday’s dataset but not today's, indicating deletions.
Soft-Deleting Downstream: Mark the identified deleted records appropriately in downstream storage.
Step-by-Step Solution from First Principles

Identify Yesterday's and Today’s Snapshots:

Assume yesterday_data is a table holding data from yesterday’s snapshot.
Assume today_data is a table holding today’s snapshot.

Use EXCEPT/ANTI-JOIN to Identify Deletions:

Perform an anti-join between yesterday_data and today_data. This will give us records in yesterday_data that do not exist in today_data.

Mark Records as Soft-Deleted Downstream:

Update the downstream storage to mark these identified deletions appropriately.
Detailed SQL Implementation
SQL
-- Step 1: Identify yesterday's and today’s snapshots
CREATE OR REPLACE TABLE yesterday_data AS
SELECT * FROM source_table WHERE snapshot_date = (CURRENT_DATE - INTERVAL '1 day');

CREATE OR REPLACE TABLE today_data AS
SELECT * FROM source_table WHERE snapshot_date = CURRENT_DATE;

-- Step 2: Use EXCEPT/ANTI-JOIN to identify deletions
WITH deleted_records AS (
    SELECT yd.*
    FROM yesterday_data yd
    LEFT JOIN today_data td ON yd.key_field = td.key_field
    WHERE td.key_field IS NULL
)
SELECT * INTO soft_deleted_table FROM deleted_records;

-- Step 3: Mark records as soft-deleted downstream
UPDATE target_table SET is_soft_deleted = TRUE, deletion_date = CURRENT_DATE
WHERE key_field IN (SELECT key_field FROM soft_deleted_table);

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach effectively captures hard deletes by leveraging SQL’s EXCEPT/anti-join capabilities. It ensures accurate tracking of deletions without requiring continuous synchronization with the source system, making it efficient for large-scale data management.

#### Common Mistakes

Incorrect Join Conditions: Ensure that the join condition (key_field) is correctly defined to match records across snapshots.
Data Schema Mismatch: Make sure yesterday_data and today_data have consistent schemas for proper anti-join operations.

#### Performance Considerations
For very large datasets, consider performance optimizations like partitioning or indexing on frequently accessed columns.

#### Follow-Ups

How would you handle scenarios where the source system sends partial snapshots?
What if we need to track not only deletes but also updates and inserts?

**Comparison:**

| ID | Name         | Snapshot Date |
|----|--------------|---------------|
| 1  | John Doe     | 2023-10-01    |
| 2  | Jane Smith   | 2023-10-01    |
| 3  | Bob Johnson  | 2023-10-01    |
| ID | Name         | Snapshot Date |
|----|--------------|---------------|
| 1  | John Doe     | 2023-10-02    |
| 3  | Bob Johnson  | 2023-10-02    |
| 4  | Alice Brown  | 2023-10-02    |
| ID | Name         | Snapshot Date   |
|----|--------------|-----------------|
| 2  | Jane Smith   | 2023-10-01      |

**Code:**

```sql
-- SQL Code for identifying deleted records using EXCEPT/ANTI-JOIN
CREATE OR REPLACE TABLE yesterday_data AS
SELECT * FROM source_table WHERE snapshot_date = (CURRENT_DATE - INTERVAL '1 day');

CREATE OR REPLACE TABLE today_data AS
SELECT * FROM source_table WHERE snapshot_date = CURRENT_DATE;

WITH deleted_records AS (
    SELECT yd.*
    FROM yesterday_data yd
    LEFT JOIN today_data td ON yd.key_field = td.key_field
    WHERE td.key_field IS NULL
)
SELECT * INTO soft_deleted_table FROM deleted_records;
```

**Example input / output:**

```text
-- Input from yesterday_data and today_data
yesterday_data = {
  (1, 'John Doe', '2023-10-01'),
  (2, 'Jane Smith', '2023-10-01'),
  (3, 'Bob Johnson', '2023-10-01')
}

today_data = {
  (1, 'John Doe', '2023-10-02'),
  (3, 'Bob Johnson', '2023-10-02'),
  (4, 'Alice Brown', '2023-10-02')
}

-- Output from soft_deleted_table
soft_deleted_table = {
  (2, 'Jane Smith', '2023-10-01')
}
```

**Data:**

```sql
-- Sample data schema for source_table
CREATE TABLE source_table (
  id INT,
  name VARCHAR(100),
  snapshot_date DATE
);
```

**Flow:**

1. Yes
2. No
3. Identify yesterday's andtoday’s snapshots
4. Are schemas consistent?
5. Perform EXCEPT/ANTI-JOIN
6. Adjust schema or handlemismatch
7. Identify deleted records indownstream storage
8. Mark as soft-deleted andupdate target table

