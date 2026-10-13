---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to forward-fill a null status using the last non-null value with LAG IGNORE NULLS (or an equivalent).

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to use window functions in SQL to forward-fill any null values in the status column of a table using the last non-null value. This is particularly useful when dealing with time-series data where missing values need to be imputed based on previous non-missing values.

#### Triaging the Question

The question asks for writing an SQL query that uses window functions to handle null values effectively. Specifically, we are asked to use LAG and ignore nulls in the process of finding the last non-null value. This means if there is a null at position i, it should be replaced by the value from the previous non-null position.

#### Assumptions
The table has a column named status which might contain null values.
There could be multiple rows with different timestamps or identifiers, and we need to apply this logic across all such rows.
Step-by-Step Solution Using First Principles
Identify the Last Non-null Value: We will use the LAG function, which allows us to access data from previous rows within a window of rows.
Ignore Nulls with Conditionals: Since we want to ignore null values when looking for the last non-null value, we need to ensure that only valid (non-null) statuses are considered.

Here’s how you can achieve this:

SQL
SELECT
    id,
    timestamp,
    status,
    COALESCE(
        LAG(status IGNORE NULLS) OVER (
            PARTITION BY <identifier_column>
            ORDER BY timestamp
        ),
        status  -- Fallback to the current value if no non-null value is found in previous rows.
    ) AS filled_status
FROM your_table;

In this query:

LAG(status IGNORE NULLS) accesses the last non-null value of the status column from previous rows, considering only non-null values.
The PARTITION BY <identifier_column> clause ensures that we handle each group or individual row separately (you can replace <identifier_column> with an actual identifier like a user ID if your data is grouped).
ORDER BY timestamp orders the rows within each partition by time, ensuring that we get the most recent non-null value.
The COALESCE function ensures that if there are no non-null values to the left of the current row, the original value in the status column is retained.

#### Closing Summary
and Common Mistakes

This query effectively uses window functions to handle nulls by leveraging the LAG function along with ignoring nulls. The key points are:

Using LAG(status IGNORE NULLS) ensures that only non-null values are considered.
Ensuring proper partitioning helps in handling different groups of data separately.

#### Common mistakes
include not correctly specifying the PARTITION BY clause, misusing the order by clause, or failing to use conditional logic effectively with COALESCE.

