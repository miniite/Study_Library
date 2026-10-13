---
type: "practice"
difficulty: "Intermediate"
---

# Explain how Snowflake's QUALIFY clause is compiled and why it can avoid an extra subquery scan compared to filtering a window result in an outer WHERE.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain Snowflake's QUALIFY clause in SQL, let’s start by summarizing what it does and how it differs from using a WHERE clause with window functions. The QUALIFY clause is used to filter rows within each partition of the result set after applying the window function, much like a HAVING clause but for row-level filtering.

#### Triage
What the Question Is Asking

The question probes your understanding of how Snowflake optimizes SQL queries using its QUALIFY clause. Specifically, it wants you to explain why QUALIFY can avoid an extra subquery scan compared to using a separate WHERE clause after applying window functions.

Triaging the Problem
Concept: Window Functions and their use in querying large datasets.
Key Idea: QUALIFY vs. WHERE: Understanding the execution plan differences between these two clauses.

#### Assumptions
The query uses a window function to calculate some metric (e.g., rank, dense_rank, etc.) over a partition of data.
Both the WHERE and QUALIFY clauses are used for row-level filtering after applying the window function.
The goal is to minimize unnecessary scans or operations in the execution plan.
Step-by-Step Solution
1. Understanding Window Functions

Window functions operate on a set of rows related to the current row, allowing you to perform calculations like ranking, summing, and averaging over these sets. For example:

SQL
SELECT
    id,
    value,
    RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
FROM
    data_table;

This query ranks value within each partition of group.

2. Using a WHERE Clause with Window Functions

When you use a WHERE clause after the window function, Snowflake needs to first apply the window function to all rows and then filter based on the condition in the WHERE clause.

SQL
SELECT
    id,
    value,
    rank
FROM (
    SELECT
        id,
        value,
        RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
    FROM
        data_table
) WHERE rank = 1;

In this example, all rows are processed by the window function to calculate rank, and then only those with a rank of 1 are selected.

3. Using the QUALIFY Clause

The QUALIFY clause is more efficient because it can filter rows within each partition before fully materializing the entire result set.

SQL
SELECT
    id,
    value,
    RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
FROM
    data_table QUALIFY rank = 1;

Here, Snowflake only calculates rank for rows that meet the condition in the QUALIFY clause. This can significantly reduce the number of rows processed and thus improve performance.

4. Execution Plan Differences
WHERE Clause: All rows are read, window functions are applied, and then filtered.
QUALIFY Clause: Rows are read and filtered within each partition before fully processing all rows.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key advantage of QUALIFY is that it can avoid the full scan by filtering early in the execution plan. This makes it particularly useful for large datasets where reducing the number of processed rows can significantly improve performance. A common mistake is to overlook the QUALIFY clause and default to using a separate WHERE clause, which can lead to unnecessary processing.

#### Common Mistakes
Using WHERE instead of QUALIFY: This leads to an extra subquery scan.
Overlooking the PARTITION BY in QUALIFY: Properly partitioning data is crucial for effective window function usage and performance optimization.

#### Follow-Ups
Discuss how other database systems handle similar scenarios.
Explain specific use cases where WHERE might be preferred over QUALIFY.

**Comparison:**

```
WHERE{All rows processed,then filtered}
QUALIFY{Filter early inexecution plan}
Input
WHERE vs QUALIFY
Full Scan
Efficient Execution
```

**Code:**

```sql
-- Example with WHERE
SELECT
    id,
    value,
    rank
FROM (
    SELECT
        id,
        value,
        RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
    FROM
        data_table
) WHERE rank = 1;

-- Example with QUALIFY
SELECT
    id,
    value,
    RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
FROM
    data_table QUALIFY rank = 1;
```

**Example input / output:**

```text
-- Input with WHERE
SELECT
    id,
    value,
    rank
FROM (
    SELECT
        id,
        value,
        RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
    FROM
        data_table
) WHERE rank = 1;

-- Output with WHERE
+----+-------+-----+
| ID | VALUE | RANK|
+----+-------+-----+
|  3 |     8 |   1 |
+----+-------+-----+

-- Input with QUALIFY
SELECT
    id,
    value,
    RANK() OVER (PARTITION BY group ORDER BY value DESC) as rank
FROM
    data_table QUALIFY rank = 1;

-- Output with QUALIFY
+----+-------+-----+
| ID | VALUE | RANK|
+----+-------+-----+
|  3 |     8 |   1 |
+----+-------+-----+
```

**Data:**

```json
{
  "data_table": [
    {"id": 1, "value": 5},
    {"id": 2, "value": 3},
    {"id": 3, "value": 8},
    {"id": 4, "value": 6},
    {"id": 5, "value": 7}
  ]
}
```

**Flow:**

1. Input
2. Window Function Applied
3. WHERE Clause
4. Full Scan of Rows
5. Filter Based on Condition
6. QUALIFY Clause
7. Partition-wise Filtering
8. Efficient Row Processing

