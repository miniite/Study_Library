---
type: "practice"
difficulty: "Deep"
---

# You store product attributes in jsonb and need to query by a specific key frequently; would you build an expression index on (data->>'brand') or a GIN index, and how does the query shape decide which the planner picks?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of whether to use an expression index on (data->>'brand') or a GIN index when querying by brand in PostgreSQL, we need to understand the differences between these indexing strategies and how the query planner makes decisions.

#### Triaging the Question

Expression Index (e.g., B-tree index):

An expression index is created on an SQL expression that involves columns from a table.
It can be useful when you frequently filter or sort by complex expressions, as it allows the database to quickly access rows based on these expressions.

GIN Index:

GIN (Generalized Inverted Index) is used for full-text search and JSONB data types.
It excels at handling queries that involve multiple values in a single column or when you need to query by any part of the JSON document.

Query Shape and Query Planner:

The query planner in PostgreSQL evaluates various possible plans and chooses the most efficient one based on estimated costs, statistics, and other factors.
The query shape (e.g., SELECT * FROM products WHERE data->>'brand' = 'Samsung';) is a critical factor in this decision.

#### First-Principles Solution

Expression Index:

An expression index can be created on (data->>'brand') using the following command:
SQL
CREATE INDEX idx_brand ON products ((data->>'brand'));
This index will help speed up queries that filter by brand, as it directly indexes the result of the JSONB path expression.

GIN Index:

A GIN index can be created on the entire JSONB column:
SQL
CREATE INDEX idx_gin_brand ON products USING GIN (data);
This allows more flexible queries, including partial matches and wildcards, but might not be as efficient for simple equality checks.

Query Shape:

For a query like SELECT * FROM products WHERE data->>'brand' = 'Samsung';, the expression index on (data->>'brand') is likely to be chosen by the planner due to its direct applicability.
However, if the query involves more complex filtering or sorting, a GIN index might provide better overall performance.

Query Planner Decisions:

The planner considers multiple factors such as data distribution, available indexes, and estimated costs of different execution plans.
For simple equality checks, an expression index is often preferred because it directly targets the required result without scanning more than necessary rows.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Expression Index: Best for simple queries where you filter by a specific key value.
GIN Index: Better suited for complex queries involving multiple values or partial matches within JSONB documents.
Planner Considerations: The planner will pick the index that minimizes costs, which can vary based on query shape and statistics.

#### Common Mistakes
Overusing GIN Indexes: Using a GIN index when a simple expression index would suffice for common queries.
Ignoring Query Shape: Not considering how different parts of the query impact indexing decisions.
Neglecting Statistics: Failing to update or consider statistics that can significantly affect plan choices.

#### Follow-Ups
Multi-Column Indexes:
If you frequently combine brand with other attributes in queries, a multi-column index might be beneficial:
SQL
CREATE INDEX idx_brand_other ON products ((data->>'brand'), (data->>'color'));
Index Maintenance: Regularly update statistics and consider reindexing if data distribution changes significantly.
Alternative Approaches:
For very complex queries, consider denormalizing the schema or using a different database system better suited for JSONB.

