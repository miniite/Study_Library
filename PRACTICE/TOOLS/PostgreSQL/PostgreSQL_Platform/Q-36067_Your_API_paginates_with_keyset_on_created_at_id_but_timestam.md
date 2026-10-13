---
type: "practice"
difficulty: "Deep"
---

# Your API paginates with keyset on (created_at, id) but timestamps have duplicates and sometimes rows are skipped or repeated; walk me through the exact tuple-comparison predicate that makes keyset correct with ties.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure accurate keyset pagination in PostgreSQL when dealing with duplicate timestamps, it's crucial to understand how the database handles tuple comparisons and sorting. The key lies in crafting a predicate that correctly identifies rows based on both created_at and id, ensuring no duplicates or skips.

#### Triage

The question is about creating a predicate for keyset pagination using (created_at, id) where timestamps can be duplicated, leading to potential issues like row duplication or skipping. The goal is to understand how PostgreSQL handles such cases and ensure the pagination logic correctly identifies unique rows.

#### First-Principles Solution

PostgreSQL's ORDER BY clause uses a total order comparison, which means it compares tuples based on all columns in the order specified. When dealing with duplicates, we need to ensure that our predicate correctly identifies unique keys by leveraging PostgreSQL's handling of ties and ordering.

Step-by-Step Explanation

Understanding Tuple Comparison:

ORDER BY uses a total order comparison, where each column is compared sequentially.
If the first column matches (e.g., created_at), it moves to the next column (id) for comparison.
This ensures that even if timestamps are the same, rows with different ids will be treated as distinct.

Crafting the Predicate:

We need to ensure that our predicate is based on a total order of (created_at, id).
The predicate should leverage PostgreSQL's ability to handle ties and order correctly.

Implementing Keyset Pagination with Ties:

To handle ties effectively, we can use the ROW_NUMBER() window function or an equivalent mechanism.
Ensure that the pagination logic respects the total order of (created_at, id).
Full Solution

Here's a detailed solution using PostgreSQL to ensure accurate keyset pagination:

SQL
-- Sample query for keyset pagination with ties handling

WITH paginated_data AS (
  SELECT
    id,
    created_at,
    ROW_NUMBER() OVER (ORDER BY created_at DESC NULLS LAST, id DESC) as row_num
  FROM
    your_table
)

SELECT
  id,
  created_at
FROM
  paginated_data
WHERE
  row_num > :last_row_num AND -- Replace with the last row number from previous page
  (created_at = :next_created_at OR (created_at < :next_created_at AND id > :next_id)) -- Replace placeholders with actual values
ORDER BY
  created_at DESC NULLS LAST, id DESC
LIMIT :limit; -- Number of rows to fetch in this page
Explanation:
The ROW_NUMBER() function assigns a unique row number based on the total order of (created_at, id).
In the WHERE clause, we ensure that only rows with a higher row_num than the last row from the previous page are selected.
The predicate (created_at = :next_created_at OR (created_at < :next_created_at AND id > :next_id)) ensures that if timestamps match, the rows with larger id values are considered.

#### Closing Summary

By leveraging PostgreSQL's tuple comparison and using window functions like ROW_NUMBER(), we can ensure accurate keyset pagination even when dealing with duplicate timestamps. The predicate (created_at = :next_created_at OR (created_at < :next_created_at AND id > :next_id)) ensures that rows are correctly identified, preventing duplicates or skips.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to account for ties in ORDER BY, leading to incorrect row ordering.

#### Follow-Up
Discussing the impact of NULL values in created_at and how they should be handled (e.g., using NULLS LAST).
Advanced Discussion: Exploring other window functions like RANK() or DENSE_RANK() and their implications for pagination.

**Comparison:**

```
id	created_at
102	2023-10-05 14:00:01
103	2023-10-05 14:00:02
104	2023-10-05 14:00:03
...	...
```

**Code:**

```sql
WITH paginated_data AS (
  SELECT
    id,
    created_at,
    ROW_NUMBER() OVER (ORDER BY created_at DESC NULLS LAST, id DESC) as row_num
  FROM
    your_table
)

SELECT
  id,
  created_at
FROM
  paginated_data
WHERE
  row_num > :last_row_num AND -- Replace with the last row number from previous page
  (created_at = :next_created_at OR (created_at < :next_created_at AND id > :next_id)) -- Replace placeholders with actual values
ORDER BY
  created_at DESC NULLS LAST, id DESC
LIMIT :limit; -- Number of rows to fetch in this page
```

**Example input / output:**

```text
WITH paginated_data AS (
  SELECT
    id,
    created_at,
    ROW_NUMBER() OVER (ORDER BY created_at DESC NULLS LAST, id DESC) as row_num
  FROM
    your_table
)

SELECT
  id,
  created_at
FROM
  paginated_data
WHERE
  row_num > 10 AND -- Replace with the last row number from previous page
  (created_at = '2023-10-05 14:00:00' OR (created_at < '2023-10-05 14:00:00' AND id > 1005)) -- Replace placeholders with actual values
ORDER BY
  created_at DESC NULLS LAST, id DESC
LIMIT 10; -- Number of rows to fetch in this page
```

**Data:**

```json
{
  "last_row_num": 10,
  "next_created_at": "2023-10-05 14:00:00",
  "next_id": 1005,
  "limit": 10
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is timestamp a tie?
5. Return next row based on id
6. Compare ids, return higherone if equal timestamps
7. End of pagination logic

