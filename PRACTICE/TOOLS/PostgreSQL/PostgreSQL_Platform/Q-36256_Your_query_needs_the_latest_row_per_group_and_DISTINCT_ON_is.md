---
type: "practice"
difficulty: "Deep"
---

# Your query needs the latest row per group and DISTINCT ON is fast but you're told window functions are more standard; compare DISTINCT ON versus ROW_NUMBER() = 1 in a subquery on both plan cost and index usage.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare DISTINCT ON with using ROW_NUMBER() in PostgreSQL, let's first understand what each approach does and then delve into their performance differences based on query plans and index usage. The core of this comparison revolves around the cost and efficiency of fetching the latest row per group.

DISTINCT ON is a powerful feature that allows you to select one row from a group of rows in a SELECT statement, effectively acting as an inline DISTINCT with an ORDER BY clause. It's straightforward and works well for simple use cases but can have limitations when dealing with complex conditions or large datasets.

On the other hand, using ROW_NUMBER() involves creating a subquery where you assign a unique row number to each row within a partition of the result set based on your order by criteria. Then, in the outer query, you filter for rows where ROW_NUMBER() = 1. This approach is more flexible and can handle complex queries but might be less efficient due to its additional computation.

Now, let's compare these methods from a performance perspective:

#### Triage
Query Plan Cost: We will examine how each method performs in terms of cost by looking at the EXPLAIN output.
Index Usage: We will analyze which approach utilizes indexes more effectively and if there are any potential bottlenecks.

#### Assumptions
Both queries assume a table with multiple rows per group, where we want to fetch the latest row based on some timestamp or order column.
Step-by-Step Solution
1. Example Table

#### Cons
ider an orders table:

SQL
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP NOT NULL,
    total DECIMAL(10,2)
);
2. Query with DISTINCT ON

The query using DISTINCT ON would look like this:

SQL
EXPLAIN SELECT DISTINCT ON (customer_id) * FROM orders ORDER BY customer_id, order_date DESC;
3. Query with ROW_NUMBER()

The query using ROW_NUMBER() might look something like this:

SQL
WITH ranked_orders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rn
)
SELECT * FROM ranked_orders WHERE rn = 1;
4. Analyzing the Plan Cost and Index Usage
DISTINCT ON Query Plan

Running EXPLAIN on the DISTINCT ON query:

SQL
EXPLAIN SELECT DISTINCT ON (customer_id) * FROM orders ORDER BY customer_id, order_date DESC;

The output might show that PostgreSQL uses a unique scan or index scan to handle the DISTINCT ON operation.

ROW_NUMBER() Query Plan

Running EXPLAIN on the ROW_NUMBER() query:

SQL
EXPLAIN WITH ranked_orders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rn
)
SELECT * FROM ranked_orders WHERE rn = 1;

The output might reveal that PostgreSQL first computes the row numbers and then filters, which can be more costly in terms of additional sort operations.

#### Closing Summary

In summary, DISTINCT ON is generally faster and more efficient for fetching distinct rows with a simple order by clause. It leverages existing index scans effectively and minimizes additional computations. However, when dealing with more complex queries or large datasets, ROW_NUMBER() provides greater flexibility but at the cost of potentially higher plan costs and increased index usage.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that DISTINCT ON is always faster without considering the complexity of the order by clause.

#### Follow-Up
Discussing how different database systems handle these operations, such as differences in PostgreSQL vs. MySQL.

