---
type: "practice"
difficulty: "Deep"
---

# You inherit a query with three OR'd conditions across different columns. The planner does a full scan. Walk through converting it to a UNION or bitmap-OR-friendly form and the index set needed.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we have a query with multiple OR conditions across different columns, which is causing a full table scan. The goal is to optimize this query by leveraging database indexing or transforming it into a form that can benefit from bitmap OR operations. We will walk through the process of converting the query and identifying the necessary indexes.

The core idea is to break down the OR conditions into separate queries, each with its own set of AND conditions. This transformation allows us to use multiple indexes more effectively, potentially reducing the need for a full table scan.

