---
type: "practice"
difficulty: "Intermediate"
---

# What is a materialized view and how does it differ from a regular view?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Materialized View Trade-offs](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A materialized view (MV) is a precomputed result set stored as a database table, which can significantly improve query performance by avoiding repeated computation of the same data. Unlike regular views, MVs are stored physically and can be updated to reflect changes in the underlying tables.

Question Overview: The question asks about the concept of materialized views (MVs) and their differences from regular views. The interviewer is likely probing your understanding of how databases manage query performance through different types of views. Key assumptions include familiarity with database concepts like tables, queries, and indexes.

Step-by-Step Solution From First Principles:

Definition and Basic Use Case: A materialized view (MV) is a database object that stores the result set of a query as a table in the database. Unlike regular views, which are just SQL queries with no stored data, MVs have their own physical storage space where the results are cached.

Regular View vs Materialized View:

Regular View: A regular view is simply a virtual table derived from one or more base tables via a query. It does not store any of its data; it just stores and returns the result set each time someone queries it.
Materialized View: An MV stores the results of a query in a physical database table, which can be queried directly like any other table.

Use Cases:

Performance Improvement: MVs are ideal for complex queries that run frequently or take a long time to execute. By storing the result set, subsequent queries on the same data can be much faster.
Data Consistency: Regular views do not enforce referential integrity rules and might return inconsistent results if the underlying tables change. MVs ensure consistent and accurate data.

Maintenance Considerations:

Refresh Mechanism: MVs need to be refreshed periodically to ensure they remain up-to-date with the latest data changes in their source tables.
Disk Space Usage: Since MVs store data, they can consume significant disk space. This should be managed carefully, especially in environments where storage is a critical resource.

#### Common Mistakes

Ignoring the need for refreshing MVs, which can lead to stale data.
Misusing MVs for complex queries that change frequently or are not repeatable, leading to unnecessary disk space usage and maintenance overhead.

#### Example Scenario
Suppose we have a query that joins three large tables to generate a report on sales performance. By creating an MV from this query, subsequent requests for the same data can be much faster.

SQL
CREATE MATERIALIZED VIEW sales_report AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
GROUP BY customer_id;

#### Closing Summary
Materialized views provide a powerful tool for improving query performance by caching the result sets of complex or frequently executed queries. However, they require careful management to ensure data consistency and efficient storage usage.

**Code:**

```sql
CREATE MATERIALIZED VIEW sales_report AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
GROUP BY customer_id;
```

