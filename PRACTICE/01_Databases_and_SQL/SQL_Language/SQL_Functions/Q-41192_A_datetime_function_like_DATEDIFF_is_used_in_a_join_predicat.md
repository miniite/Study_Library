---
type: "practice"
difficulty: "Deep"
---

# A datetime function like DATEDIFF is used in a join predicate and the optimizer picks a nested loop with a scan. How would you refactor to make the join index-friendly?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ UDF Performance](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a SQL query uses DATEDIFF in its join condition, it often leads to suboptimal execution plans like nested loops with scans because these functions are not easily evaluated by the database optimizer during the planning phase. To refactor such queries and make them index-friendly, we need to optimize the join predicate by leveraging indexed columns or creating custom SQL functions that can be pushed down to the storage layer. This approach ensures that the query planner can more effectively use indexes for joining operations.

Firstly, let’s understand what’s happening in the current scenario:

The optimizer cannot efficiently evaluate DATEDIFF directly during the join planning.
As a result, it resorts to nested loop joins or scans, which are not always the most efficient strategies.

To improve this, we can consider several approaches:

1. Pushing Down Calculations

One effective method is to push down the calculation of DATEDIFF into the storage layer by creating an indexed column that reflects the difference between two datetime fields. For instance:

SQL
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATETIME,
    delivery_date DATETIME,
    days_between_order_and_delivery AS DATEDIFF(delivery_date, order_date) STORED
);

This indexed column days_between_order_and_delivery allows the optimizer to use an index on this field for join operations.

2. Using Window Functions

Alternatively, we can use window functions like LAG or LEAD in a subquery to compute the difference and then join with the original table:

SQL
WITH order_diff AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        delivery_date,
        DATEDIFF(delivery_date, order_date) as days_between_order_and_delivery
    FROM orders
)
SELECT
    o.order_id,
    o.customer_id
FROM
    customers c
JOIN
    (SELECT * FROM order_diff WHERE days_between_order_and_delivery > 0) o
ON
    c.customer_id = o.customer_id;

Here, the order_diff CTE computes the difference and filters out unnecessary rows before joining with the customer table.

3. Custom SQL Functions

For more complex scenarios, we can create a custom scalar function that pushes down to the storage layer:

SQL
CREATE FUNCTION days_between_dates(datetime1 DATETIME, datetime2 DATETIME)
RETURNS INTEGER DETERMINISTIC
BEGIN
    RETURN DATEDIFF(datetime2, datetime1);
END;

Then use this function in the join condition:

SQL
SELECT
    o.order_id,
    c.customer_id
FROM
    customers c
JOIN
    orders o ON c.customer_id = o.customer_id AND days_between_dates(o.order_date, o.delivery_date) > 0;

By pushing down these calculations, we enable the optimizer to use indexes more effectively for join operations.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, refactoring queries that use DATEDIFF in join conditions can significantly improve performance by leveraging indexed columns or custom SQL functions. Common mistakes include:

Overlooking the need to push down calculations to storage.
Failing to consider window functions as an alternative approach.

During follow-ups, it's crucial to discuss potential trade-offs such as increased write operations for maintaining computed fields and ensuring that the additional computations do not negatively impact read performance. Additionally, testing with realistic data is essential to verify the effectiveness of these optimizations in real-world scenarios.

**Code:**

```sql
-- Example of creating an indexed column
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATETIME,
    delivery_date DATETIME,
    days_between_order_and_delivery AS DATEDIFF(delivery_date, order_date) STORED
);

-- Using a window function in a subquery
WITH order_diff AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        delivery_date,
        DATEDIFF(delivery_date, order_date) as days_between_order_and_delivery
    FROM orders
)
SELECT
    o.order_id,
    o.customer_id
FROM
    customers c
JOIN
    (SELECT * FROM order_diff WHERE days_between_order_and_delivery > 0) o
ON
    c.customer_id = o.customer_id;

-- Creating a custom SQL function
CREATE FUNCTION days_between_dates(datetime1 DATETIME, datetime2 DATETIME)
RETURNS INTEGER DETERMINISTIC
BEGIN
    RETURN DATEDIFF(datetime2, datetime1);
END;
```

