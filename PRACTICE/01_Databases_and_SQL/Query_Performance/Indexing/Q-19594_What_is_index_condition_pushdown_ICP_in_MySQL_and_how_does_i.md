---
type: "practice"
difficulty: "Expert"
---

# What is index condition pushdown (ICP) in MySQL and how does it reduce heap fetches?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Index Condition Pushdown (ICP) in MySQL is a powerful optimization technique that allows the database engine to perform certain conditions directly on indexed columns, thereby reducing unnecessary heap fetches and improving overall query performance. Essentially, ICP helps by pushing down parts of the WHERE clause or JOIN condition from the main execution plan into the storage layer where the data resides.

#### Triaging the Question

The question asks for a detailed explanation of ICP in MySQL and how it reduces heap fetches. The interviewer is likely probing your understanding of query optimization, indexing strategies, and storage engine mechanics. We need to assume that you are familiar with basic SQL concepts and common database performance issues.

#### First-Principles Solution

Let's start by defining the problem: when a query involves conditions on columns not present in an index (often referred to as "heap fetches"), MySQL has to perform additional I/O operations to retrieve the full row data from the heap. This can be inefficient, especially for large tables.

Step 1: Understanding Index Condition Pushdown

Index Condition Pushdown works by allowing the storage engine to evaluate conditions on indexed columns directly without fetching the entire row from the heap. For example, if a query has a WHERE clause that filters based on an indexable column, MySQL can push down this condition and use it to filter data at the storage level.

Step 2: How ICP Reduces Heap Fetches

When ICP is applied:

Filtering: The storage engine evaluates the condition on indexed columns.
Row IDs Collection: It collects only the row identifiers (e.g., primary key values) that match the condition.
Heap Fetch Reduction: Instead of fetching every row from the heap, MySQL retrieves only the rows corresponding to the collected row IDs.

This approach significantly reduces the number of heap fetches, improving query performance and reducing I/O overhead.

Step 3: Common Scenarios

ICP is particularly beneficial in JOIN operations or queries with complex WHERE clauses. For instance:

JOIN Optimization: When joining tables based on a common indexed column.
Range Queries: Filtering rows within a specific range of an indexed column.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, ICP is a critical optimization technique that enhances query performance by reducing unnecessary heap fetches. It leverages indexes to filter data at the storage level, thereby minimizing I/O operations. Understanding how to enable and utilize ICP can significantly improve your database's efficiency.

Some common mistakes include:

Not recognizing situations where ICP could be applied.
Overlooking the need for appropriate indexing to make ICP effective.
Misconfiguring MySQL settings that might disable or limit ICP optimizations.

For follow-ups, you might discuss advanced topics such as parallel index condition pushdown, its limitations with certain types of indexes (e.g., partial indexes), and how it interacts with other query optimization techniques like query partitioning.

**Code:**

```sql
-- Example Query Without ICP
SELECT * FROM customers WHERE state = 'CA' AND balance > 100;

-- Potential Optimized Query With ICP
SELECT * FROM customers USE INDEX (idx_state_balance) WHERE state = 'CA' AND balance > 100;
```

**Example input / output:**

```text
Input: SELECT * FROM customers WHERE state = 'CA' AND balance > 100;

Without ICP:
- Full table scan (heap fetches for all rows)
- Filter based on the condition in memory

With ICP:
- Evaluate 'state = "CA"' at storage level, collect row IDs
- Fetch only relevant rows from heap based on collected IDs
```

**Flow:**

1. No
2. Yes
3. Query
4. Is ICP Enabled?
5. Full Table Scan
6. Evaluate Condition at StorageLevel
7. Collect Row IDs
8. Fetch Relevant Rows fromHeap
9. Filter in Memory

