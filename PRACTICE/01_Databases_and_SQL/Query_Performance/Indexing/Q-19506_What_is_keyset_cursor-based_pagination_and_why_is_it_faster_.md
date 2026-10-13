---
type: "practice"
difficulty: "Deep"
---

# What is keyset (cursor-based) pagination and why is it faster than OFFSET?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Keyset pagination, also known as cursor-based pagination, is a technique used in database querying where you fetch records based on their position relative to a previously fetched record rather than using an offset. This method can be significantly faster and more efficient compared to the traditional OFFSET approach.

#### Triage

The question is essentially asking about the concept of keyset pagination and its performance benefits over OFFSET-based pagination. The interviewer is probing your understanding of database indexing, query optimization, and how different pagination techniques impact query execution speed and resource usage. Key assumptions include familiarity with SQL queries and database operations.

#### First-Principles Solution

Firstly, let's understand the traditional OFFSET-based pagination. When using OFFSET, you fetch a certain number of records from the start and then offset by n rows to get the next set of records. This approach can become inefficient for large datasets because it requires scanning through the entire dataset up to the offset point each time.

Keyset pagination, on the other hand, uses a cursor or key value to fetch the next set of records. The cursor is typically the primary key or any unique identifier in your table that you use as a reference point. Here’s how it works:

Initial Fetch: You start with an initial query and retrieve a small number of records along with their keys.
Subsequent Queries: For subsequent pages, instead of using OFFSET, you pass the cursor (key) from the last record of the previous page to fetch the next set of records.
Detailed Explanation

To illustrate this, consider a simple scenario where you have a users table and want to paginate through users based on their ID. Here’s how keyset pagination would work:

Initial Query:

SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;

This query retrieves the first 10 records from the users table.

Cursor (Key) Extraction: From the result, you extract the ID of the last record returned in this initial fetch. For example, if the last user has an ID of 567, that becomes your cursor.

Subsequent Query:

SQL
SELECT * FROM users WHERE id > 567 ORDER BY id LIMIT 10;

In this query, you use the extracted cursor (ID 567) to fetch the next set of records where id is greater than 567.

Performance Analysis
OFFSET-Based Pagination:
Scalability Issues: For large datasets, each page request requires scanning from the beginning up to the offset point, which becomes increasingly slow.
Memory Usage: The database needs to hold a significant amount of data in memory for the scan, leading to potential performance bottlenecks.
Keyset-Based Pagination:
Efficiency: Only requires fetching records based on the cursor value, making it faster and more scalable.
Concurrency: Can handle concurrent requests without needing to lock large portions of the database table.

#### Cons
istency: Works well with distributed systems where consistent queries are required.

#### Common Mistakes
and Follow-Ups

Mistake 1: Assuming keyset pagination always performs better than OFFSET. While it is more scalable, there can be scenarios where OFFSET might still be faster for small datasets or simpler use cases.

#### Follow-Up
Discuss when OFFSET could be preferable, such as in simple systems with a manageable number of records.

Mistake 2: Ignoring the need for proper indexing on the column used as the key. If you don’t have an appropriate index, keyset pagination can degrade to full table scans.

#### Follow-Up
Explain how important it is to ensure that the indexed columns are well-chosen and discuss best practices for choosing indexes.

Mistake 3: Not considering edge cases where a user might start from the beginning or end of the dataset. In such cases, keyset pagination requires handling initial conditions differently.

#### Follow-Up
Describe how to handle edge cases like starting at the first record or navigating to the last page using keyset pagination.

#### Closing Summary

Keyset pagination is an efficient technique for implementing pagination in large databases by using a cursor-based approach. It provides better performance, scalability, and concurrency compared to OFFSET-based pagination. However, it requires proper indexing and careful handling of edge cases to fully leverage its benefits.

#### Common Mistakes/Follow-Ups

Always ensure that the indexed columns are well-chosen, consider edge cases, and be mindful of when OFFSET might still be a better choice for simpler use cases.

**Comparison:**

| User ID | Name     |
|---------|----------|
| 1       | Alice    |
| 2       | Bob      |
| 3       | Charlie  |
| ...     | ...      |
| 567     | Zoe      |
| 568     | Zoe      |
| 569     | Anna     |
| ...     | ...      |

**Code:**

```sql
-- Initial Query Example
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;

-- Subsequent Query Example Using Keyset Pagination
SELECT * FROM users WHERE id > 567 ORDER BY id LIMIT 10;
```

**Example input / output:**

```text
Initial Query Input:
- SQL: SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0

Output (first 10 records):
- [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}, ... , {"id": 10, "name": "John"}]

Cursor Extraction:
- Last ID: 567

Subsequent Query Input:
- SQL: SELECT * FROM users WHERE id > 567 ORDER BY id LIMIT 10

Output (next 10 records):
- [{"id": 568, "name": "Zoe"}, {"id": 569, "name": "Anna"}, ... , {"id": 577, "name": "Mark"}]
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
    {"id": 3, "name": "Charlie"},
    ...
    {"id": 567, "name": "Zoe"}
  ]
}
```

**Flow:**

1. Initial Query
2. (Select * from users ORDER BYid LIMIT 10 OFFSET 0)
3. (Fetch first 10 records)
4. (Extract last record ID ascursor)
5. (Cursor = 567)
6. Subsequent Query
7. (Select * from users WHERE id> 567 ORDER BY id LIMIT 10)
8. (Fetch next 10 records usingcursor)

