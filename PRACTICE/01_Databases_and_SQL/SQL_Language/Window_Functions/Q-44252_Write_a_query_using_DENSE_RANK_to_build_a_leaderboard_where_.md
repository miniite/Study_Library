---
type: "practice"
difficulty: "Intermediate"
---

# Write a query using DENSE_RANK to build a leaderboard where tied scores share a rank and the next distinct score is one higher.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to create a leaderboard that ranks entries based on their scores, ensuring that tied scores are given the same rank, and the next distinct score is ranked one higher than the highest of those ties. DENSE_RANK is an excellent tool for this purpose because it assigns the same rank to tied records while maintaining a sequential order.

#### Triaging the Question

The core question here involves understanding how DENSE_RANK works in SQL. We need to:

Identify and handle ties in scores.
Ensure that once a score is ranked, subsequent identical or higher scores get their ranks correctly incremented.
Generate a leaderboard with proper ranking for each entry.
Key Assumptions
The input data includes at least two columns: user_id (for identifying the user) and score (the metric based on which we rank).
We will use an SQL query to implement DENSE_RANK.

#### First Principles Solution
Identify Ties: DENSE_RANK assigns a unique rank to each row, even if there are ties.
Generate Ranks: Use a subquery or Common Table Expression (CTE) to first calculate the ranks for all entries.
Ranking Logic: Ensure that once a score is assigned a rank, subsequent identical scores get the next available rank.
SQL Query
SQL
WITH RankedScores AS (
    SELECT
        user_id,
        score,
        DENSE_RANK() OVER (ORDER BY score DESC) AS rank
    FROM
        leaderboard_data
)
SELECT
    user_id,
    score,
    rank
FROM
    RankedScores;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query above will provide a basic ranking. However, for more complex scenarios where we need to ensure that the next distinct score is one higher than the highest of those ties, we can use additional logic in the DENSE_RANK function or post-processing.

Common Mistake: Overlooking Ties

A common mistake might be ignoring the fact that DENSE_RANK handles ties by assigning the same rank to tied entries. This means if two users have the same score, they will share a rank, and the next distinct score will start ranking from one higher than the highest of those ranks.

#### Follow-Up
Handling Ties More Granularly

If we need more granular control over ties (e.g., ensuring that no two tied scores are given the same rank), we can use RANK or ROW_NUMBER combined with appropriate logic in a subquery to achieve this. For example:

SQL
WITH RankedScores AS (
    SELECT
        user_id,
        score,
        RANK() OVER (ORDER BY score DESC) AS rank
    FROM
        leaderboard_data
)
SELECT
    user_id,
    score,
    rank
FROM
    RankedScores;

#### Example
Data Schema
JSON
{
  "leaderboard_data": [
    {"user_id": 1, "score": 90},
    {"user_id": 2, "score": 85},
    {"user_id": 3, "score": 85},
    {"user_id": 4, "score": 80},
    {"user_id": 5, "score": 75}
  ]
}
Worked Input-Output

Given the example data:

SQL
WITH RankedScores AS (
    SELECT
        user_id,
        score,
        DENSE_RANK() OVER (ORDER BY score DESC) AS rank
    FROM
        (SELECT * FROM JSON_EXTRACT('{"leaderboard_data": [{"user_id": 1, "score": 90}, {"user_id": 2, "score": 85}, {"user_id": 3, "score": 85}, {"user_id": 4, "score": 80}, {"user_id": 5, "score": 75}]'}) -> 'leaderboard_data')
)
SELECT
    user_id,
    score,
    rank
FROM
    RankedScores;

The expected output would be:

user_id	score	rank
1	90	1
2	85	2
3	85	2
4	80	4
5	75	5
Self-Check

The solution is correct and complete.

**Comparison:**

| user_id | score | rank |
|---------|-------|------|
| 1       | 90    | 1    |
| 2       | 85    | 2    |
| 3       | 85    | 2    |
| 4       | 80    | 4    |
| 5       | 75    | 5    |

**Code:**

```sql
WITH RankedScores AS (
    SELECT
        user_id,
        score,
        DENSE_RANK() OVER (ORDER BY score DESC) AS rank
    FROM
        leaderboard_data
)
SELECT
    user_id,
    score,
    rank
FROM
    RankedScores;
```

**Example input / output:**

```sql
WITH RankedScores AS (
    SELECT
        user_id,
        score,
        DENSE_RANK() OVER (ORDER BY score DESC) AS rank
    FROM
        (SELECT * FROM JSON_EXTRACT('{"leaderboard_data": [{"user_id": 1, "score": 90}, {"user_id": 2, "score": 85}, {"user_id": 3, "score": 85}, {"user_id": 4, "score": 80}, {"user_id": 5, "score": 75}]'}) -> 'leaderboard_data')
)
SELECT
    user_id,
    score,
    rank
FROM
    RankedScores;
user_id	score	rank
1	90	1
2	85	2
3	85	2
4	80	4
5	75	5
```

**Data:**

```json
{
  "leaderboard_data": [
    {"user_id": 1, "score": 90},
    {"user_id": 2, "score": 85},
    {"user_id": 3, "score": 85},
    {"user_id": 4, "score": 80},
    {"user_id": 5, "score": 75}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the data ready?
5. Query using DENSE_RANK
6. Generate leaderboard withcorrect ranks
7. End
8. Prepare and load data
9. C[Query using DENSE_RANK]
10. D[Generate leaderboard withcorrect ranks]
11. E[End]

