---
type: "practice"
difficulty: "Deep"
---

# For keyset pagination on (created_at, id), what exact index and WHERE clause do you need for stable ordering?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address keyset pagination using (created_at, id) as a composite primary key, we need to carefully design our database schema and query optimization strategy to ensure both performance and stability of the ordered results. Let's break down the solution from first principles.

#### Triage
Understanding the Problem

The goal is to implement stable ordering in keyset pagination using a composite index on (created_at, id). The key aspect here is ensuring that our query can reliably fetch subsequent pages based on the last record of the previous page. We need to ensure that any changes in id for records with the same created_at do not affect the pagination order.

#### Assumptions
We are using a relational database system, such as PostgreSQL or MySQL.
The table has columns: id, created_at, and other relevant fields.
We have a primary key on (created_at, id) to ensure uniqueness based on both columns.
The last record from the previous page is used as the starting point for fetching the next page.

#### First-Principles Solution
Index Design: Create an index on (created_at DESC, id). This ensures that records are ordered first by created_at in descending order and then by id within each day.
Query Construction:
Use a WHERE clause to filter based on the last record's created_at and id.
Include a LIMIT to fetch only the desired number of rows for the next page.
Detailed Steps

Create the Index

SQL
CREATE INDEX idx_created_at_id ON your_table (created_at DESC, id);

Query Construction Assume the last record from the previous page had created_at value '2023-10-01 14:30:00' and id of 123.

SQL
SELECT *
FROM your_table
WHERE (created_at, id) > ('2023-10-01 14:30:00', 123)
  AND created_at <= '2023-10-01'
ORDER BY created_at DESC, id ASC
LIMIT 10;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, for keyset pagination using (created_at, id), we need to:

Create an index on (created_at DESC, id) to ensure stable ordering.
Use a WHERE clause with the last record's created_at and id values to fetch subsequent pages.

#### Common Mistakes

Forgetting Descending Order: Ensure the index includes descending order for created_at.
Incorrect Filtering: Using only one column in the WHERE clause can lead to incorrect pagination.
Lack of Limitation: Not applying a limit to the query results may cause performance issues.
Additional Considerations
Partitioning: If the table is very large, consider partitioning by created_at to improve query performance.
Update Operations: Be cautious with updates on records within the same day as they might affect pagination stability.

**Code:**

```sql
-- Create index
CREATE INDEX idx_created_at_id ON your_table (created_at DESC, id);
```

**Example input / output:**

```text
SELECT *
FROM your_table
WHERE (created_at, id) > ('2023-10-01 14:30:00', 123)
AND created_at <= '2023-10-01'
ORDER BY created_at DESC, id ASC
LIMIT 10;
```

**Flow:**

1. Query Construction
2. Index on (created_at DESC, id)
3. Index Creation
4. WHERE clause for last record'screated_at and id
5. ORDER BY to ensure stablepagination
6. LIMIT 10 to fetch next page

