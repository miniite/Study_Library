---
type: "practice"
difficulty: "Deep"
---

# You own a table where 30% of queries filter on a JSONB path and 70% on scalar columns. How do you avoid a heavy GIN index dominating your write budget?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ JSON and Text Search Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of balancing query performance with write efficiency in a database where 30% of queries filter on JSONB paths and 70% on scalar columns, we need to optimize indexing strategies without overwhelming our write budget. The key is to carefully design indexes that cater to both types of queries while minimizing index maintenance costs.

Firstly, let's clarify what the question is probing:

Indexing Strategy: We need a balanced approach where read performance is optimized for both JSONB and scalar column filters.
Write Efficiency: Ensuring that the write operations on our database are cost-effective by avoiding excessive overhead from too many or poorly designed indexes.

We will reason step-by-step through first principles, considering different index types and their implications.

Step 1: Understanding Index Types

Database indexing comes in various flavors. For this problem, we need to consider:

B-Tree Indexes: Suitable for scalar columns.
GIN (Generalized Inverted Index): Ideal for JSONB path queries due to its flexibility and ability to support complex query conditions.
Step 2: Designing the Schema

Assume our table user_data has a structure like this:

SQL
CREATE TABLE user_data (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    metadata JSONB NOT NULL, -- Contains nested data
    age INT NOT NULL,
    city TEXT NOT NULL
);
Step 3: Indexing Strategy

To avoid the heavy GIN index dominating our write budget:

B-Tree Index for Scalar Columns: Create a B-Tree index on columns that are frequently used in scalar queries.
GIN Index for JSONB Path Queries: Create a GIN index specifically to handle complex JSONB path filtering.
SQL
CREATE INDEX idx_name ON user_data (name);
CREATE INDEX idx_metadata_path ON user_data USING GIN ((metadata->>'path_to_key'));
Step 4: Analyzing Write Costs
B-Tree Index Writes: These are relatively cheaper as they involve updating a single tree structure.
GIN Index Writes: These can be more expensive due to the need for additional nodes and potential cascading updates.

To mitigate write costs, consider:

Selective GIN Indexes: Create GIN indexes only on the most frequently queried JSONB paths.
Index Fragmentation Management: Regularly analyze index usage and drop/recreate less effective indexes as needed.
Step 5: Monitoring and Optimization

Regular monitoring of query performance and write costs will help in fine-tuning our indexing strategy:

Use EXPLAIN to understand the execution plans of your queries.
Analyze pg_stat_user_indexes for insights into index usage and maintenance cost.

#### Closing Summary

By carefully balancing B-Tree and GIN indexes, we can optimize query performance without significantly impacting write efficiency. Regular monitoring and adaptive indexing strategies ensure that our database remains both performant and cost-effective.

#### Common Mistakes/Follow-Ups

Over-indexing: Creating too many indexes, especially GIN, which can lead to high maintenance costs.
Under-utilization of Indexes: Not leveraging existing index types effectively, leading to suboptimal query performance.
Neglecting Write Costs: Focusing only on read optimization without considering the write overhead.

**Comparison:**

```
Index Type	Columns	Use Case
B-Tree	name	Scalar queries on name column.
GIN	(metadata->>'path_to_key')	Complex JSONB path filtering.
```

**Code:**

```sql
CREATE INDEX idx_name ON user_data (name);
CREATE INDEX idx_metadata_path ON user_data USING GIN ((metadata->>'path_to_key'));
```

