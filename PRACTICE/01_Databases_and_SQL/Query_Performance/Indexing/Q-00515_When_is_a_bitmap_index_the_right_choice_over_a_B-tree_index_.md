---
type: "practice"
difficulty: "Deep"
---

# When is a bitmap index the right choice over a B-tree index and why does column cardinality drive that decision?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary Overview:

Bitmap indexes are a specialized type of database index used to speed up query performance, particularly in scenarios with low cardinality columns. In this context, we will explore when a bitmap index is more appropriate than a B-tree index and how column cardinality influences that decision.

#### Triage
Understanding the Question

The question asks for the conditions under which a bitmap index would be preferred over a B-tree index, with a specific focus on the role of column cardinality. The interviewer likely seeks an in-depth understanding of both indexing techniques and their respective trade-offs, particularly in terms of performance optimization.

Key Assumptions:
We are working within a relational database system that supports bitmap indexes.
Column cardinality refers to the number of distinct values a column can take.

#### First-Principles Solution
1. Understanding B-tree Indexes

B-tree indexes store data in a balanced tree structure, making them efficient for range queries and joins. They are well-suited for columns with high cardinality (many unique values) because they provide fast access to individual rows based on key values.

2. Introduction to Bitmap Indexes

Bitmap indexes represent the presence or absence of row identifiers using bits in a bitmap array. Each bit corresponds to a specific value in the indexed column, allowing efficient querying for multiple conditions and complex predicates.

3. When to Use Bitmap Indexes:
Low Cardinality Columns: Bitmap indexes are highly effective when a column has a small number of distinct values. For example, columns like status, gender, or region often have limited sets of possible values.
Highly Selective Queries: When queries involve complex conditions that can be evaluated using bitmasks (e.g., multiple ANDs and ORs), bitmap indexes excel because they directly manipulate bits to return the relevant rows.
4. Performance Considerations:
Space Efficiency: Bitmap indexes are space-efficient, especially for low-cardinality columns where the number of distinct values is small.
Query Optimization: Bitmap indexes allow efficient evaluation of complex predicates and can be used in conjunction with B-tree indexes to further optimize query performance.
Insert/Update Overhead: Insertions and updates to bitmap indexes require rebuilding the entire index, which can be resource-intensive. However, this overhead is typically acceptable for low-cardinality columns where the number of changes is limited.
5. Column Cardinality as a Driver:

Column cardinality drives the decision because:

High Cardinality Columns: B-tree indexes are generally more appropriate due to their superior performance in handling large datasets and complex queries.
Low Cardinality Columns: Bitmap indexes provide significant benefits, especially when dealing with multiple conditions that can be efficiently evaluated using bitmasks.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Closing Summary
Bitmap indexes are ideal for columns with low cardinality where they offer space efficiency and performance advantages in evaluating complex queries. However, the decision should be based on a thorough understanding of the specific use case and workload characteristics.

#### Common Mistakes/Follow-Ups

Overusing Bitmap Indexes: Using bitmap indexes excessively can lead to increased storage costs without significant benefits.
Ignoring Workload Characteristics: Failing to consider the nature of the queries that will be run against the database can result in suboptimal performance.

In summary, bitmap indexes are a valuable tool but should be carefully considered based on column cardinality and query patterns. A deep understanding of both B-tree and bitmap indexing is crucial for making informed decisions about database design.

**Comparison:**

```
Column Name	Cardinality	Values
status	4	['active', 'inactive', 'suspended']
region	10	['north', 'south', 'east', 'west']
age	200	[18, 35, 60]
```

**Code:**

```sql
-- Example SQL to create a bitmap index
CREATE INDEX idx_gender ON users (gender);
```

**Example input / output:**

```text
Input Query:
SELECT * FROM users WHERE status = 'active' AND region IN ('north', 'south');

Output Execution Plan (Assuming Bitmap Indexes):
- Use bitmap index on `status` to narrow down rows.
- Use bitmap index on `region` to further filter the results using bitmasks.
```

**Data:**

```json
{
  "columns": [
    {"name": "status", "cardinality": 4, "values": ["active", "inactive", "suspended"]},
    {"name": "region", "cardinality": 10, "values": ["north", "south", "east", "west"]},
    {"name": "age", "cardinality": 200, "values": [18, 35, 60]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query arrives
4. Is the column low cardinality?
5. Use bitmap index
6. Use B-tree index
7. Index lookup and evaluationusing bitmasks
8. Standard B-tree operations forrange queries

