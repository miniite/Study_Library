---
type: "practice"
difficulty: "Intermediate"
---

# Compare RANK, DENSE_RANK, and ROW_NUMBER for a leaderboard with ties. Which one do you choose when you must limit to exactly the top 10 and why?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Window Functions Overview](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are comparing three SQL ranking functions—RANK, DENSE_RANK, and ROW_NUMBER—to determine which is best suited for a leaderboard where ties must be handled but only the top 10 entries should be displayed. The key difference between these functions lies in how they handle tied ranks. Let's break down each function first, then discuss their implications for our specific requirement.

RANK, DENSE_RANK, and ROW_NUMBER are all used to assign a unique rank to rows within a result set based on some specified criteria. Here’s what they do:

RANK(): If two or more rows have the same ranking value, it will skip the subsequent rankings that would normally follow (e.g., 1st, 2nd, 2nd, 4th).
DENSE_RANK(): If two or more rows have the same ranking value, no rankings are skipped. The next rank is assigned immediately after the last tied row.
ROW_NUMBER(): Each row gets a unique number regardless of ties (e.g., 1st, 2nd, 3rd, 4th).

Given that we must limit the results to exactly the top 10 and handle ties correctly, let's examine each function:

RANK

#### Example
SQL
SELECT
    id,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM
    leaderboard;
Output:
CODE
|id|score|rank|
+--+-----+----+
|123|95   | 1  |
|456|87   | 2  |
|789|87   | 3  |
|012|76   | 4  |
|345|76   | 5  |
|678|65   | 7  |
|901|54   | 8  |
|123|54   | 9  |
|456|54   | 11 |
DENSE_RANK

#### Example
SQL
SELECT
    id,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM
    leaderboard;
Output:
CODE
|id|score|dense_rank|
+--+-----+----------+
|123|95   | 1        |
|456|87   | 2        |
|789|87   | 2        |
|012|76   | 4        |
|345|76   | 4        |
|678|65   | 6        |
|901|54   | 7        |
|123|54   | 8        |
|456|54   | 8        |
ROW_NUMBER

#### Example
SQL
SELECT
    id,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_number
FROM
    leaderboard;
Output:
CODE
|id|score|row_number|
+--+-----+----------+
|123|95   | 1        |
|456|87   | 2        |
|789|87   | 3        |
|012|76   | 4        |
|345|76   | 5        |
|678|65   | 6        |
|901|54   | 7        |
|123|54   | 8        |
|456|54   | 9        |
Selection

For our requirement of displaying exactly the top 10 rows with ties handled appropriately, DENSE_RANK is the best choice. Here’s why:

Handling Ties: Both RANK and DENSE_RANK handle ties, but only DENSE_RANK continues ranking from where it left off without skipping numbers.
Limiting to Top 10: When we apply a limit of 10 rows using DENSE_RANK, the tied scores will be grouped together. This ensures that if there are two or more rows with the same score, they all get the same rank, and only the first 10 such groups (including single-score entries) are returned.

#### Example
SQL Query
SQL
SELECT
    id,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM
    leaderboard
LIMIT 10;

This query will ensure that the top 10 rows, including ties, are displayed correctly. If you only want to display unique scores within those top 10, DENSE_RANK is ideal.

**Comparison:**

|id|score|dense_rank|
|123|95   | 1        |
|456|87   | 2        |
|789|87   | 2        |
|012|76   | 4        |
|345|76   | 4        |

**Code:**

```sql
-- Example of using DENSE_RANK with a LIMIT
SELECT
    id,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM
    leaderboard
LIMIT 10;
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT
    id,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM
    leaderboard;

Output:

PLAINTEXT
Copy
|id|score|dense_rank|
+--+-----+----------+
|123|95   | 1        |
|456|87   | 2        |
|789|87   | 2        |
|012|76   | 4        |
|345|76   | 4        |

Input:

SQL
Copy
SELECT
    id,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM
    leaderboard
LIMIT 10;

Output:

PLAINTEXT
Copy
|id|score|dense_rank|
+--+-----+----------+
|123|95   | 1        |
|456|87   | 2        |
|789|87   | 2        |
|012|76   | 4        |
|345|76   | 4        |

Explanation:

The LIMIT ensures that only the top 10 rows are displayed, and ties are handled correctly by DENSE_RANK.
```

**Data:**

```
Sample data:
SQL
Copy
CREATE TABLE leaderboard (
    id INT PRIMARY KEY,
    score INT
);

INSERT INTO leaderboard (id, score) VALUES
(123, 95),
(456, 87),
(789, 87),
(012, 76),
(345, 76),
(678, 65),
(901, 54),
(123, 54),
(456, 54);
```

**Flow:**

1. - Flowchart for DENSE_RANK and LIMIT:
2. Yes
3. No
4. Input Query
5. DENSE_RANK with ORDER BY
6. Tie?
7. Dense Rank Continues
8. Single Score Handling
9. LIMIT 10 to Top Ranks
10. Continue Ranking
11. Output Top 10 Rows

