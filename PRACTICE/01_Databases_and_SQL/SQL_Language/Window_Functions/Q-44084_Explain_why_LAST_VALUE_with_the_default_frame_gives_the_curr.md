---
type: "practice"
difficulty: "Deep"
---

# Explain why LAST_VALUE with the default frame gives the current row and walk through the exact frame you'd set to get the true partition last value.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why LAST_VALUE with a default frame in SQL returns the current row, we need to delve into how window functions operate and their underlying mechanics. Essentially, LAST_VALUE is designed to return the value of the last row within a specified window. However, when no explicit frame specification is provided, the default behavior often considers only the current row, which can lead to confusion.

#### Triaging the Question

The question asks us to explain why LAST_VALUE with the default frame returns the current row and then walk through how we would set an appropriate frame to get the true partition last value. This involves understanding the mechanics of window functions, specifically the role of frames in defining the scope over which operations are performed.

#### First-Principles Solution
Understanding Window Functions: A window function operates on a set of rows that is related to, but not necessarily identical to, the current row being processed.
Default Frame Behavior: In many SQL implementations (like PostgreSQL and SQL Server), when no explicit frame specification is provided with LAST_VALUE, it defaults to considering only the current row in the partition.
Specifying the Frame: To get the true last value within a partition, we need to specify an appropriate frame that includes all rows up to the current row.
Detailed Explanation

Window Function Basics:

Window functions like LAST_VALUE are defined over a window of rows, which can be specified using a PARTITION BY clause and optional ROWS BETWEEN or RANGE BETWEEN clauses.

Default Frame with LAST_VALUE:

By default, if no frame is specified in the LAST_VALUE function, it often considers only the current row within the partition. This behavior can be demonstrated with a simple query.

#### Example
Query
SQL
SELECT
    id,
    value,
    LAST_VALUE(value) OVER (PARTITION BY some_column)
FROM
    your_table;

In this example, if no explicit frame is provided for LAST_VALUE, it will return the current row's value within each partition. To illustrate:

For a table with rows (id, value) like (1, 10), (2, 20), (3, 30), and a partition by column some_column where all values have the same partition key, LAST_VALUE(value) will return 30 for each row because it considers only the current row.
Setting the Frame to Get True Last Value

To get the true last value within a partition, we need to specify an appropriate frame that includes all rows up to and including the current row. This can be done using the ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW clause.

#### Example
Query with Explicit Frame
SQL
SELECT
    id,
    value,
    LAST_VALUE(value) OVER (
        PARTITION BY some_column
        ORDER BY order_column
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS last_value
FROM
    your_table;

In this query:

PARTITION BY some_column defines the partition.
ORDER BY order_column specifies the order within each partition.
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ensures that all rows from the start of the partition up to and including the current row are considered.

#### Closing Summary

By default, LAST_VALUE without an explicit frame specification in SQL often returns only the current row's value. To get the true last value within a partition, we need to specify an appropriate frame using the ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW clause. This ensures that all rows up to and including the current row are included in the computation.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that LAST_VALUE always considers multiple rows when no explicit frame is provided.

#### Follow-Up
Discussing other window functions like LEAD, LAG, or ROW_NUMBER and their default behaviors, as well as how to set them appropriately.

**Comparison:**

```
tableDiagram
    your_table -->|PARTITION BY some_column, ORDER BY order_column| LAST_VALUE_window_function
    LAST_VALUE_window_function -->|ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW| last_value
```

**Code:**

```sql
-- Default behavior (current row only)
SELECT
    id,
    value,
    LAST_VALUE(value) OVER (PARTITION BY some_column)
FROM
    your_table;

-- Explicit frame for true last value
SELECT
    id,
    value,
    LAST_VALUE(value) OVER (
        PARTITION BY some_column
        ORDER BY order_column
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS last_value
FROM
    your_table;
```

**Example input / output:**

```text
Input:
Table: your_table

+----+-------+------------+----------------+
| id | value | some_column | order_column   |
+----+-------+------------+----------------+
| 1  | 10    | A          | 1              |
| 2  | 20    | A          | 2              |
| 3  | 30    | A          | 3              |
+----+-------+------------+----------------+

Default LAST_VALUE query:
SELECT
    id,
    value,
    LAST_VALUE(value) OVER (PARTITION BY some_column)
FROM
    your_table;

Output:
+----+-------+----------------------------------+
| id | value | LAST_VALUE(value) OVER (PARTITION BY some_column) |
+----+-------+----------------------------------+
| 1  | 30    | 30                               |
| 2  | 30    | 30                               |
| 3  | 30    | 30                               |
+----+-------+----------------------------------+

Explicit LAST_VALUE query:
SELECT
    id,
    value,
    LAST_VALUE(value) OVER (
        PARTITION BY some_column
        ORDER BY order_column
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS last_value
FROM
    your_table;

Output:
+----+-------+----------------------------------+
| id | value | last_value                       |
+----+-------+----------------------------------+
| 1  | 10    | 30                               |
| 2  | 20    | 30                               |
| 3  | 30    | 30                               |
+----+-------+----------------------------------+
```

**Data:**

```json
{
  "your_table": [
    {"id": 1, "value": 10, "some_column": "A", "order_column": 1},
    {"id": 2, "value": 20, "some_column": "A", "order_column": 2},
    {"id": 3, "value": 30, "some_column": "A", "order_column": 3}
  ]
}
```

**Flow:**

1. Define Window Function
2. LAST_VALUE(value)
3. PARTITION BY some_column
4. ORDER BY order_column
5. ROWS BETWEENUNBOUNDED PRECEDINGAND CURRENT ROW
6. Compute Last Value for EachRow in Partition

