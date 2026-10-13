---
type: "practice"
difficulty: "Deep"
---

# How would you handle materialized views defined over slowly-changing dimensions so historical aggregates remain correct as dimension attributes evolve?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Materialized View Refresh and Maintenance](<../../../../01_Databases_and_SQL/Database_Objects/Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Materialized views are precomputed and stored results of SQL queries, used to speed up complex query execution by caching their computed outputs. In scenarios with slowly-changing dimensions (SCDs), ensuring historical aggregates remain correct as dimension attributes evolve becomes a critical task. This involves understanding the different types of SCDs, how they impact aggregations, and designing a strategy to maintain accuracy.

#### Triage

The core issue here is maintaining accurate historical aggregates in materialized views when underlying slowly-changing dimensions are updated over time. We need to consider both the type of SCD being used (e.g., Type 2) and the specific aggregation logic involved.

#### Assumptions
Slowly-Changing Dimensions (SCDs): We focus on Type 2, where existing records are not deleted but new rows are inserted with updated attributes.
Historical Aggregates: These need to be recomputed whenever changes occur in the dimension values that affect the aggregates.

#### First Principles Solution

Identify Relevant Dimensions:

Determine which dimensions impact the aggregations (e.g., time, location).

Update Strategy for Materialized Views:

For Type 2 SCDs, when a change is detected in the dimension attributes, update the materialized view to reflect this change.
Ensure that old records are included in the aggregates until they are no longer relevant.

Version Control:

Maintain versioning of historical data. This ensures that previous states can be referenced and aggregated accurately.

Incremental Updates:

Instead of recomputing the entire materialized view, incrementally update only those parts affected by the change.

#### Cons
istency Checks:

Perform periodic consistency checks to ensure that historical aggregates are still valid.

#### Implementation
Steps

Detect Changes in SCDs:

Use triggers or scheduled jobs to monitor changes in dimension attributes.

Update Materialized View Logic:

SQL
-- Example update logic for a materialized view with Type 2 SCDs
CREATE MATERIALIZED VIEW sales_summary AS
SELECT
    customer_id,
    date_trunc('month', sale_date) as month,
    SUM(sale_amount) as total_sales,
    COUNT(*) as num_sales
FROM
    sales
GROUP BY
    customer_id, month;

-- Trigger to update the materialized view when a dimension changes
CREATE OR REPLACE TRIGGER update_sales_summary
AFTER UPDATE OF sale_date ON sales
FOR EACH ROW
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
END;

Maintain Historical Data:

Ensure that old records are retained in the materialized view until they are no longer needed.

Periodic Rebuilds:

Schedule periodic rebuilds of materialized views to ensure data consistency and accuracy.

#### Closing Summary

Maintaining accurate historical aggregates with materialized views involves a combination of identifying relevant dimensions, using appropriate SCD strategies, implementing incremental update logic, and performing regular checks for consistency. The key is to balance the need for speed and accuracy by carefully managing how and when updates are applied to the materialized view.

#### Common Mistakes
Ignoring Historical Data: Failing to retain old records can lead to incorrect aggregates.
Overcomplicating Triggers: Complex triggers may introduce performance issues or inconsistencies if not managed properly.
Neglecting Consistency Checks: Periodic checks are essential to ensure the integrity of historical data.

**Comparison:**

```
Customer ID	Month	Total Sales	Num Sales
1	2023-01	100	1
1	2023-02	150	1
1	2023-03	0	0
2	2023-01	200	1
```

**Code:**

```sql
-- Example SQL for creating a materialized view and a trigger

CREATE MATERIALIZED VIEW sales_summary AS
SELECT
    customer_id,
    date_trunc('month', sale_date) as month,
    SUM(sale_amount) as total_sales,
    COUNT(*) as num_sales
FROM
    sales
GROUP BY
    customer_id, month;

-- Trigger to update the materialized view when a dimension changes
CREATE OR REPLACE TRIGGER update_sales_summary
AFTER UPDATE OF sale_date ON sales
FOR EACH ROW
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
END;
```

**Example input / output:**

```text
-- Input
UPDATE sales SET sale_date = '2023-03-01' WHERE customer_id = 1;

-- Output (materialized view update)
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

**Data:**

```json
{
  "sales": [
    { "customer_id": 1, "sale_amount": 100, "sale_date": "2023-01-01" },
    { "customer_id": 1, "sale_amount": 150, "sale_date": "2023-02-01" },
    { "customer_id": 2, "sale_amount": 200, "sale_date": "2023-01-01" }
  ]
}
```

**Flow:**

1. Detect SCD Changes
2. Update Materialized ViewLogic
3. Maintain Historical Data
4. Periodic Rebuilds ofMaterialized Views
5. Ensure Consistency Checks
6. Historical Aggregates RemainCorrect

