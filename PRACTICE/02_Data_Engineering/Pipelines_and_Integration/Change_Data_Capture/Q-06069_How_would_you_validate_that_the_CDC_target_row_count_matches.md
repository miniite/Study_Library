---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that the CDC target row count matches the source after a load?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To validate whether the Change Data Capture (CDC) target row count matches the source after a data load, we need to ensure data integrity and consistency between our source and target databases. The key idea is to compare the number of rows in both environments post-load.

First, let's restate what the question is probing: it aims to understand how you would implement and verify CDC processes, ensuring that data transformations and loads are accurate and complete. We'll need to consider assumptions such as the presence of unique keys, transactional integrity, and potential delays in replication.

Step-by-Step Solution
Identify Unique Keys: Assuming the source and target tables have a unique key (e.g., id), we can use this to match rows between them.
Post-Load Check: After loading new or updated data into the target, we will perform a row count comparison.
Incremental Validation: Use incremental validation techniques to ensure only changes are being captured and applied.
First Principles

Source Row Count:

Query the source database to get the current row count before the load.
SQL
SELECT COUNT(*) AS source_row_count FROM source_table;

Target Pre-Load Row Count:

Query the target database to get its current row count before loading any new data.
SQL
SELECT COUNT(*) AS target_pre_load_row_count FROM target_table;

Post-Load Target Row Count:

After the load, query the target again to get the updated row count.
SQL
SELECT COUNT(*) AS target_post_load_row_count FROM target_table;

Validation Logic:

Compare the post-load target row count with the sum of pre-load target rows and source rows loaded during this operation.
SQL
SELECT (target_pre_load_row_count + new_rows_loaded) = target_post_load_row_count AS validation_passed FROM (
  -- SQL queries to fetch row counts mentioned above
);

#### Closing Summary

In summary, we use a combination of pre- and post-load checks to ensure that the CDC process has accurately captured and applied all changes. This method not only validates the integrity but also helps in identifying any discrepancies early.

#### Common Mistakes/Follow-Ups
Overlooking Unique Keys: If no unique key exists, using other columns for validation might be less reliable.
Failure to Handle Duplicates: Ensure that duplicates are properly managed and do not affect row counts.
Incomplete Loading: Verify that all changes have been applied correctly without missing any rows.

**Comparison:**

```
Source Table	Target Pre-Load	Target Post-Load
100	95	120
```

**Code:**

```sql
-- Example SQL queries
SELECT COUNT(*) AS source_row_count FROM source_table;

SELECT COUNT(*) AS target_pre_load_row_count FROM target_table;

INSERT INTO target_table (columns) VALUES (...);

SELECT COUNT(*) AS target_post_load_row_count FROM target_table;
```

**Example input / output:**

```text
Before load:
Source row count: 100

Target pre-load row count: 95

Post-load (assuming 25 new rows loaded):
Target post-load row count: 120

Validation:
(95 + 25) = 120 -> True, validation passed.
```

**Data:**

```text
- `source_table`: Schema with unique key.
- `target_table`: Schema with potential updates.
```

**Flow:**

1. Yes
2. No
3. Yes
4. True
5. False
6. Identify Unique Keys
7. Is Unique Key Present?
8. Perform Pre-load Target RowCount
9. Are All Changes Loaded?
10. Handle Incomplete Loading
11. Post-Load Target Row Count
12. Compare Counts
13. Validation Passed?
14. Continue Monitoring
15. Fix Issues, Re-run Validation
16. K

