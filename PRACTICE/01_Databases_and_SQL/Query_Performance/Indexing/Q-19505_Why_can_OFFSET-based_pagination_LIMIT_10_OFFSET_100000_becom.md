---
type: "practice"
difficulty: "Deep"
---

# Why can OFFSET-based pagination (LIMIT 10 OFFSET 100000) become slow, and what is the fix?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

OFFSET-based pagination using LIMIT 10 OFFSET 100000 can become significantly slow due to its inefficiency in database operations. The core issue lies in how SQL engines handle such queries, which often require scanning through a large portion of the table before returning results. This approach is particularly problematic for large datasets or frequently accessed tables.

#### Triage
Understanding the Problem

The question is probing your understanding of:

Pagination Mechanisms: How different pagination techniques work.
Database Performance Considerations: Why certain queries can degrade performance and how to optimize them.
Query Optimization Techniques: What tools and strategies are available to improve query performance.

#### First-Principles Solution

The issue with OFFSET 100000 is that the database must first retrieve all rows up to the point of offset, which can be extremely resource-intensive. This process involves reading a large number of records from disk, potentially leading to increased I/O operations and memory usage. A more efficient approach would be to use a cursor-based or keyset pagination mechanism.

Step-by-Step Solution

Identify the Need for Pagination: Understand that for large datasets, you need to limit the amount of data returned in each request.

Choose an Efficient Pagination Method:

Cursor-Based Pagination: Use a unique identifier (such as an ID) from the last row of the previous page to fetch the next set of records.
Keyset-based Pagination: Similar to cursor-based, but typically more robust since it can handle deleted rows better.

Implement the Efficient Method:

SQL
-- Example with MySQL
SELECT * FROM users
WHERE id > 10000 AND id <= 20000
LIMIT 10;

Monitor and Optimize: Continuously monitor query performance and adjust the pagination logic as needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Assuming that OFFSET is a lightweight operation, leading to inefficient use of resources.

#### Follow-Up
Question: How would you implement cursor-based pagination in your application? What are the trade-offs?

**Comparison:**

```
ID	Name
9500	User A
9600	User B
...	...
10000	User Z
```

**Code:**

```sql
-- Example SQL query for keyset-based pagination
SELECT * FROM users
WHERE id > 10000 AND id <= 20000
LIMIT 10;
```

**Example input / output:**

```text
Input:
- Previous cursor ID = 9500
- Number of records to fetch = 10

Output:
- Next set of user IDs: [9600, 9700, ..., 9800]
```

**Data:**

```json
{
    "users": [
        {"id": 9500, "name": "User A"},
        {"id": 9600, "name": "User B"},
        // ... 9800 more users
        {"id": 10000, "name": "User Z"}
    ]
}
```

**Flow:**

1. Initiate Query
2. Check for Previous Cursor
3. Fetch Records with Keyset
4. Return Results
5. Next Page?
6. Request New Cursor
7. Repeat if Needed

