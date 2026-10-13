---
type: "practice"
difficulty: "Deep"
---

# How do you choose between server-side keyset streaming with a cursor and offset-limit pagination for an API that must scroll a huge feed reliably?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Bulk Collect and FORALL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API to fetch items from a large, potentially unbounded dataset like a feed, choosing between server-side keyset streaming with cursors and offset-limit pagination is crucial. The choice impacts performance, scalability, and user experience significantly. Here’s how we can approach this decision:

Summary: Keyset streaming offers better performance by leveraging primary keys or unique identifiers to fetch subsequent sets of data, reducing the need for server-side sorting or scanning. Offset-limit pagination, on the other hand, requires an initial query to find a starting point and then limits the number of records returned.

#### Triage
The question is essentially about choosing between two methods for fetching paginated results in a large dataset:

Offset-Limit Pagination: Uses an offset (starting position) and limit (number of items per page) to fetch data.
Keyset Streaming with Cursored Pagination: Utilizes unique keys from the last fetched row as cursors to fetch subsequent sets of data.

First-Principles Reasoning:

Offset-Limit Limitations:

Requires a full table scan or index scan if not using an appropriate index.
Performance degrades with larger datasets due to increased I/O and memory usage.
Difficult to implement efficiently in all cases, especially when dealing with non-sorted data.

Keyset Streaming Advantages:

More efficient as it leverages existing indexes on primary keys or unique columns.
Better for large datasets as it minimizes the number of rows fetched and scanned by the database engine.
Easier to implement when fetching sorted data, as sorting is typically already done.

#### Solution
The choice depends on several factors:

Data Size and Distribution:

For very large datasets with a well-distributed primary key, cursor-based pagination is more efficient.
For smaller or less frequently updated datasets, offset-limit might be simpler to implement initially but can still work.

Query Performance:

Use indexes effectively on the columns used for cursors (primary keys) to minimize I/O and speed up queries.

User Experience:

Cursor-based pagination provides a smoother user experience as it fetches only what is needed, reducing load times.
Offset-limit can lead to jerky performance if not optimized properly.

#### Closing Summary
In summary, keyset streaming with cursors is generally more efficient and scalable for fetching paginated results from large datasets, especially when dealing with sorted data or well-distributed primary keys. However, the choice should also consider the specific requirements of your application, such as data size, query performance, and user experience.

#### Common Mistakes/Follow-Ups

Failing to use indexes effectively on columns used for cursors can lead to suboptimal performance.
Not considering edge cases where the cursor value might not be unique or might change unexpectedly.
Ignoring potential limitations of offset-limit pagination in terms of performance and implementation complexity.

**Comparison:**

| Column | Data Type | Nullable? |
|--------|-----------|----------|
| id     | int       | NO       |
| content| text      | YES      |

**Code:**

```sql
-- Example SQL query using a cursor-based approach
WITH cursor_data AS (
  SELECT * FROM large_table WHERE id > :last_cursor_value ORDER BY id LIMIT 10
)
SELECT * FROM cursor_data;
```

**Example input / output:**

```
Input:
Request from the client with last_cursor_value = 4234567.
Output:
JSON
Copy
[
  {"id": 423458, "content": "Content for row 1"},
  {"id": 423459, "content": "Content for row 2"},
  ...
  {"id": 423467, "content": "Content for row 10"}
]
```

**Data:**

```
large_table: A table with primary key id.
JSON
Copy
{
  "large_table": {
    "columns": ["id", "content"],
    "primary_key": "id",
    "data_size": "10,000,000 rows"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is dataset large andwell-distributed?
5. Cursors
6. Offset-Limit Pagination
7. Implement cursor-basedpagination
8. Implement offset-limitpagination

