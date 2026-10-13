---
type: "practice"
difficulty: "Deep"
---

# Why is predicate and aggregate pushdown critical for OLAP query cost, and how does a wrapping subquery or non-sargable expression silently defeat it — how would you catch it in review?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Physical Design for OLTP vs OLAP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Physical_Design_for_OLTP_vs_OLAP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
OLTP (Online Transaction Processing) and OLAP (Online Analytical Processing) are two fundamentally different approaches to handling data, each optimized for distinct use cases. Pushdown of predicates and aggregates is a key optimization technique in OLAP systems to improve query performance by reducing the amount of data processed. However, wrapping subqueries or non-sargable expressions can silently defeat these optimizations, leading to suboptimal query execution plans.

#### Triage
The question delves into understanding why pushdown optimizations are crucial for OLAP and how certain coding practices can undermine them. The interviewer is probing your knowledge of query optimization techniques, the impact of different SQL constructs on performance, and your ability to identify potential pitfalls in query design. Assumptions include familiarity with SQL, database indexing, and common anti-patterns.

#### First Principles Solution

Understanding Pushdown Optimization: In OLAP systems, queries often involve complex aggregation operations over large datasets. To optimize these queries, the database engine pushes down predicates (conditions) and aggregates to lower levels in the execution plan, minimizing the amount of data transferred from higher layers. This reduces the overall computational load and speeds up query execution.

Predicate Pushdown: When a predicate is pushed down, it filters data early in the processing pipeline before any aggregation occurs. For example, if you have an OLAP query like SELECT SUM(column) FROM table WHERE condition, pushing the WHERE clause down to the storage layer ensures that only relevant rows are aggregated, significantly reducing the amount of data processed.

Aggregate Pushdown: Aggregate pushdown involves moving parts of the aggregation operation to where the data is stored, rather than performing all aggregations in memory or on the server side. This can be achieved through materialized views or by optimizing queries that involve subqueries and joins.

Wrapping Subquery or Non-Sargable Expressions: These constructs can undermine pushdown optimizations because they prevent efficient execution plans from being generated. For instance, a subquery like WHERE column IN (SELECT value FROM another_table) forces the database to perform an additional join operation before applying the predicate, which defeats both predicate and aggregate pushdown.

Detection Mechanisms: To catch such issues in code reviews:

Identify and Eliminate Non-Sargable Expressions: Ensure that conditions are based on indexed columns or simple comparisons.
Simplify Subqueries: Replace correlated subqueries with joins where possible, as the latter can be more efficiently optimized by the database engine.
Use Indexes Wisely: Ensure that indexes support the query patterns and use them judiciously to avoid index scanning.
Static Code Analysis Tools: Utilize tools like Linter for SQL queries to automatically flag potential issues.

#### Example Scenario

SQL
-- Original Query with Non-Sargable Subquery
SELECT SUM(amount) FROM sales WHERE order_id IN (SELECT id FROM orders WHERE status = 'completed');

-- Optimized Query with Join
SELECT SUM(s.amount)
FROM sales s
JOIN orders o ON s.order_id = o.id
WHERE o.status = 'completed';

#### Closing Summary
Pushdown of predicates and aggregates is critical for OLAP query performance as it minimizes data transfer and processing overhead. Wrapping subqueries or using non-sargable expressions can silently defeat these optimizations by forcing the database to perform additional, less efficient operations. Detecting such issues requires careful review of SQL queries, judicious use of indexes, and leveraging static code analysis tools.

#### Common Mistakes/Follow-Ups

Mistake: Failing to recognize non-sargable expressions or correlated subqueries.

#### Follow-Up
Discussing advanced optimization techniques like materialized views and their trade-offs in OLAP systems.
Mistake: Over-indexing leading to performance degradation due to excessive overhead.

#### Follow-Up
Exploring the balance between indexing and query optimization, and when not to index.

