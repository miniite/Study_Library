---
type: "practice"
difficulty: "Basic"
---

# What is a FULL OUTER JOIN and what rows does it return?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: A FULL OUTER JOIN combines the results of both INNER and LEFT and RIGHT JOINs, returning all records from both tables, filling in NULL values where there are no matching records.

Question Breakdown: The interviewer is probing your understanding of SQL join operations specifically focusing on the behavior of a FULL OUTER JOIN. You should assume basic knowledge of relational databases and common join types like INNER, LEFT, and RIGHT joins.

#### Solution

A FULL OUTER JOIN returns all rows from both tables being joined (Table A and Table B), filling in NULL values where there is no matching record in one or the other table. Here’s a step-by-step explanation:

Understanding INNER JOIN: This type of join only returns records that have matching entries in both tables.
LEFT JOIN (Left Outer Join): Returns all records from the left table and the matched records from the right table. If there is no match, the result is NULL on the side where there is no match.
RIGHT JOIN (Right Outer Join): This works similarly to LEFT JOIN but returns all records from the right table with NULLs in columns from the left table.

Now, let's combine these concepts into a FULL OUTER JOIN:

A row will be included if it matches in either Table A or Table B.
If there is no match, the result for that row will have NULL values where there are no matches on the other side.

Here’s an example to illustrate this using T-SQL:

SQL
SELECT
    a.Column1 AS A_Column1,
    a.Column2 AS A_Column2,
    b.Column3 AS B_Column3,
    b.Column4 AS B_Column4
FROM
    TableA a
FULL OUTER JOIN
    TableB b
ON
    a.KeyColumn = b.KeyColumn;

In this query:

TableA and TableB are the tables being joined.
The KeyColumn is the column on which the join condition is based.
If there's no match between a.KeyColumn and b.KeyColumn, both columns in the result will contain NULLs for that row.

#### Closing Summary
A FULL OUTER JOIN ensures you get all records from both tables, with NULLs filling in where there are no matches. This can be incredibly useful when you need to compare or combine data across two datasets without losing any rows due to missing matches.

#### Common Mistakes/Follow-Ups

Misunderstanding the difference between FULL OUTER JOIN and UNION with INNER JOIN.
Failing to consider performance implications, especially on large tables.
Not realizing that NULL values are used to fill in gaps where there is no match.

**Comparison:**

| TableA                  |           | TableB                  |
| ------------------------|-----------|------------------------|
| KeyColumn (String)       | Column1 (Int) | Column2 (String)      |
|                         | Column3 (Int) | Column4 (String)      |
| Full Outer Join         | A_Column1 (Int) | A_Column2 (String) | B_Column3 (Int) | B_Column4 (String) |
| ------------------------|-----------------|--------------------|-----------------|--------------------|
| KeyColumn (String)       | NULL            | 'Alpha'            | 100             | 10                 |
|                         | 200             | 'Y'                | NULL            | NULL               |
|                         | 300             | 'Z'                | NULL            | NULL               |
|                         | NULL            | NULL               | 20              | 'Beta'             |
|                         | NULL            | NULL               | 30              | 'Gamma'            |

**Code:**

```sql
SELECT
    a.Column1 AS A_Column1,
    a.Column2 AS A_Column2,
    b.Column3 AS B_Column3,
    b.Column4 AS B_Column4
FROM
    TableA a
FULL OUTER JOIN
    TableB b
ON
    a.KeyColumn = b.KeyColumn;
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT
    a.Column1 AS A_Column1,
    a.Column2 AS A_Column2,
    b.Column3 AS B_Column3,
    b.Column4 AS B_Column4
FROM
    TableA a
FULL OUTER JOIN
    TableB b
ON
    a.KeyColumn = b.KeyColumn;

Output:

PLAINTEXT
Copy
KeyColumn | Column1 | Column2 | Column3 | Column4
--------- | ------- | ------- | ------- | ------
    A     |   100   | 'X'     |   10    | 'Alpha'
    B     |   200   | 'Y'     | NULL    | NULL
    C     |   300   | 'Z'     | NULL    | NULL
    D     | NULL    | NULL    |   20    | 'Beta'
    E     | NULL    | NULL    |   30    | 'Gamma'
```

**Data:**

```

Example data for TableA:

PLAINTEXT
Copy
KeyColumn | Column1 | Column2
--------- | ------- | ------
    A     |   100   | 'X'
    B     |   200   | 'Y'
    C     |   300   | 'Z'

Example data for TableB:

PLAINTEXT
Copy
KeyColumn | Column3 | Column4
--------- | ------- | ------
    A     |   10    | 'Alpha'
    D     |   20    | 'Beta'
    E     |   30    | 'Gamma'

Expected result after FULL OUTER JOIN:

PLAINTEXT
Copy
KeyColumn | Column1 | Column2 | Column3 | Column4
--------- | ------- | ------- | ------- | ------
    A     |   100   | 'X'     |   10    | 'Alpha'
    B     |   200   | 'Y'     | NULL    | NULL
    C     |   300   | 'Z'     | NULL    | NULL
    D     | NULL    | NULL    |   20    | 'Beta'
    E     | NULL    | NULL    |   30    | 'Gamma'
```

**Flow:**

1. ONKeyColumn=b.KeyColumn
2. FULL OUTER JOIN
3. Yes
4. No
5. TableA
6. TableB
7. Full Outer Join
8. KeyMatch
9. A_Column1, A_Column2,B_Column3, B_Column4
10. NULL, NULL, NULL, NULL

