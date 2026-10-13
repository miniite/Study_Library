---
type: "practice"
difficulty: "Deep"
---

# What is a descending index and when does adding DESC on a column let the planner avoid a sort for ORDER BY ... DESC?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Index Access Paths](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you add DESC to a column in an SQL query, it indicates that the results should be ordered in descending order. A descending index (or index with a descending direction) is a type of index where the data is stored in reverse order compared to how it's stored without the DESC. This can significantly impact the performance and behavior of your queries. Specifically, when you use ORDER BY ... DESC, adding an appropriate descending index can sometimes avoid the need for additional sorting operations by the database planner.

#### Triage

The question is asking about the conditions under which a descending index can help the database query planner (such as PostgreSQL's) avoid performing a sort operation during an ORDER BY ... DESC clause. The key here involves understanding how indexes and the query planner interact, particularly in scenarios where sorting is necessary but potentially redundant.

#### Assumptions
We are working with a relational database management system (RDBMS) that supports descending indexes.
The column or columns specified for the DESC keyword are part of an existing index or can be included in one.
The query planner uses information about available indexes to make decisions on how to execute queries.

#### First-Principles Solution
Step 1: Understanding Indexes and Sorting

Indexes improve read performance by reducing the number of disk accesses required to retrieve data. A descending index essentially stores a column in reverse order, which can be beneficial for certain query patterns.

Step 2: Query Planner Decisions

The database query planner evaluates multiple factors before deciding how to execute a query:

Cost Estimation: The planner estimates the cost (time and resources) of different execution plans.
Index Usage: It checks if an existing index can be used for the ORDER BY clause.
Sorting Overhead: If no suitable index exists, sorting might be necessary.
Step 3: Using a Descending Index

When you specify DESC in your query, the planner will consider whether there is already an index that stores data in descending order. If such an index is available and can satisfy the ORDER BY ... DESC clause without additional overhead:

The planner may choose to use this index directly.
This avoids the need for a separate sorting step, leading to improved performance.
Step 4: Practical Example

#### Cons
ider a simple example where you have a table orders with columns order_id, customer_id, and order_date. You frequently run queries like:

SQL
SELECT * FROM orders ORDER BY order_date DESC;

If there is an index on order_date in descending order, the planner can use this index to directly fetch the data in the desired order.

Step 5: Cost Considerations

While using a descending index can avoid sorting, it’s important to consider:

Index Maintenance: Descending indexes still require maintenance and updates, which could affect write performance.
Query Patterns: Not all queries benefit equally from descending indexes. For example, ORDER BY ... ASC might not use the same index.

#### Closing Summary

In summary, a descending index can help avoid additional sorting by allowing the query planner to directly leverage an existing index that stores data in reverse order. This optimization is particularly useful for frequently executed queries with large datasets where sorting would otherwise be costly.

#### Common Mistakes/Follow-Ups
Overusing Descending Indexes: Not all columns or tables benefit equally from descending indexes, and they can introduce unnecessary overhead.
Cost Analysis: Always perform cost analysis to determine if the benefits of using a descending index outweigh the costs.
Multiple Columns: Consider whether multiple columns might be more effectively indexed together rather than individually with DESC.

**Code:**

```sql
-- Example query using ORDER BY DESC and a potential descending index
SELECT * FROM orders ORDER BY order_date DESC;
```

