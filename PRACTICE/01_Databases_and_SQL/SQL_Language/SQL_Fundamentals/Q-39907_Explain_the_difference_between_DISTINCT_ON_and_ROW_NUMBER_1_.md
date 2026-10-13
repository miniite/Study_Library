---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between DISTINCT ON and ROW_NUMBER() = 1 for picking the latest row per group, and what ORDER BY guarantees you need for determinism.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between DISTINCT ON and ROW_NUMBER() in SQL, let's first clarify their core purposes and how they achieve picking the "latest" row per group. Both are commonly used to select a single record per group based on some criteria, but they work differently under the hood.

#### Triage
Question Core: The question is about understanding when and why you might use DISTINCT ON versus ROW_NUMBER() for selecting one record per group in SQL.
Interviewer's Probing: The interviewer is testing your knowledge of these specific SQL clauses, their syntax differences, and the conditions necessary for them to function correctly. They are also checking if you can explain the underlying logic that ensures determinism.

#### First Principles Solution

DISTINCT ON is a PostgreSQL-specific feature that selects unique rows from a result set based on the specified columns, ensuring only one row per group. On the other hand, ROW_NUMBER() is a more general-purpose window function used to assign a unique sequential integer to records within a partition of a result set.

Key Differences

Syntax and SQL Compatibility:

DISTINCT ON: Works in PostgreSQL.
ROW_NUMBER(): Available in most modern SQL databases like MySQL, SQLite, and Oracle with slight variations (e.g., RANK()).

Usage Scenarios:

DISTINCT ON: Simplifies the query by directly selecting unique rows per group without needing to explicitly number them.
ROW_NUMBER(): More flexible as it can be used in conjunction with other window functions and operations.

Determinism:

Both require an ORDER BY clause to ensure a deterministic result, but the nature of the ordering differs:
DISTINCT ON: The first row per group is selected based on the specified columns.
ROW_NUMBER(): The rows are numbered in the order defined by the ORDER BY, and then you select the top number.
Detailed Explanation

Let's consider a practical example where we want to find the latest order for each customer:

SQL
-- Example table: orders
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date TIMESTAMP NOT NULL
);

INSERT INTO orders (customer_id, order_date) VALUES
(1, '2023-01-01 09:00'),
(1, '2023-01-02 10:00'),
(2, '2023-02-01 11:00');
Using DISTINCT ON
SQL
SELECT customer_id, order_date
FROM (
    SELECT DISTINCT ON (customer_id) *
    FROM orders
    ORDER BY customer_id, order_date DESC
)
ORDER BY customer_id;

This query first uses DISTINCT ON to ensure only one row per customer_id is selected, based on the latest order_date. The outer SELECT ... ORDER BY ensures that these rows are returned in a deterministic order.

Using ROW_NUMBER()
SQL
WITH ranked_orders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as row_num
    FROM orders
)
SELECT customer_id, order_date
FROM ranked_orders
WHERE row_num = 1;

This query uses a Common Table Expression (CTE) to first rank the rows by customer_id and then by order_date in descending order. The outer query filters for only those rows where row_num is 1, ensuring that we get the latest order date per customer.

Order By Requirements
DISTINCT ON: Requires an ORDER BY clause to specify which row should be selected when there are multiple matches.
ROW_NUMBER(): Similarly requires an ORDER BY to define the sequence of rows. The partition and order must align with your criteria for "latest" or any other grouping logic.

#### Common Mistakes
and Follow-Ups
Forgetting ORDER BY: Both methods require a proper ORDER BY clause to avoid undefined behavior.
Misunderstanding Row Numbering: Ensuring the correct partition and ordering in ROW_NUMBER() can be tricky, especially when dealing with complex queries involving multiple columns or window functions.
Ignoring Determinism: Without a proper order, both methods might return different results across executions due to non-deterministic behavior.

In summary, while DISTINCT ON is more concise for simple cases in PostgreSQL, ROW_NUMBER() offers greater flexibility and is widely applicable in various SQL databases. Both require careful consideration of the ORDER BY clause to ensure determinism.

**Comparison:**

```
Create Orders Table
INSERT Data
DISTINCT ON Example
SELECT customer_id,order_date FROM (SELECT *,ROW_NUMBER() OVER(PARTITION BY customer_idORDER BY order_date DESC) asrow_num FROM orders WHERErow_num = 1)
```

**Code:**

```sql
-- Example code
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date TIMESTAMP NOT NULL
);

INSERT INTO orders (customer_id, order_date) VALUES
(1, '2023-01-01 09:00'),
(1, '2023-01-02 10:00'),
(2, '2023-02-01 11:00');
```

**Example input / output:**

```text
-- DISTINCT ON Example
WITH ranked_orders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as row_num
    FROM orders
)
SELECT customer_id, order_date
FROM ranked_orders
WHERE row_num = 1;

-- Result:
+-----------+---------------------+
| customer_id |       order_date    |
+-----------+---------------------+
|           1 | 2023-01-02 10:00   |
|           2 | 2023-02-01 11:00   |
+-----------+---------------------+
```

**Data:**

```json
[
    {"id": 1, "customer_id": 1, "order_date": "2023-01-01 09:00"},
    {"id": 2, "customer_id": 1, "order_date": "2023-01-02 10:00"},
    {"id": 3, "customer_id": 2, "order_date": "2023-02-01 11:00"}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the SQL databasePostgreSQL?
5. Use DISTINCT ON
6. Use ROW_NUMBER()
7. RUN QUERY
8. RUN QUERY
9. Check Results
10. Check Results

