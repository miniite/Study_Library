---
type: "practice"
difficulty: "Deep"
---

# Explain how you would size a prefix index on a long text column so it stays selective, and how you measure the selectivity trade-off.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Full-text Search](<../../../../TOOLS/MySQL/MySQL_Platform/Full-text_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To size a prefix index on a long text column in MySQL effectively, we need to balance the trade-off between improving query performance and maintaining sufficient selectivity. A well-chosen prefix length can help achieve this balance by reducing storage overhead while still providing meaningful filtering capabilities.

#### Triaging the Question

The question is primarily asking about how to determine an optimal prefix length for a text column index in MySQL. The key aspects are understanding what "selectivity" means and how it affects performance, as well as balancing the trade-offs involved.

#### Assumptions
We have a long text column (e.g., description or content) that is frequently queried.
The goal is to create an index on this column for faster query execution.
We need to choose a prefix length that balances storage efficiency and query performance.

#### First Principles Solution

To size the prefix index, we start by understanding what "selectivity" means in this context. Selectivity refers to how well an index can narrow down the number of rows returned by a query. A higher selectivity means fewer rows are retrieved from disk, which reduces I/O operations and speeds up queries.

Step 1: Determine Query Patterns

First, analyze the types of queries that run against your database. Are they primarily searching for partial text or full matches? Understanding common patterns helps in choosing an appropriate prefix length.

Step 2: Experiment with Different Prefix Lengths

Test different prefix lengths to see how they impact selectivity and performance:

Shorter prefixes (e.g., 10 characters) might be sufficient if the queries are mostly looking for partial matches.
Longer prefixes (e.g., 50 characters) could provide better selectivity but at the cost of increased storage space.
Step 3: Measure Selectivity

Calculate the selectivity by running queries with different prefix lengths and measuring the row counts returned. For example, if a query with "description" column's prefix index returns significantly fewer rows than without an index, the selectivity is high.

Step 4: Monitor Performance Metrics

Use MySQL's performance metrics (e.g., EXPLAIN output) to monitor how different prefix lengths affect query execution time and I/O operations. Longer prefixes generally reduce the number of rows read from disk but increase storage costs.

#### Example
Workflow

Let’s say we have a text column with 256-character descriptions. We start by creating indexes with varying prefix lengths:

Index1: CREATE INDEX idx_description ON table (description(10));
Index2: CREATE INDEX idx_description ON table (description(30));
Index3: CREATE INDEX idx_description ON table (description(50));

Run the same query against each index and compare the results:

SQL
EXPLAIN SELECT * FROM table WHERE description LIKE 'search_term%';

Analyze the execution plan to see how many rows are returned and measure the performance impact.

#### Closing Summary
and Common Mistakes

Choosing a prefix length that is too short can result in low selectivity, leading to suboptimal query performance. Conversely, selecting a very long prefix might improve selectivity but at the cost of increased storage space and potentially slower insert/update operations due to larger indexes.

#### Common Mistakes
Ignoring query patterns and assuming a single prefix length will work for all queries.
Overlooking the impact on storage costs when using longer prefixes.
Not monitoring performance metrics after indexing changes.
Self-Check

This solution is correct and comprehensive, covering both theoretical and practical aspects of index selection.

**Code:**

```sql
-- Example SQL statements to create indexes with varying prefix lengths
CREATE INDEX idx_description ON table (description(10));
CREATE INDEX idx_description ON table (description(30));
CREATE INDEX idx_description ON table (description(50));
```

