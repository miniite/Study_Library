---
type: "practice"
difficulty: "Deep"
---

# When a window-heavy transformation is materialized as an incremental table, how do you decide the lookback window (e.g. last 3 days) so late-arriving data still gets correct running aggregates?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of ensuring that late-arriving data in a materialized incremental table with a lookback window (e.g., last 3 days) correctly updates running aggregates, we need to carefully design our approach. This involves understanding how window functions operate and considering the trade-offs between different strategies.

#### Triage

The core question is about designing an incremental table that maintains correct running aggregates over a certain lookback period even when new data arrives late. The key elements are:

Window Definition: How do we define the sliding or fixed window?
Materialization Strategy: What approach should be taken to update the aggregate values efficiently and accurately?
Late-Arriving Data Handling: How can we ensure that late-arrived data does not cause incorrect aggregates?

#### First-Principles Solution

To achieve this, we need a mechanism that can handle late-arrived data without disrupting the ongoing calculation of running aggregates. Here’s a step-by-step approach:

Define the Window: Decide on whether to use a fixed window (e.g., last 3 days) or a sliding window.
Aggregate Calculation: Use SQL window functions like SUM() OVER (), AVG() OVER (), etc., in your query.
Materialization Strategy: Materialize the results incrementally and ensure that each row includes metadata about its position within the window.
Handling Late Data: Implement a strategy to update existing aggregates when late data arrives.
Detailed Solution
1. Define the Window

For a fixed window like "last 3 days," we can use:

SQL
SELECT
    id,
    value,
    SUM(value) OVER (
        PARTITION BY id
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_sum
FROM
    your_table
WHERE
    timestamp >= current_date - 3;

This query calculates the running sum for each id over a fixed window of the last three days.

2. Aggregate Calculation

In SQL, you can use window functions to calculate various aggregates such as sums, averages, etc. For example:

SQL
SELECT
    id,
    value,
    SUM(value) OVER (
        PARTITION BY id
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_sum,
    COUNT(*) OVER (
        PARTITION BY id
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS count
FROM
    your_table;
3. Materialization Strategy

Materialize the results by periodically writing these aggregates to a materialized view or an incremental table:

SQL
CREATE TABLE aggregated_data (
    id INT,
    value FLOAT,
    running_sum FLOAT,
    count INT
) PARTITION BY RANGE (id);

You can then insert rows into this table using batch processing or streaming data ingestion.

4. Handling Late Data

To handle late-arriving data, you can use a technique like "late arrival handling" where you update the existing aggregates when new data arrives:

SQL
UPDATE aggregated_data
SET running_sum = (SELECT SUM(value) OVER (
    PARTITION BY id
    ORDER BY timestamp
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
), count = (SELECT COUNT(*) OVER (
    PARTITION BY id
    ORDER BY timestamp
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
))
WHERE id IN (SELECT id FROM your_table WHERE timestamp > current_date - 3);

Alternatively, you can use a more sophisticated approach like maintaining a rolling window in the materialized view.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The key is to ensure that late-arriving data does not disrupt ongoing calculations by using appropriate window definitions and handling mechanisms.

#### Common Mistakes
Not considering the impact of late-arrived data on running aggregates.
Failing to define clear window boundaries and update strategies.
Overlooking the need for periodic materialization to ensure correctness.

#### Follow-Ups
How do you handle very large datasets?
What about distributed systems where data is processed in multiple nodes?
Are there alternative techniques or tools that can help manage late-arrived data more efficiently?

**Comparison:**

    | id | value | running_sum | count |
    |---|----|---------|------|
    | 123 | 10.5 | 46.0    | 3     |

**Code:**

```sql
-- Example SQL for running sum with fixed window
SELECT
    id,
    value,
    SUM(value) OVER (
        PARTITION BY id
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_sum
FROM
    your_table
WHERE
    timestamp >= current_date - 3;

-- Example SQL for updating materialized view with late data
UPDATE aggregated_data
SET running_sum = (SELECT SUM(value) OVER (
    PARTITION BY id
    ORDER BY timestamp
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
), count = (SELECT COUNT(*) OVER (
    PARTITION BY id
    ORDER BY timestamp
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
))
WHERE id IN (SELECT id FROM your_table WHERE timestamp > current_date - 3);
```

**Example input / output:**

```text
Input:
- Table: your_table
- New record: {"id": 123, "value": 15.0, "timestamp": "2023-10-04T12:00:00Z"}

Output:
- Updated running sum for id 123 in aggregated_data table.
```

**Data:**

```json
{
  "id": 123,
  "value": 10.5,
  "timestamp": "2023-10-01T12:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is new data late?
5. Maintain rolling window andupdate aggregates
6. Calculate running aggregatesdirectly
7. Update materialized view
8. Insert new aggregate intotable
9. Maintain consistentaggregates

