---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a query to reconcile a source sum (e.g. total revenue) against the loaded warehouse sum to prove the load was complete?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconcile a source sum against the loaded warehouse sum, we need to ensure that the data imported into our warehouse accurately reflects the original dataset's totals. This task involves writing an SQL query that compares the total revenue from the source system with the total revenue in the warehouse after loading. The key idea is to validate whether the load was successful and complete.

The question is probing your understanding of SQL, specifically how to join and aggregate data across tables, and how to handle potential discrepancies. We will assume the following:

source_table contains the original data with a column for revenue.
warehouse_table has been loaded from source_table, but we need to verify if the sums match.

Step-by-step, let's break down the solution:

Join and Aggregate: Join the source table with the warehouse table on relevant keys (if any) and aggregate the revenue columns.
Calculate Totals: Compute the sum of revenues from both tables.
Compare Totals: Use a CASE statement or a simple comparison to determine if the totals match.

Here's how you can implement this in SQL:

SQL
WITH source_totals AS (
    SELECT SUM(revenue) as source_total_revenue
    FROM source_table
),
warehouse_totals AS (
    SELECT SUM(revenue) as warehouse_total_revenue
    FROM warehouse_table
)
SELECT
    source_total_revenue,
    warehouse_total_revenue,
    CASE
        WHEN source_total_revenue = warehouse_total_revenue THEN 'Reconciliation Successful'
        ELSE 'Reconciliation Failed: Discrepancy Detected'
    END AS reconciliation_status;

This query first calculates the total revenue from both the source_table and the warehouse_table. It then compares these totals, providing a straightforward result indicating whether the load was successful.

**Comparison:**

| source_total_revenue | warehouse_total_revenue | reconciliation_status                      |
|---------------------|------------------------|---------------------------------------------|
| 1000.00             | 1000.00                | Reconciliation Successful                  |

**Code:**

```sql
WITH source_totals AS (
    SELECT SUM(revenue) as source_total_revenue
    FROM source_table
),
warehouse_totals AS (
    SELECT SUM(revenue) as warehouse_total_revenue
    FROM warehouse_table
)
SELECT
    source_total_revenue,
    warehouse_total_revenue,
    CASE
        WHEN source_total_revenue = warehouse_total_revenue THEN 'Reconciliation Successful'
        ELSE 'Reconciliation Failed: Discrepancy Detected'
    END AS reconciliation_status;
```

**Example input / output:**

```

Input:

SQL
Copy
WITH source_totals AS (
    SELECT SUM(revenue) as source_total_revenue
    FROM source_table
),
warehouse_totals AS (
    SELECT SUM(revenue) as warehouse_total_revenue
    FROM warehouse_table
)
SELECT
    source_total_revenue,
    warehouse_total_revenue,
    CASE
        WHEN source_total_revenue = warehouse_total_revenue THEN 'Reconciliation Successful'
        ELSE 'Reconciliation Failed: Discrepancy Detected'
    END AS reconciliation_status;

Output:

SQL
Copy
+------------------+----------------------+---------------------------------------------+
| source_total_revenue | warehouse_total_revenue | reconciliation_status                      |
+------------------+----------------------+---------------------------------------------+
| 1000.00          | 1000.00              | Reconciliation Successful                  |
+------------------+----------------------+---------------------------------------------+
```

**Data:**

```sql
-- Example schema for illustration:
CREATE TABLE source_table (
    id INT,
    revenue DECIMAL(10,2)
);

CREATE TABLE warehouse_table (
    id INT,
    revenue DECIMAL(10,2)
);
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start
5. Is Source and Warehouse TableLoaded?
6. Calculate Total Revenue fromSource
7. Calculate Total Revenue fromWarehouse
8. Are Totals Equal?
9. Reconciliation Successful
10. Reconciliation Failed:Discrepancy Detected
11. End

