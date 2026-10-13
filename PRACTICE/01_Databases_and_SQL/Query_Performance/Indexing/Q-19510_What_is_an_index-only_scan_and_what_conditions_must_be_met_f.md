---
type: "practice"
difficulty: "Deep"
---

# What is an index-only scan and what conditions must be met for one?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An index-only scan is a query execution method in PostgreSQL where, if all necessary data can be retrieved from the index itself without needing to access the table rows, the database engine performs only an index scan and returns the required result set directly. This technique is highly efficient because it minimizes disk I/O operations by leveraging indexes.

To fully understand the conditions that must be met for an index-only scan, we need to break down the concept from first principles. An index-only scan can occur under specific circumstances, which are critical to optimizing database queries and reducing storage and performance overheads.

#### Triaging the Question
What is an Index-Only Scan? - We need to explain what this technique does.
Conditions for Index-Only Scans: The question asks us to list the conditions that must be satisfied for such a scan to take place.

#### Assumptions
and Context: Consider the context of PostgreSQL, as it is known for supporting index-only scans.

#### First-Principles Solution
Key Conditions for an Index-Only Scan:

Index Type Compatibility:

The query must select columns that are present in a B-tree or hash index.
A GiST or SP-GiST index can also support partial index-only scans if all necessary data is contained within the index.

All Necessary Data in Index:

All columns required by the query must be included in the index. If any column needed for the projection (SELECT list) isn’t in the index, an index-only scan cannot occur.
This means that no additional disk accesses are needed to fetch data from the table.

Filter Clause:

The WHERE clause of the query should filter only by columns present in the index. If a column used for filtering is not indexed, or if there’s a need to join with other tables, an index-only scan cannot be performed.
An index-only scan will fail if the database needs to access data outside the scope of the index.

Sort Order:

The query must respect the sort order defined by the index. If the query requires sorting that doesn’t align with the index’s order, an index-only scan might not be possible.
A covered index (where all necessary columns are in the index) often includes a B-tree structure for efficient retrieval.

Join Conditions:

The query should avoid joining tables outside the indexed data. If there is a need to join other tables, an index-only scan cannot be performed.
Joins involving non-indexed columns or additional table scans will prevent an index-only scan from occurring.

#### Example Scenario

#### Cons
ider a simple example where we have a users table and a B-tree index on the email column. A query like:

SQL
SELECT email FROM users WHERE email = 'user@example.com';

Would be eligible for an index-only scan because:

The required data (email) is in the index.
There’s no need to access additional columns from the table.

However, a query such as:

SQL
SELECT email, phone FROM users WHERE email = 'user@example.com';

Would not qualify for an index-only scan because it requires accessing the phone column which is not part of the indexed data.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Ignoring the requirement that all necessary columns must be present in the index, failing to consider the filtering conditions, or misunderstanding the impact of join operations.

#### Follow-Up
Questions: The interviewer might ask about performance implications, how to design indexes for better query optimization, or differences between different types of indexes.

By understanding these key points and their practical applications, you can leverage index-only scans effectively in your database queries. Always ensure that all required columns are part of the indexed data structure to maximize efficiency.

**Comparison:**

```
Column	Index Type	Eligibility for Index-Only Scan
email	B-tree	Yes
phone	Not in index	No
```

**Code:**

```sql
-- Example of a simple query eligible for an index-only scan
SELECT email FROM users WHERE email = 'user@example.com';

-- Example where an index-only scan cannot be performed due to additional columns
SELECT email, phone FROM users WHERE email = 'user@example.com';
```

**Example input / output:**

| Input Query | Index-Only Scan Eligible |
|-------------|-------------------------|
| `SELECT email FROM users WHERE email = 'user@example.com';` | Yes                     |
| `SELECT email, phone FROM users WHERE email = 'user@example.com';` | No                      |

**Data:**

```json
{
  "users": [
    { "id": 1, "email": "user1@example.com", "phone": "1234567890" },
    { "id": 2, "email": "user2@example.com", "phone": "0987654321" }
  ]
}
```

**Flow:**

1. Yes
2. All Columns Present
3. Missing Columns
4. No
5. Query Analysis
6. Index Present?
7. Check Covered Index
8. Index-Only Scan Possible
9. Fetch Table Data
10. Not Eligible for Index-OnlyScan

