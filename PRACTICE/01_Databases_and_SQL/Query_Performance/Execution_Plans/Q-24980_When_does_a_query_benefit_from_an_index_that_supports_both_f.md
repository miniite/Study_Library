---
type: "practice"
difficulty: "Intermediate"
---

# When does a query benefit from an index that supports both filtering and ordering to avoid a separate Sort node?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's first understand the core concepts involved—query execution plans, indexing, and how these interact with sorting operations in SQL databases. We'll then reason through the scenarios where an index that supports both filtering and ordering can benefit a query.

#### Triaging the Question

The key here is to identify when using an index for both filtering (WHERE clause) and ordering (ORDER BY clause) can avoid the need for a separate Sort node during execution, thus potentially improving performance. The interviewer is probing your understanding of how indexes work within SQL engines, specifically in terms of reducing the overhead associated with sorting.

#### Assumptions
We are working with a relational database management system (RDBMS) that uses an optimizer to generate execution plans.
The query involves both filtering and ordering operations.
The index can support the WHERE clause conditions as well as the ORDER BY clause.
Step-by-Step Solution

Understanding Indexes: An index in a database is a data structure that improves the speed of data retrieval operations on a database table at the cost of additional writes, storage space and maintenance during updates.

Filtering vs. Ordering:

Filtering: Using an index for filtering (WHERE clause) allows the database to quickly locate rows that match certain conditions without scanning the entire table.
Ordering: Using an index for ordering (ORDER BY clause) can eliminate the need for a separate Sort node in the execution plan.

Index Selection:

A multi-column index (covering both columns used in the WHERE and ORDER BY clauses) is ideal because it can be used to satisfy both filtering and ordering operations.
Single-column indexes on each of the columns separately might work, but they could require additional sorting steps or scans.

Execution Plan Analysis:

The database optimizer will consider multiple factors such as index statistics (e.g., cardinality), cost estimates for different execution plans, and cache usage when deciding whether to use an index.
If the index supports both filtering and ordering efficiently, it can lead to a more optimized plan with fewer physical I/O operations.

#### Example Scenario

#### Cons
ider a Orders table where we want to retrieve all orders placed in the last month and sorted by order date:

SQL
SELECT * FROM Orders WHERE OrderDate >= '2023-10-01' ORDER BY OrderDate;

Suppose there is an index on (OrderDate). The database can use this index both for filtering (to find orders from the last month) and for ordering (to sort by OrderDate). This avoids a separate Sort node, reducing the overall complexity of the query execution.

#### Closing Summary

In summary, an index that supports both filtering and ordering operations is beneficial when:

The WHERE clause conditions can be matched using the index.
The ORDER BY clause can also use the same index without requiring additional sorting steps.
The cost of maintaining the index (in terms of write overhead) is outweighed by performance benefits.

#### Common Mistakes/Follow-Ups
Ignoring Index Maintenance Costs: While indexes improve read performance, they increase write and maintenance costs. Make sure to consider these trade-offs.
Overusing Full Table Scans: Always ensure that the index can significantly reduce the number of rows being scanned; otherwise, a full table scan might be more efficient.
Complex Queries: For complex queries involving multiple conditions or joins, an index that supports only part of the query may still require additional steps.

**Comparison:**

```
Yes
No
Index Used for Filtering
Is the Index Also Suitable forSorting?
Execution Plan with Index
Separate Sort Node Required
```

**Code:**

```sql
-- Example SQL Query
SELECT * FROM Orders WHERE OrderDate >= '2023-10-01' ORDER BY OrderDate;
```

**Example input / output:**

| Input Query | Execution Plan |
|-------------|----------------|
| `SELECT * FROM Orders WHERE OrderDate >= '2023-10-01' ORDER BY OrderDate` | Index Scan + Sort (Avoided) |

**Data:**

```json
{
  "Orders": [
    {"OrderID": 1, "OrderDate": "2023-10-05"},
    {"OrderID": 2, "OrderDate": "2023-10-06"},
    {"OrderID": 3, "OrderDate": "2023-09-01"}
  ]
}
```

