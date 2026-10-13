---
type: "practice"
difficulty: "Deep"
---

# You need to make WHERE lower(username) LIKE ? || '%' fast for a prefix autocomplete. What index type and operator class would you choose and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize the WHERE lower(username) LIKE ? || '%' query, we need to ensure that the database can efficiently search through usernames in a case-insensitive manner. The key is to leverage an index type and operator class that support this specific pattern effectively.

#### Triage

The question asks for an index type and operator class to optimize a WHERE lower(username) LIKE ? || '%' query. This involves understanding how the database handles case-insensitivity, wildcard searches, and efficient indexing. We need to consider trade-offs between performance and storage costs.

#### First-Principles Solution

Choose the Right Index Type:

For case-insensitive search patterns, a B-tree index on lower(username) is not ideal because it does not support LIKE queries well.
Instead, use a GiST (Generalized Search Tree) index with an appropriate operator class.

Operator Class Selection:

The text_pattern_ops operator class allows us to use the LIKE operator effectively while preserving case insensitivity.
We will create a partial index that only includes rows matching the LIKE pattern, optimizing for fast prefix searches.

Index Definition:

SQL
CREATE INDEX idx_username_prefix ON users USING GiST (lower(username) gist_ops_text_pattern_ops);

Explanation of Choices:

GiST indexes support efficient range queries and pattern matching.
The gist_ops_text_pattern_ops operator class ensures that the index understands the LIKE ? || '%' syntax correctly.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Why GiST? It supports efficient range queries, including LIKE patterns with a wildcard at the end. B-tree indexes do not handle this pattern well.
Partial Indexes: By creating a partial index, we reduce storage costs and improve performance for specific query types.
Operator Class text_pattern_ops: This ensures that the index understands case-insensitive matching.

#### Common Mistakes
Using B-Tree Indexes Incorrectly: B-tree indexes do not support LIKE queries with a wildcard at the end efficiently. Using them in this context would result in full table scans.
Ignoring Case Insensitivity: Failing to account for case insensitivity can lead to inefficient query execution plans.

#### Trade-Offs
Storage Cost: GiST indexes may require more storage space compared to B-tree indexes, but the performance benefits often outweigh this cost.
Complexity: Using operator classes and partial indexes introduces additional complexity in schema design and maintenance.

**Code:**

```sql
CREATE INDEX idx_username_prefix ON users USING GiST (lower(username) gist_ops_text_pattern_ops);
```

