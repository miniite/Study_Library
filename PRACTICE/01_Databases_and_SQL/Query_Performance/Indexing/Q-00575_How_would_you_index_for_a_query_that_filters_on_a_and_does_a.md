---
type: "practice"
difficulty: "Deep"
---

# How would you index for a query that filters on a=? and does a LIKE 'prefix%' on b — can one composite index serve both?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When designing indexes for queries involving filtering a = ? and performing a LIKE 'prefix%' search on column b, a composite index can indeed be used to serve both purposes effectively, provided certain conditions are met. This solution leverages the concept of covering indexes to minimize disk I/O operations and ensure efficient query execution.

#### Triage

The question asks whether a single composite index can handle two distinct types of queries: one that filters on an exact match (a = ?) and another that uses LIKE 'prefix%' for partial matching. This involves understanding how database engines use indexes, specifically composite indexes and the concept of covering indexes.

#### First-Principles Solution

Understanding the Queries:

The first query is a simple equality filter on column a.
The second query uses LIKE 'prefix%', which is a partial match starting with prefix.

Composite Index Design:

A composite index can be created on columns (a, b). This allows the database engine to use the index efficiently for both queries.

Index Usage Analysis:

For the equality filter a = ?, the database will stop at the first row where a matches the query value.
For the LIKE 'prefix%' search, the index can be used to quickly locate rows starting with prefix in column b.

Covering Index Consideration:

A covering index includes all columns needed by a query, thus reducing the need for additional disk I/O operations.
If both queries require minimal or no additional columns from other tables, the composite index can be a covering index.

Index Structure:

The composite index would typically have a as the leading column and b as the trailing column: (a, b).

#### Closing Summary

By creating a composite index on columns (a, b), we can optimize both types of queries effectively. This single index allows the database engine to quickly locate relevant rows for filtering by a and perform efficient searches starting with prefix in b. However, it’s crucial to consider other factors such as data distribution, cardinality, and potential updates that may impact the index's efficiency.

#### Common Mistakes/Follow-Ups

Mistake 1: Ignoring the prefix nature of the LIKE 'prefix%' query. Without a leading column in the composite index, the engine might perform a full table scan.

#### Follow-Up
Always ensure that the most selective part of the filter is included first in the index definition.

Mistake 2: Overlooking the possibility of a covering index if other columns are not needed by the query.

#### Follow-Up
Verify whether all necessary columns for the queries are covered by the index to avoid additional disk I/O operations.

**Code:**

```sql
CREATE INDEX idx_a_b ON table_name(a, b);
```

**Data:**

```json
{
  "table_name": {
    "columns": ["a", "b"],
    "rows": [
      {"a": 1, "b": "prefix_sample"},
      {"a": 2, "b": "other_value"},
      {"a": 3, "b": "prefix_another"}
    ]
  }
}
```

