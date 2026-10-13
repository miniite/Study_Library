---
type: "practice"
difficulty: "Intermediate"
---

# What is the impact of joining tables in a different order and does the optimizer reorder joins for you?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL databases, understanding how join orders affect query performance is crucial. The key takeaway here is that the order in which you join tables can significantly impact execution time and resource utilization. Additionally, database optimizers often do try to reorder joins to optimize performance, but this is not always guaranteed or sufficient for all scenarios.

#### Triage

The question asks about two main points:

Impact of Join Order: How the order in which tables are joined affects query performance.
Optimizer Reordering: Whether and how database optimizers automatically reorder joins to optimize queries.

To answer these, we need to consider the structure of SQL execution plans, the cost-based optimizer (if applicable), and potential manual optimizations.

#### First-Principles Solution

Firstly, understand that a join operation in SQL involves combining rows from two or more tables based on a related column. The order in which you specify tables can drastically change the efficiency of these operations because:

Nested Loop Joins: A nested loop join (the simplest and least efficient method) fetches each row from one table, compares it with all rows in another table, and then selects matching rows.
Hash Join: This method creates a hash table from one table and uses this to quickly find matches in the other table. It can be more efficient than nested loops but still depends on the data distribution.
Merge Join: Efficient for sorted tables where it compares row by row based on join keys.

When you specify joins in a different order, you change which of these methods is used and how many operations are performed. Optimizers try to choose the most cost-effective method, considering factors like table sizes, indexes, and statistics.

Detailed Explanation

Optimizer Reordering: Most modern database management systems (DBMS) like PostgreSQL, MySQL, or Oracle have optimizers that attempt to reorder joins in a way that minimizes overall query execution costs. However:

The optimizer's effectiveness depends on the completeness of statistics collected about table sizes and distributions.
Optimizers might not always find the absolute optimal order, especially with complex queries involving multiple join conditions.

Manual Reordering: You can manually reorder joins to influence the optimizer’s choice, which is particularly useful in cases where:

The default optimization does not yield good performance.
Specific join orders are known to be more efficient based on historical data or experience.

Performance Impact:

Changing join order can reduce the number of rows processed early in the query, thus saving computational resources.
Improper ordering can lead to unnecessary full table scans and higher overall costs.

#### Example Scenario

#### Cons
ider a simple schema with three tables: orders, customers, and products. Suppose you want to find all orders placed by customers who have purchased product 123. The query might look like this:

SQL
SELECT * FROM orders
JOIN customers ON orders.customer_id = customers.id
JOIN products ON orders.product_id = products.id
WHERE products.id = 123;

If you change the order to:

SQL
SELECT * FROM products
JOIN orders ON products.id = orders.product_id
JOIN customers ON orders.customer_id = customers.id
WHERE products.id = 123;

The optimizer might choose a different join method or strategy, which could affect performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the order of joins can significantly impact query performance. Optimizers try to reorder joins for optimization but may not always find the best solution. It’s important to understand these mechanisms and consider manual reordering in complex queries.

#### Common mistakes
include:

Assuming the default join order is optimal without testing.
Not considering the data distribution or statistics when relying on automatic optimization.
Ignoring the potential benefits of manual query tuning.

#### Follow-ups
could involve discussing specific database engines, the role of indexes, and more advanced techniques like query rewriting for performance optimization.

**Comparison:**

```
table "products" {
    id (int, primary key)
    name (varchar)
}

table "orders" {
    id (int, primary key)
    product_id (int, foreign key to products(id))
    customer_id (int, foreign key to customers(id))
}

table "customers" {
    id (int, primary key)
    name (varchar)
}
```

**Code:**

```sql
-- Example of an optimized query with correct join order
SELECT * FROM products
JOIN orders ON products.id = orders.product_id
JOIN customers ON orders.customer_id = customers.id
WHERE products.id = 123;

-- Default order which might perform worse due to different optimization strategy
SELECT * FROM orders
JOIN customers ON orders.customer_id = customers.id
JOIN products ON orders.product_id = products.id
WHERE products.id = 123;
```

**Example input / output:**

```text
Input:
SELECT * FROM products
JOIN orders ON products.id = orders.product_id
JOIN customers ON orders.customer_id = customers.id
WHERE products.id = 123;

Output (Sample):
+----+------------+-------+---------+-----------+
| id | name       | id    | product_id| customer_id|
+----+------------+-------+-----------+------------+
| 123| Product B  | 101   | 123      | 2          |
+----+------------+-------+-----------+------------+
```

**Data:**

```json
{
  "products": [
    {"id": 1, "name": "Product A"},
    {"id": 123, "name": "Product B"}
  ],
  "orders": [
    {"id": 100, "product_id": 1, "customer_id": 1},
    {"id": 101, "product_id": 123, "customer_id": 2}
  ],
  "customers": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Success
4. Failure
5. Optimizer Receives Query
6. Is Statistics Available?
7. Optimizer Chooses Join OrderBased on Cost
8. Use Default Join Order orManual Tuning
9. Execution Plan with OptimalJoins
10. Execution Plan with DefaultJoins or Tuned Manually
11. Query Runs Efficiently
12. Query May Run Less Efficiently
13. Optimized Performance
14. Performance Issues, ManualTuning Needed
15. End
16. Review Query and Optimize
17. Improved Understanding ofDatabase Mechanics

