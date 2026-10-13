---
type: "practice"
difficulty: "Intermediate"
---

# A query performs well until an ORM adds LEFT JOIN to eager-load an optional relation, flipping the plan. How do you keep eager-loading without defeating the primary index?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an ORM introduces a LEFT JOIN to eagerly load an optional relationship, it often disrupts the database's query optimization and can defeat the purpose of a primary index. This happens because the join operation may force the query planner to reconsider its execution plan, potentially leading to suboptimal performance. As a backend architect or SDE-3, our goal is to maintain efficient eager-loading while respecting the existing indexes.

In this scenario, we need to understand what's happening under the hood and how we can tweak the query or indexing strategy to preserve good performance. Let’s break down the problem step by step:

Understanding the Problem: The ORM adds a LEFT JOIN that was not there before, forcing the database to consider different execution plans. The primary index might no longer be used effectively because of the join condition.

#### Assumptions
and Context:

We have a database with an existing query and a specific index.
An ORM introduces a LEFT JOIN that eager-loads an optional relationship, which changes the performance characteristics.

#### First-Principles Solution

Analyze the Query Plan: First, use tools like EXPLAIN or SQL profiling to understand how the new query plan differs from the old one.
Evaluate Index Usage: Check if the index is still being used effectively by looking at the output of your query planner. If not, we need to consider whether the index needs adjustment or a different strategy.

Step-by-Step Reasoning:

Identify the Problematic Join Condition: Determine which part of the join condition is causing the issue.
Adjust the Index Strategy: Consider adding a composite index that includes all relevant columns used in the query and join conditions.
Optimize the Query Logic: Ensure that the ORM’s eager-loading logic does not overly complicate the query, possibly by using SELECT statements or specific query hints.

#### Implementation
Example:

Suppose we have a table orders with a primary index on (order_id), and now we need to eagerly load optional related records from another table products. The ORM adds a LEFT JOIN products ON orders.product_id = products.id.

We might initially see the query plan using an index scan followed by a nested loop join, which is inefficient. To address this:

Add a Composite Index: Create a composite index on (product_id, order_id) to help with the join condition.
SQL
CREATE INDEX idx_product_order ON orders (product_id, order_id);
Review ORM Configuration: Ensure that eager-loading is not overly complicating the query. For instance, use SELECT statements directly or optimize the ORM configuration.

#### Closing Summary
and Common Mistakes:

By carefully analyzing the query plan, understanding how indexes are used, and making strategic adjustments, we can maintain efficient performance even with eager-loading.

#### Common mistakes
include not recognizing the impact of LEFT JOIN on index usage or overcomplicating the solution by ignoring existing indexes.

**Comparison:**

```
Index Name	Columns
idx_product_order	product_id, order_id
```

**Code:**

```sql
-- Example of creating a composite index
CREATE INDEX idx_product_order ON orders (product_id, order_id);
```

**Flow:**

1. Identify Problematic Join
2. Analyze Query Plan
3. Evaluate Index Usage
4. Adjust Index Strategy
5. Optimize Query Logic

