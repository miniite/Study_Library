---
type: "practice"
difficulty: "Deep"
---

# What does SUM return if the table is empty versus what COUNT returns for the same empty table?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ COUNT Semantics](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Let's dive into a fundamental aspect of SQL: aggregate functions and their behavior with an empty table.

TL;DR: SUM will return 0, while COUNT will return NULL if the table is empty. This distinction is crucial for understanding how these functions handle edge cases in database queries.

When you encounter questions about aggregate functions, it's important to consider both what they are designed to do and their behavior under different conditions. In this case, we'll explore two common aggregate functions: SUM and COUNT.

#### Triage
The question is asking us to understand the difference between SUM and COUNT when applied to an empty table. The interviewer might be probing your understanding of how these functions work at a fundamental level and their practical implications.

We need to assume that we are dealing with a SQL database, specifically a relational one, as this context is most relevant for aggregate functions like SUM and COUNT.

#### First-Principles Solution

Understanding SUM:

The SUM function adds up all the values in a column.
If there are no rows to sum (i.e., an empty table), it must return 0 because adding zero numbers should result in zero.

Understanding COUNT:

The COUNT function counts the number of non-NULL items in a specified column or all columns if no specific column is provided.
For an empty table, there are no rows to count, so it should return NULL since there's nothing to count.

To illustrate this with some SQL examples:

SQL
-- Example for SUM
CREATE TABLE empty_table (value INT);

SELECT SUM(value) FROM empty_table;

The query above will output 0 because the sum of zero values is zero.

SQL
-- Example for COUNT
SELECT COUNT(*) FROM empty_table;

This query will return NULL since there are no rows in the table to count.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, when dealing with an empty table:

SUM returns 0 because it’s summing up zero values.
COUNT returns NULL because there are no items to count.

#### Common mistakes
could involve assuming that both functions behave the same way or not considering edge cases like this. Always double-check the behavior of aggregate functions in your database documentation.

For further discussion, you might be asked about how these results impact query logic, especially in scenarios where you need to handle empty tables gracefully.

**Code:**

```sql
CREATE TABLE empty_table (value INT);

-- Example for SUM
SELECT SUM(value) FROM empty_table;

-- Example for COUNT
SELECT COUNT(*) FROM empty_table;
```

