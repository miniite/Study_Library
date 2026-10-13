---
type: "practice"
difficulty: "Intermediate"
---

# Your client caches a paginated list and a user reports a duplicated row across pages during active writes. How does non-stable ordering plus OFFSET cause this, and how does keyset fix it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of duplicated rows in a paginated list when using OFFSET with active writes, we need to understand how database indexing, query optimization, and pagination mechanisms interact. Here’s a detailed explanation:

#### Overview
The problem arises because non-stable ordering during reads (due to missing indexes or lack of stable order guarantees) combined with the use of OFFSET can lead to duplicates when data is actively being written. Keyset pagination, on the other hand, resolves this by ensuring consistent and deterministic retrieval of rows.

#### Triage
The question is probing our understanding of database indexing, query optimization techniques, and how different pagination methods affect the integrity of paginated queries. We need to assume that the application uses a relational database system like MySQL or PostgreSQL for its operations. Additionally, we will consider that there are frequent writes happening to the table.

Step-by-Step Solution:

Understanding Non-Stable Ordering: In many cases, especially with non-primary key orders, the order of rows in a query can change between executions due to changes in the underlying data or index structure. This instability is particularly problematic when using OFFSET for pagination.

Using OFFSET for Pagination: When we use LIMIT and OFFSET together in SQL queries, we rely on the fact that the rows returned by the database are ordered consistently across queries. However, if there are frequent updates (writes) to the table, the order of the data can shift, leading to potential duplication issues.

#### Example Scenario
Consider a users table with an id column as the primary key and additional columns like name, email. If we use a query like:

SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

Without a stable order (like using the primary key), the result set can change if new rows are inserted between the first and second query execution.

Keyset Pagination: Keyset pagination addresses this issue by fetching data based on the last known value of the row, rather than relying on OFFSET. For example:

SQL
SELECT * FROM users WHERE id > 100 ORDER BY id LIMIT 10;

This approach ensures that the same rows are fetched even if new rows are inserted. The client application keeps track of the last seen id and uses it as a starting point for the next query.

#### Closing Summary
In summary, non-stable ordering plus OFFSET can lead to duplicate rows in paginated lists during active writes because the order of the data changes unpredictably. Keyset pagination avoids this issue by using a stable identifier (like a primary key) to fetch the correct set of rows consistently.

#### Common Mistakes
and Follow-Ups:

Mistake: Relying solely on OFFSET for pagination in scenarios with frequent updates.

#### Follow-Up
Discuss how database transactions, consistency levels, and eventual consistency models (e.g., in NoSQL databases) can also impact pagination mechanisms.

**Comparison:**

```
User ID	Name	Email
1	Alice	alice@example.com
2	Bob	bob@example.com
3	Charlie	charlie@example.com
```

**Code:**

```sql
-- Example of a stable ORDER BY using the primary key
SELECT * FROM users WHERE id > 100 ORDER BY id LIMIT 10;

-- Example of using OFFSET with potential duplication issues
SELECT * FROM users ORDER BY name LIMIT 10 OFFSET 20;
```

**Example input / output:**

```text
Input:
- Current state of the 'users' table

Output:
1. Using keyset pagination to fetch next page:
   - Last seen id: 300
   - Query: SELECT * FROM users WHERE id > 300 ORDER BY id LIMIT 10;
2. Using OFFSET with potential duplication issues:
   - Current query: SELECT * FROM users ORDER BY name LIMIT 10 OFFSET 20;
   - Expected result: Duplicates if new rows are inserted between the first and second execution.
```

**Data:**

```json
{
  "users": [
    { "id": 1, "name": "Alice", "email": "alice@example.com" },
    { "id": 2, "name": "Bob", "email": "bob@example.com" },
    { "id": 3, "name": "Charlie", "email": "charlie@example.com" },
    // ... more users
  ]
}
```

**Flow:**

1. Stable ORDER BY and KeysetPagination
2. Fetch Next Page Consistently
3. Non-Stable ORDER BY withOFFSET
4. Potential Duplication Issues

