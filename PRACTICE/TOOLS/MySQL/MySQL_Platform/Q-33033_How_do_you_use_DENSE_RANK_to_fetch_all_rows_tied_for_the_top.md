---
type: "practice"
difficulty: "Deep"
---

# How do you use DENSE_RANK to fetch all rows tied for the top value, unlike LIMIT 1?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how to use DENSE_RANK in MySQL to fetch all rows that share the same highest value, let's first summarize the core idea and then dive into a step-by-step explanation from first principles.

TL;DR

The key is leveraging the DENSE_RANK() function along with appropriate window functions to identify and return all records tied for the top rank. This approach goes beyond just using LIMIT 1 by ensuring that multiple rows can be returned if they share the highest value.

#### Triaging the Question

The interviewer might be probing your understanding of:

Window functions in SQL.
Rank aggregation functions like DENSE_RANK().
How to manipulate and query data efficiently with MySQL.

#### Assumptions
We are working with a table that has numeric or comparable values in a column (e.g., scores, prices).
The goal is to retrieve all rows that have the highest value in this column.

#### Solution

To fetch all rows tied for the top value using DENSE_RANK, follow these steps:

Ranking Rows: Use DENSE_RANK() to assign a rank to each row based on the values of the specified column.
Filter Top Rank: Filter out only those rows that have the highest rank.

Here’s an example query:

SQL
SELECT *
FROM (
    SELECT *, DENSE_RANK() OVER (ORDER BY score DESC) as rank
    FROM scores_table
) AS ranked_scores
WHERE rank = 1;

In this example, scores_table is your data source and score is the column whose top value you want to find. The subquery ranks all rows in descending order of score, then the outer query filters those with a rank of 1 (the highest).

Full Solution

Let's break down the solution further:

SQL
-- Step 1: Rank the rows based on the score column
WITH ranked_scores AS (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY score DESC) as rank
    FROM scores_table
)
-- Step 2: Filter out only those rows with a rank of 1
SELECT *
FROM ranked_scores
WHERE rank = 1;

This WITH statement creates an intermediate result set where each row is given a rank. The outer query then filters this result to return only the rows that have the highest rank (i.e., rank = 1).

#### Closing Summary
and Common Mistakes/Follow-Ups

Using DENSE_RANK() ensures that you get all rows tied for the top value, whereas using just LIMIT would only return one row. This is particularly useful when you need to see multiple records with the same maximum value.

#### Common mistakes

Not understanding how window functions work.
Misusing RANK() instead of DENSE_RANK(), which might skip rows tied for the highest rank.

For follow-ups, the interviewer might ask about performance considerations or alternative solutions using other MySQL features like Common Table Expressions (CTEs).

#### Conclusion

Understanding and effectively using advanced SQL constructs like DENSE_RANK is crucial for efficient data querying. It helps in scenarios where you need to return multiple rows tied for a top value.

**Code:**

```sql
WITH ranked_scores AS (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY score DESC) as rank
    FROM scores_table
)
SELECT *
FROM ranked_scores
WHERE rank = 1;
```

**Example input / output:**

```text
Input:
WITH ranked_scores AS (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY score DESC) as rank
    FROM scores_table
)
SELECT *
FROM ranked_scores
WHERE rank = 1;

Output:
+----+-------+
| id | score |
+----+-------+
|  1 |   95  |
|  4 |   95  |
+----+-------+
```

**Data:**

```json
{
  "scores_table": [
    {"id": 1, "score": 95},
    {"id": 2, "score": 85},
    {"id": 3, "score": 90},
    {"id": 4, "score": 95}
  ]
}
```

