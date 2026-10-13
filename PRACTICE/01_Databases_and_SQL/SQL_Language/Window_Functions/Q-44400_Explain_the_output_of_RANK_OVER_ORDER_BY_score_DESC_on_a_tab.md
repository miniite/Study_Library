---
type: "practice"
difficulty: "Basic"
---

# Explain the output of RANK() OVER (ORDER BY score DESC) on a table with tied scores.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with window functions in SQL, RANK() is used to assign a unique rank to rows within a result set that have the same values in the specified order. In this scenario, we are ranking the scores in descending order and handling ties by assigning each tied score the same rank.

To illustrate the output of RANK() OVER (ORDER BY score DESC) on a table with tied scores, let's consider an example.

**Code:**

```sql
CREATE TABLE scores (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    score INT
);

INSERT INTO scores (id, name, score) VALUES
(1, 'Alice', 90),
(2, 'Bob', 85),
(3, 'Charlie', 85),
(4, 'David', 75),
(5, 'Eve', 90);
```

**Example input / output:**

```

To see the output of RANK() OVER (ORDER BY score DESC), we can run the following SQL query:

SQL
Copy
SELECT id, name, score,
       RANK() OVER (ORDER BY score DESC) AS rank_score
FROM scores;

This would yield the following result set:

id	name	score	rank_score
5	Eve	90	1
1	Alice	90	1
2	Bob	85	2
3	Charlie	85	2
4	David	75	3

In this output, we see that both Alice and Eve have a rank of 1 because their scores are the highest (90). Similarly, Bob and Charlie share the second rank due to having the same score (85), while David has the third rank.
```

