---
type: "practice"
difficulty: "Deep"
---

# What is the difference between JSON and JSONB and which is faster to query?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore the differences between PostgreSQL's JSON and JSONB data types, focusing on their storage formats, querying capabilities, and performance trade-offs. By understanding these aspects, you can make informed decisions based on your application's requirements.

#### Triaging the Question

The question is asking about the fundamental differences between JSON and JSONB, as well as which one performs better for queries. The key points to consider are:

Storage Format: How data is stored in each type.
Query Capabilities: Which operations can be performed on each type more efficiently.

#### Performance Considerations
The impact of these differences on query performance.

#### First-Principles Solution
1. Storage Formats
JSON: Stores the data as a binary string, similar to a text file. This means that PostgreSQL does not parse or validate the JSON content at storage time.
JSONB: Stores the data in a more efficient format called "binary JSON," which allows for faster parsing and querying.
2. Query Capabilities
JSON: Does not support indexing on its fields, making complex queries slower since it requires scanning all rows to find matches.
JSONB: Supports indexing on specific fields, allowing for faster query execution because PostgreSQL can use indexes during the search process.
3. Performance Considerations
JSON: Generally slower due to the need to parse and validate each JSON string at runtime.
JSONB: Faster for querying operations that benefit from indexed fields but may be slightly larger in storage size due to the additional overhead of indexing.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, if you expect frequent queries on specific fields within your JSON data, JSONB is likely a better choice due to its support for indexing. However, if minimizing storage space and handling occasional ad-hoc queries are more critical, using JSON might be preferable.

#### Common mistakes
include:

Overuse of JSON: Storing large amounts of unstructured data as JSON without considering the performance implications.
Neglecting Indexes: Not utilizing the indexing capabilities of JSONB, leading to slower query performance.

#### Follow-up
questions could involve discussing specific use cases, trade-offs between storage and querying speed, or strategies for optimizing JSON-based queries in PostgreSQL.

**Comparison:**

| Column   | JSON        | JSONB          |
|----------|-------------|----------------|
| Storage  | Binary      | Efficient binary |
| Querying | Slower (no indexing) | Faster with indexes |
| Indexing | No         | Yes            |

**Code:**

```sql
-- Example of creating a table with both JSON and JSONB columns
CREATE TABLE example_table (
    id SERIAL PRIMARY KEY,
    json_column JSON,
    jsonb_column JSONB
);

-- Inserting sample data
INSERT INTO example_table (json_column, jsonb_column) VALUES ('{"key": "value"}', '{"key": "value"}');
```

**Example input / output:**

```text
-- Querying with JSONB for indexed fields
SELECT * FROM example_table WHERE jsonb_column->>'name' = 'Alice';

-- Output:
+----+-----------------+-----------------+
| id | json_column     | jsonb_column    |
+----+-----------------+-----------------+
| 1  | {"name": "Alice"}| {"name": "Bob"}  |
+----+-----------------+-----------------+
```

**Data:**

```json
[
    {
        "id": 1,
        "json_column": {"name": "Alice", "age": 30},
        "jsonb_column": {"name": "Bob", "age": 25}
    }
]
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Data Type
6. Is JSON?
7. Use JSON for ad-hoc queries
8. Do you need indexed fields?
9. Use JSONB for faster parsingand storage efficiency
10. Use JSONB with indexes foroptimized querying

