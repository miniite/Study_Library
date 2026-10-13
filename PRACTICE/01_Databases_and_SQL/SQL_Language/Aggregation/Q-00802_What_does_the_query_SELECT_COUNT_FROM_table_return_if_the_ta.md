---
type: "practice"
difficulty: "Basic"
---

# What does the query SELECT COUNT(*) FROM table return if the table is empty?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ COUNT Semantics](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The query SELECT COUNT(*) FROM table returns 0 when the table is empty. This simple SQL query is used to count the number of rows in a table, and it will yield zero if no rows exist.

To understand this better, let's break down the concept of aggregate functions like COUNT(*). These functions are typically used to perform calculations on a set of values. In this case, COUNT(*) counts all the rows in the table, regardless of column values. When there are no rows, naturally, the count will be zero.

Here’s a step-by-step explanation:

The SQL query SELECT COUNT(*) FROM table is requesting that the database count the number of rows in the specified table.
If the table has no rows, the count function returns 0 because there are zero entries to count.
This result is independent of any data in other columns; it only depends on the presence or absence of rows.

In practice, this query can be used to check if a table contains any records before performing operations that might require row existence. For example:

SQL
SELECT COUNT(*) AS num_rows FROM my_table;

If num_rows is 0, then we know the table is empty.

#### Common mistakes
and follow-ups include not considering edge cases or forgetting about aggregate functions' behavior on empty tables.

