---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether a query filtering on a JSONB field should get an expression index on that key, a full GIN index, or a promoted real column?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ JSON and Text Search Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the best type of index for a query filtering on a JSONB field in PostgreSQL, we need to consider several factors related to the query's structure and the data distribution. The primary choices are an expression index, a full GIN (Generalized Inverted Index) index, or promoting the JSONB key to a real column. Here’s how to approach this decision:

#### Overview
Expression indices and GIN indexes are designed for specific types of queries on JSONB fields, while promoting keys as real columns can simplify query performance but requires additional storage.

#### Triage
The question is asking you to choose the most efficient indexing strategy based on the nature of the queries and data distribution. Assumptions include that we're working with PostgreSQL 13 or later due to GIN improvements, and the JSONB field contains a specific key used frequently in filtering.
Step-by-Step Solution
1. Analyze the Query

First, analyze the type of query:

SELECT WHERE clause: If your queries are based on conditions like jsonb->>'key' = 'value', consider GIN indexing.
Aggregation or Joins: If your queries involve complex operations such as filtering and aggregating data from nested JSON structures, GIN might be better.
2. Check Data Distribution

Next, check the distribution of values in the JSONB field:

Cardinality: If a key has few distinct values (low cardinality), an expression index or even a simple B-tree index on that column could suffice.
Frequency: If a key is used frequently and its values are highly selective, GIN can be more efficient.
3. Expression Index

An expression index allows you to create an index on the result of a function applied to a JSONB field:

SQL
CREATE INDEX idx_jsonb_key ON my_table ((jsonb->>'key'));

This is useful for simple conditions but doesn't handle complex queries involving multiple keys or nested structures efficiently.

4. GIN Index

A full GIN index can handle more complex queries and is well-suited for JSONB fields:

SQL
CREATE INDEX idx_jsonb_gin_key ON my_table USING gin (jsonb->>'key');

This index excels at handling full-text search, containment checks, and other complex operations on JSONB data.

5. Promoting the Key as a Real Column

Promoting a key to a real column can simplify queries but requires additional storage:

SQL
ALTER TABLE my_table ADD COLUMN key_col text;
UPDATE my_table SET key_col = jsonb->>'key';
CREATE INDEX idx_key_col ON my_table (key_col);

This approach is beneficial if the JSONB field becomes a primary focus of your queries and can significantly reduce query execution time.

#### Closing Summary

The choice between an expression index, GIN index, or promoting the key as a real column depends on the specific characteristics of your data and queries. An expression index is suitable for simple conditions, while a full GIN index handles complex operations efficiently. Promoting the key to a real column simplifies query structure but increases storage needs.

#### Common Mistakes
Overlooking the benefits of GIN indexes when dealing with nested JSON structures.
Failing to consider data cardinality and frequency in decision-making.
Ignoring the trade-offs between index size and query performance.

**Comparison:**

| Index Type        | Characteristics                                | Suitable For                      |
|------------------|------------------------------------------------|----------------------------------|
| Expression Index  | Simple conditions, e.g., `jsonb->>'key' = 'val'` | Fast for simple queries          |
| GIN Index         | Complex queries on JSONB fields                | Full-text search, containment    |
| Real Column       | Simplifies query structure                     | High data cardinality            |

**Code:**

```sql
-- Example of creating an expression index
CREATE INDEX idx_jsonb_key ON my_table ((jsonb->>'key'));

-- Creating a GIN index on JSONB field
CREATE INDEX idx_jsonb_gin_key ON my_table USING gin (jsonb->>'key');

-- Adding and populating a new column for real column promotion
ALTER TABLE my_table ADD COLUMN key_col text;
UPDATE my_table SET key_col = jsonb->>'key';
CREATE INDEX idx_key_col ON my_table (key_col);
```

**Example input / output:**

```text
Input Query:
SELECT * FROM my_table WHERE (data->>'key') = 'value1';

Output:
+----+--------------------------------+
| id | data                           |
+----+--------------------------------+
|  1 | {"key": "value1"}              |
|  3 | {"key": "value1"}              |
+----+--------------------------------+

Input Query with GIN Index:
SELECT * FROM my_table WHERE (data->>'key') = 'value1';

Output with Index:
+----+-------------------------+
| id | data                    |
+----+-------------------------+
|  1 | {"key": "value1"}       |
|  3 | {"key": "value1"}       |
+----+-------------------------+

Input Query with Promoted Real Column:
SELECT * FROM my_table WHERE key_col = 'value1';

Output with Index on Real Column:
+----+----------+--------------------------------+
| id | key_col  | data                           |
+----+----------+--------------------------------+
|  1 | value1   | {"key": "value1"}              |
|  3 | value1   | {"key": "value1"}              |
+----+----------+--------------------------------+
```

**Data:**

```json
{
  "table": {
    "name": "my_table",
    "columns": ["id", "data"],
    "data": [
      {"id": 1, "data": '{"key": "value1"}'},
      {"id": 2, "data": '{"key": "value2"}'},
      {"id": 3, "data": '{"key": "value1"}'}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Analyze Query
4. Is the key used in simpleconditions?
5. Create Expression Index
6. Check Data Cardinality andFrequency
7. Use GIN Index if cardinality islow and frequency high.
8. Promote Key as Real Column iffrequently used.

