---
type: "practice"
difficulty: "Deep"
---

# How would you write a query to find the effective Type-2 dimension row for each fact using a range containment join on valid_from/valid_to, and why can an open-ended NULL valid_to break the BETWEEN?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to understand how to identify the effective Type-2 dimension row for each fact using a range containment join based on valid_from and valid_to. The key challenge here is handling open-ended NULL values in the valid_to field. Let's break down the solution step by step.

Firstly, a Type-2 surrogate key dimension ensures that historical changes are tracked by creating new rows for each change. To find the effective row, we need to join the fact table with the dimension table using a range check based on the valid date ranges (valid_from and valid_to). However, when the valid_to is NULL, it indicates an open-ended period, which can complicate the query logic.

#### Triaging the Question

The question asks us to write a SQL query that identifies the effective Type-2 dimension row for each fact. The key points are:

Using a range containment join on valid_from and valid_to.
Handling open-ended NULL values in the valid_to field.

#### Assumptions
We have two tables: facts and dimensions.
facts table has columns [fact_id, fact_date].
dimensions table has columns [dim_id, valid_from, valid_to, dim_data].
Step-by-Step Solution
Identify the Active Dimension Row: For a given date in the facts table, we need to find the dimension row that is active on or before the fact date.
Handling NULL valid_to: If valid_to is NULL for a dimension row, it means the record is still valid and should be considered as active until now.

Here’s how you can construct the SQL query:

SQL
SELECT
    f.fact_id,
    d.dim_id,
    d.valid_from,
    d.valid_to,
    d.dim_data
FROM
    facts f
JOIN (
    SELECT
        dim_id,
        valid_from,
        CASE
            WHEN valid_to IS NULL THEN MAX(valid_to) OVER ()  -- Handle open-ended NULL values
            ELSE valid_to
        END AS valid_to
    FROM dimensions
    WHERE valid_from <= '2023-10-01'  -- Example fact date
) d ON f.fact_date BETWEEN d.valid_from AND COALESCE(d.valid_to, DATE '9999-12-31')
Explanation
Subquery: The subquery selects the relevant dimension rows that are active on or before a given fact date.
CASE Statement: This handles open-ended NULL values by setting valid_to to a future date (DATE '9999-12-31' in this example).
BETWEEN Clause: The outer join ensures that all facts are matched with the appropriate dimension row, even if the valid period is open-ended.

#### Closing Summary

This query effectively identifies the correct Type-2 dimension row for each fact by handling both finite and infinite date ranges. It demonstrates a deep understanding of SQL and range queries, as well as how to manage NULL values in practical scenarios.

#### Common Mistakes/Follow-Ups
Not Handling NULL valid_to Properly: Failing to handle open-ended periods can lead to incorrect results.
Incorrect Date Comparisons: Using the wrong date comparison logic can result in missed or incorrect matches.

#### Performance Considerations
For very large datasets, ensure that indexes are used on relevant columns (valid_from, valid_to) to optimize performance.

