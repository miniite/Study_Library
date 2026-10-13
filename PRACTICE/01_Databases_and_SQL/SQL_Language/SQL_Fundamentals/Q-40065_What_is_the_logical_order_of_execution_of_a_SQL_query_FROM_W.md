---
type: "practice"
difficulty: "Basic"
---

# What is the logical order of execution of a SQL query (FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY), and why does it matter when writing transforms?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GROUP BY vs PARTITION BY](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a SQL query, understanding the logical order of execution is crucial to ensure efficient data processing and correct results. The sequence follows these steps: FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY. Each step filters, aggregates, selects, and sorts data in a specific order.

The logical flow starts with FROM, which specifies the tables involved in the query. Then, WHERE filters rows based on conditions before any further operations are performed. Next, GROUP BY groups the filtered results by one or more columns, preparing for aggregation using functions like SUM, AVG, etc. The HAVING clause filters these grouped results based on aggregate values. Following this comes SELECT, which specifies the columns and calculations to be returned from the query. Finally, ORDER BY sorts the result set.

Understanding why each step matters is key to optimizing queries and ensuring correctness:

FROM: Specifies the data sources; crucial for accurate transformations.
WHERE: Filters out irrelevant rows early on, improving performance by reducing the number of operations later.
GROUP BY and HAVING: Ensure that aggregations are applied correctly based on specific conditions.
SELECT: Defines what data is returned. Incorrect usage can lead to unnecessary computations or missing results.
ORDER BY: Ensures the final result set is presented in a meaningful order, but it happens last so does not affect earlier steps.

**Code:**

```sql
-- Example SQL query
SELECT customer_id, SUM(order_amount) AS total_spent
FROM orders
WHERE order_date >= '2023-01-01'
GROUP BY customer_id
HAVING SUM(order_amount) > 500
ORDER BY total_spent DESC;
```

**Example input / output:**

```text
-- Input (example schema and data)
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE,
    order_amount DECIMAL(10, 2)
);

INSERT INTO orders (order_id, customer_id, order_date, order_amount) VALUES
(1, 101, '2023-01-05', 150.00),
(2, 102, '2023-02-10', 80.00),
(3, 101, '2023-02-15', 60.00),
(4, 102, '2023-01-15', 200.00);

-- Output
+------------+--------------+
| customer_id | total_spent   |
+------------+--------------+
|          101 |        210.00 |
|          102 |         80.00 |
+------------+--------------+
```

**Flow:**

1. Query Execution Flow
2. FROM- Data source specification
3. WHERE- Early row filtering
4. GROUP BY- Aggregation preparation
5. HAVING- Grouped result filtering
6. SELECT- Column and calculationdefinition
7. ORDER BY- Final sorting

