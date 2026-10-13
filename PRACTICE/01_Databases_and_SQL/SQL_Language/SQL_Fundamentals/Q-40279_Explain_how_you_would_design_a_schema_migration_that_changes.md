---
type: "practice"
difficulty: "Expert"
---

# Explain how you would design a schema migration that changes a column's type on a 500M-row table with zero downtime, including the online-backfill and dual-write phases.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a schema migration to change a column's data type in a large table with millions of rows and high traffic, ensuring zero downtime is critical. This process involves three key phases: online-backfill, dual-write, and final migration. The goal is to modify the data while maintaining service availability throughout.

Firstly, let's understand what "zero downtime" means here. It implies that users can continue using the application without any disruptions during the migration. This requirement significantly complicates the process but ensures a smooth transition of data without affecting user experience or application performance.

To achieve this, we'll employ online-backfill and dual-write strategies:

Online-Backfill Phase: We prepare the new column by backfilling it with existing data from the old column. This step occurs during low traffic periods to minimize impact.
Dual-Write Phase: Simultaneously, both columns are active until all rows have been updated. New writes go into both columns initially, ensuring consistency and availability.
Final Migration Phase: Once dual-write is complete, we remove the old column or deprecate it.

Now, let's delve into each phase in detail:

Online-Backfill Phase

During this phase, we prepare the new column by populating it with data from the old one. To minimize downtime and performance impact:

Identify Low-Traffic Periods: Schedule backfills during off-peak hours or perform them incrementally over multiple days.
Use Transactions for Atomicity: Ensure that each update is atomic to maintain consistency.
Batch Processing: Use batch inserts instead of individual row updates to reduce database load.
Dual-Write Phase

In this phase, both the old and new columns are active. This ensures no loss of writes during the transition:

Simultaneous Writes: Any insert or update operation must write to both columns.
Validation Checks: Implement validation logic to ensure data integrity (e.g., ensuring the conversion is successful).
Error Handling: Robust error handling mechanisms should be in place to deal with potential issues.
Final Migration Phase

Once all rows have been updated and validated, we can finalize the migration:

Deprecate Old Column: Remove or mark the old column as deprecated.
Cleanup: Ensure there are no lingering references to the old column.
Performance Optimization: Rebalance indexes and optimize queries to improve overall performance.

#### Common Mistakes
Ignoring Off-Peak Scheduling: Failing to schedule backfills during low traffic periods can lead to increased server load and potential downtime.
Lack of Robust Error Handling: Without proper error handling, issues in dual-write or validation phases could cause data integrity problems.
Incomplete Final Migration: Not thoroughly deprecating the old column or failing to clean up references can leave residual risks.

**Comparison:**

|Phase| Description|
|---|---|
|Online-Backfill| Backfill new column with existing data from old one.|
|Dual-Write| Both old and new columns active; simultaneous writes.|
|Final Migration| Deprecate old column, cleanup, optimization.|

**Code:**

```sql
-- Example SQL for backfilling a new column
BEGIN;
INSERT INTO new_table (new_column)
SELECT CAST(old_column AS NEW_DATA_TYPE) FROM original_table WHERE id BETWEEN 1000 AND 2000;
COMMIT;

-- Dual-write example
INSERT INTO original_table (old_column, new_column) VALUES ('value', 'converted_value');
```

**Example input / output:**

```text
Input:
- Original table with 500M rows, old_column as INT.
- Requirement: Change data type to VARCHAR(255).

Output (Online-Backfill Phase):
1. Schedule backfills for off-peak hours.
2. Perform batch inserts for each backfill.

Output (Dual-Write Phase):
1. Incremental updates of both columns during high traffic.
2. Validation checks and error handling implemented.
```

**Data:**

```json
{
    "original_table": {
        "columns": ["id", "old_column"],
        "rows_count": 500000000,
        "traffic_peak_hours": ["9:00 AM - 5:00 PM"]
    }
}
```

**Flow:**

1. Yes
2. No
3. Online-Backfill
4. Prepare New Column
5. Is Traffic Low?
6. Perform Batch Inserts
7. Reschedule Backfill
8. Dual-Write
9. Simultaneous Writes
10. Final Migration
11. Deprecate Old Column,Cleanup

