---
type: "practice"
difficulty: "Intermediate"
---

# What are the downsides of keeping redundant or duplicate indexes?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this interview question, we'll explore the disadvantages of maintaining redundant or duplicate indexes in a database system. The key points to cover include understanding what redundant indexes are, why they might be created, and the potential performance impacts on read and write operations.

Firstly, let's clarify that an index is a data structure designed to improve query performance by providing quick access to rows based on specific columns or combinations of columns. Redundant or duplicate indexes occur when multiple indexes exist that provide overlapping coverage for queries. These can be created unintentionally due to poor design practices, or intentionally as part of complex database management strategies.

#### Triage
and Assumptions

The core issue here is to identify the downsides associated with maintaining redundant indexes. The interviewer is likely probing your understanding of index management, query optimization, and the trade-offs between performance gains and resource costs.

#### Assumptions
We are discussing a typical relational database system.
Indexes can significantly speed up read operations but may slow down write operations due to the overhead of maintaining them.
Redundant indexes refer to those that provide overlapping coverage for queries, potentially leading to unnecessary data duplication and management overhead.

#### First-Principles Solution

To address redundant indexes effectively, let's reason through this step-by-step:

Index Overhead on Writes:

Each write operation (INSERT, UPDATE, DELETE) must update all relevant indexes. This can be a significant performance hit, especially for large databases.
For example, if you have two overlapping indexes covering different columns but sharing some common columns, each write to those shared columns will trigger updates in both indexes.

Increased Storage Requirements:

Redundant indexes consume additional storage space. This is particularly problematic in environments with limited storage capacity or high data volume.

#### Cons
ider a scenario where an index on customer_id and another one on email, but both include the first_name column. Maintaining these two separate indexes will double the storage requirement for the first_name column.

Complexity in Index Maintenance:

Managing multiple redundant indexes can complicate maintenance tasks, such as index rebuilding or optimization.
Debugging performance issues related to overlapping indexes can be challenging due to their complex interactions during read and write operations.

Query Performance Impact:

While redundant indexes might seem beneficial for certain queries, they often lead to suboptimal query plans being chosen by the database optimizer.
For instance, if a query benefits from one index but not another, the database may unnecessarily use both, leading to wasted resources and slower performance.

Impact on Cache Efficiency:

Overlapping indexes can reduce cache efficiency because the database might end up caching redundant data structures in memory.
This can lead to increased memory usage and reduced overall system responsiveness.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while redundant or duplicate indexes may seem like a straightforward way to optimize query performance, they often introduce significant downsides such as increased storage overhead, write latency, maintenance complexity, and suboptimal query plans. It’s crucial to carefully manage index design to ensure that each index serves a unique purpose and contributes positively to overall system performance.

#### Common mistakes
include over-indexing without considering the trade-offs, failing to monitor and adjust indexes regularly, or not understanding how overlapping indexes interact with different queries. Follow-ups might involve discussing best practices for index management, tools for identifying redundant indexes, or strategies for optimizing query plans in a production environment.

**Comparison:**

```
Index Name	Columns Covered
idx_customer_email	customer_id, email
idx_customer_first_name	customer_id, first_name, email
```

**Code:**

```sql
-- Example of an index that could be considered redundant
CREATE INDEX idx_customer_email ON customers (customer_id, email);
CREATE INDEX idx_customer_first_name ON customers (customer_id, first_name, email);
```

**Flow:**

1. Yes
2. No
3. Suboptimal Plan
4. Optimal Plan
5. Write Operation
6. Index Overlap?
7. Update Multiple Indexes
8. Update Single Index
9. Increased Write Latency
10. Query Performance Impact?
11. Resource Wastage
12. Improved Performance

