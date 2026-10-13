---
type: "practice"
difficulty: "Deep"
---

# How do you use a windowed subquery (ROW_NUMBER in a CTE) to de-duplicate before a join, and why is it safer than DISTINCT?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the problem of using a windowed subquery with ROW_NUMBER in a Common Table Expression (CTE) to deduplicate data before joining, we first need to understand why this approach is safer and more performant than using DISTINCT. The key idea here is leveraging window functions for precise control over data without losing the benefits of efficient joins.

TL;DR

Using a windowed subquery with ROW_NUMBER in a CTE allows us to deduplicate rows before performing a join operation. This approach ensures that only unique records are involved in the join, making the query safer and potentially more performant compared to using DISTINCT.

#### Triaging the Question

The core of this question is about:

Understanding Deduplication: How do we remove duplicate entries?
Windowed Subqueries and CTEs: What is a windowed subquery, and how does it work in a CTE?
Joins: How does deduplication affect join operations?

The interviewer is probing your understanding of:

Window functions and their application
The difference between DISTINCT and more targeted deduplication methods

#### Performance considerations
in SQL queries

#### First Principles Solution

Let's break down the solution step-by-step:

Identify Duplicates: Use a windowed subquery to assign a unique row number to each set of duplicate rows.
Filter Rows: Deduplicate by filtering out all but one instance of each group of duplicates.
Join Operation: Perform the join using the deduplicated data.

Here’s how you would implement this in SQL:

SQL
WITH DedupedData AS (
    SELECT
        column1,
        column2,
        ROW_NUMBER() OVER (PARTITION BY column1, column2 ORDER BY some_column) as rn
    FROM
        source_table
)
SELECT
    a.column1,
    a.column2,
    b.column3
FROM
    DedupedData a
JOIN
    another_table b ON a.column1 = b.column1 AND a.rn = 1;
Detailed Explanation
CTE DedupedData: This CTE uses the ROW_NUMBER() window function to assign a unique row number (rn) to each set of duplicate rows, partitioned by specific columns. The ORDER BY clause ensures that the row numbers are assigned in a deterministic manner.
Main Query: In the final query, we join the deduplicated data with another table only where rn = 1, ensuring that we include exactly one instance of each group of duplicates.
Why This Approach is Safer and More Efficient
Safety

Using DISTINCT in a large table can lead to incorrect results if not applied correctly. For example, if the join condition involves multiple columns, using DISTINCT might still produce duplicate rows because it operates on individual columns rather than groups of columns.

Performance
Windowed Subqueries: These are optimized by modern database engines and can be more efficient than DISTINCT, especially when dealing with large datasets.
Join Operations: By reducing the number of rows before joining, we improve join performance, making overall query execution faster.

#### Common Mistakes
and Follow-Ups
Incorrect Partitioning: Forgetting to partition correctly on all necessary columns can lead to incorrect deduplication.
Overusing DISTINCT: Relying too heavily on DISTINCT can degrade performance significantly.
Ignoring Join Conditions: Not ensuring that the join condition is correctly applied after deduplication.

#### Example

Let’s consider an example with a sales and customers table:

SQL
WITH CustomerDedup AS (
    SELECT
        customer_id,
        first_name, last_name,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY purchase_date DESC) as rn
    FROM
        sales
)
SELECT
    s.customer_id,
    c.first_name, c.last_name,
    s.sale_amount
FROM
    CustomerDedup s
JOIN
    customers c ON s.customer_id = c.customer_id AND s.rn = 1;

#### Closing Summary
and Follow-Ups

In summary, using a windowed subquery with ROW_NUMBER in a CTE is a safer and more efficient method for deduplicating data before joining. It provides precise control over which rows are included, reducing the risk of incorrect results and improving query performance.

Self-Check

This solution covers all aspects of the problem and addresses common mistakes effectively.

**Comparison:**

```
table CustomersDedup {
    rowNumber | customer_id | first_name | last_name | rn
    1         | 1           | John       | Doe       | 1
    2         | 1           | Jane       | Smith     | 2
    3         | 2           | Alice      | Johnson   | 1
}
```

**Code:**

```sql
WITH DedupedData AS (
    SELECT
        column1,
        column2,
        ROW_NUMBER() OVER (PARTITION BY column1, column2 ORDER BY some_column) as rn
    FROM
        source_table
)
SELECT
    a.column1,
    a.column2,
    b.column3
FROM
    DedupedData a
JOIN
    another_table b ON a.column1 = b.column1 AND a.rn = 1;
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Source Table (DedupedData) | Another Table |
| column1 | column2 | rn | column1 | column2 | column3 |
| 1      | A      | 1  | 1       | A      | 99     |
| 1      | A      | 2  |         |        |        |
| 2      | B      | 1  | 2       | B      | 88     |

**Data:**

```json
{
    "source_table": {
        "columns": ["column1", "column2", "some_column"],
        "rows": [
            {"column1": 1, "column2": "A", "some_column": 10},
            {"column1": 1, "column2": "A", "some_column": 5},
            {"column1": 2, "column2": "B", "some_column": 3}
        ]
    },
    "another_table": {
        "columns": ["column1", "column3"],
        "rows": [
            {"column1": 1, "column3": 99},
            {"column1": 2, "column3": 88}
        ]
    }
}
```

**Flow:**

1. Rendering diagram…

