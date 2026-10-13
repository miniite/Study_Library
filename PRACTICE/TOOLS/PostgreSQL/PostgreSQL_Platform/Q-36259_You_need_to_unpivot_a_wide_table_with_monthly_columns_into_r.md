---
type: "practice"
difficulty: "Intermediate"
---

# You need to unpivot a wide table with monthly columns into row-per-month form for reporting; how would you do this with a LATERAL join over a VALUES list versus a UNION ALL, and which scales better?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of unpivoting a wide table into row-per-month form using PostgreSQL, we need to consider two approaches: using a LATERAL JOIN over a VALUES list and using a UNION ALL. Both methods achieve the goal of transforming the data from columns (months) to rows. However, their performance characteristics differ based on the size and structure of your dataset.

#### Triaging the Question

The core of this question is about comparing two SQL techniques: LATERAL JOIN with VALUES and UNION ALL. The interviewer wants to understand how you approach a specific data transformation problem and evaluate your knowledge of PostgreSQL's features. Assumptions include that we have a wide table where each month has its own column, and the goal is to convert this into a form where each row represents one month.

#### First-Principles Solution
Using LATERAL JOIN with VALUES

Concept: A LATERAL JOIN allows us to use the result set of an inner query in the outer query's context. In our case, we can generate the list of months using a VALUES clause and join it laterally with the original table.

#### Implementation

SQL
WITH month_list AS (
    SELECT * FROM (VALUES
        ('Jan'), ('Feb'), ('Mar'), ('Apr'), ('May'), ('Jun'),
        ('Jul'), ('Aug'), ('Sep'), ('Oct'), ('Nov'), ('Dec')
    ) AS months(month)
)
SELECT t.*, m.month
FROM wide_table t
LEFT JOIN LATERAL (SELECT * FROM month_list) m ON true;

Explanation: The WITH clause generates a list of months, and the LATERAL JOIN ensures that each row in wide_table is joined with every month from month_list. This method scales well for smaller datasets but might become slow if there are many rows or months.

Using UNION ALL

Concept: A UNION ALL operation concatenates multiple result sets, which can be used to simulate the unpivot transformation by joining each month's column with a single row identifier.

#### Implementation

SQL
SELECT 'Jan' AS month, col1 FROM wide_table UNION ALL
SELECT 'Feb', col2 FROM wide_table UNION ALL
SELECT 'Mar', col3 FROM wide_table ... UNION ALL
SELECT 'Dec', col12 FROM wide_table;

Explanation: This method explicitly constructs a new row for each month, ensuring that the data is in the desired format. However, it requires writing out every column and can become cumbersome for large tables.

#### Closing Summary

Both methods achieve the same goal but have different trade-offs. The LATERAL JOIN with VALUES is more concise and easier to write, especially when dealing with a smaller number of months. It also benefits from PostgreSQL's ability to handle lateral joins efficiently. However, it may not scale well for very large datasets or tables.

On the other hand, using UNION ALL provides explicit control over each row transformation but can be prone to errors if you miss adding a column in your query.

#### Common Mistakes
and Follow-Ups
Mistake: Forgetting to handle NULL values correctly when unpivoting. Ensure that any missing months (columns) are represented as NULL.

#### Follow-Up
Discuss the impact of data cardinality on performance. In cases with many rows, LATERAL JOIN might be slower due to its join nature, whereas UNION ALL can handle more efficiently.
Further Discussion: Consider the use of window functions or CTEs (Common Table Expressions) for handling complex unpivot scenarios.

**Comparison:**

|month|Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec|
|---|---|---|---|---|---|---|---|---|---|---|---|---|
|row1|1|2|3|4|5|6|7|8|9|10|11|12|
|row2|13|14|15|16|17|18|19|20|21|22|23|24|

**Code:**

```

WITH month_list AS ( SELECT * FROM (VALUES ('Jan'), ('Feb'), ('Mar'), ('Apr'), ('May'), ('Jun'), ('Jul'), ('Aug'), ('Sep'), ('Oct'), ('Nov'), ('Dec') ) AS months(month) ) SELECT t.*, m.month FROM wide_table t LEFT JOIN LATERAL (SELECT * FROM month_list) m ON true;

CODE
Copy

```sql
SELECT 'Jan' AS month, col1 FROM wide_table UNION ALL
SELECT 'Feb', col2 FROM wide_table UNION ALL
SELECT 'Mar', col3 FROM wide_table ... UNION ALL
SELECT 'Dec', col12 FROM wide_table;
```

**Example input / output:**

```text
Input:
wide_table = [
    {"Jan": 1, "Feb": 2, "Mar": 3, "Apr": 4, "May": 5, "Jun": 6, "Jul": 7, "Aug": 8, "Sep": 9, "Oct": 10, "Nov": 11, "Dec": 12},
    {"Jan": 13, "Feb": 14, "Mar": 15, "Apr": 16, "May": 17, "Jun": 18, "Jul": 19, "Aug": 20, "Sep": 21, "Oct": 22, "Nov": 23, "Dec": 24}
]

Output:
[
    {"month": "Jan", "col1": 1, "col2": 13},
    {"month": "Feb", "col1": 2, "col2": 14},
    {"month": "Mar", "col1": 3, "col2": 15},
    ...
    {"month": "Dec", "col1": 12, "col2": 24}
]
```

**Data:**

```json
{
    "wide_table": [
        {"Jan": 1, "Feb": 2, "Mar": 3, "Apr": 4, "May": 5, "Jun": 6, "Jul": 7, "Aug": 8, "Sep": 9, "Oct": 10, "Nov": 11, "Dec": 12},
        {"Jan": 13, "Feb": 14, "Mar": 15, "Apr": 16, "May": 17, "Jun": 18, "Jul": 19, "Aug": 20, "Sep": 21, "Oct": 22, "Nov": 23, "Dec": 24}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the dataset large?
5. LATERAL JOIN with VALUES
6. UNION ALL
7. Handle NULL values
8. Simplify query writing
9. Ensure performance isacceptable
10. Simplify query writing

