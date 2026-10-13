---
type: "practice"
difficulty: "Deep"
---

# Your queries filter jsonb_col ->> 'status' constantly and it's slow. Compare a GIN index, an expression B-tree index on the extracted key, and promoting status to a real column for equality versus containment.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with frequent queries that filter on jsonb_col ->> 'status', we need to choose the most efficient indexing strategy to speed up these operations. Let's compare three approaches: a GIN index, an expression B-tree index on the extracted key, and promoting status as a real column for equality versus containment.

#### Triaging the Question

The core of this question is to evaluate different indexing strategies based on their performance characteristics when filtering on a JSONB column. The key points are:

GIN Index: Designed for efficient querying of JSON data.
Expression B-tree Index: Uses an expression to index the result of a function, such as extracting status from jsonb_col.
Promoting Status as Real Column: Storing status directly in a real column allows for faster equality checks.

#### Assumptions
and Reasoning
Assumption 1: The JSONB column contains a high volume of data, making query performance critical.
Assumption 2: The queries are primarily filtering on the status field within the JSONB column.
Assumption 3: The database is PostgreSQL, as GIN and B-tree indexes are commonly used in this context.

#### First-Principles Solution

GIN Index:

When to Use: GIN (Generalized Inverted Index) is ideal for JSONB columns because it can efficiently handle complex queries like containment.

#### Performance Considerations
GIN index creation and maintenance might be slower, but query performance can be significantly faster.

Expression B-tree Index:

When to Use: An expression B-tree index on jsonb_col ->> 'status' creates a B-tree index based on the result of the function applied to the JSONB column.

#### Performance Considerations
This approach can be more efficient in terms of query execution because it avoids parsing and searching the full JSON structure. However, it requires an additional function call during insertions.

Promoting Status as Real Column:

When to Use: If status is frequently queried and does not change often, promoting it to a real column can significantly speed up equality checks.

#### Performance Considerations
This approach reduces the complexity of queries but requires additional storage and might complicate data updates.
Detailed Analysis

GIN Index:

SQL
CREATE INDEX idx_jsonb_status_gin ON my_table USING GIN (jsonb_col ->> 'status');

Expression B-tree Index:

SQL
CREATE INDEX idx_jsonb_status_expression ON my_table ((jsonb_col ->> 'status')::text);

Promoting Status as Real Column:

SQL
ALTER TABLE my_table ADD COLUMN status TEXT;

UPDATE my_table SET status = jsonb_col ->> 'status';

CREATE INDEX idx_status ON my_table (status);

#### Closing Summary
and Common Mistakes
Recommendation: Use a GIN index for frequent containment queries, an expression B-tree index for efficient equality checks, or promote status to a real column if it is static.

#### Common Mistakes
Overlooking the overhead of maintaining complex indexes like GIN.
Failing to consider the trade-offs between query speed and insert/update performance when using expression-based indexes.
Not evaluating the impact on storage requirements with promoted columns.
Final Thoughts

The choice depends heavily on the specific use case. For high-frequency queries, an index that directly targets the status field will be more efficient. Always test different strategies in a staging environment before implementing them in production to ensure optimal performance.

