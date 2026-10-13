---
type: "practice"
difficulty: "Basic"
---

# What is the difference between joining tables and using UNION to combine results?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Joins vs Subqueries and Join Types](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, particularly when working with relational databases, understanding how to join tables versus combining their results using UNION is crucial. Both operations are fundamental in manipulating data, but they serve different purposes and have distinct functionalities.

TL;DR:

Joins combine rows from two or more tables based on a related column between them.
UNION combines the result sets of two or more SELECT statements by removing duplicate rows.

#### Triaging the Question

The question essentially asks you to explain the difference between joining tables and using UNION to combine results. The key idea here is understanding that these are different operations with distinct objectives:

Joins aim to retrieve related data from multiple tables, often for a specific operation like filtering or calculating.
UNION combines the result sets of two or more queries into one set, removing any duplicate rows.

#### Assumptions
You understand basic concepts of SQL and relational databases.
The questions are about typical INNER JOIN, LEFT JOIN, RIGHT JOIN, and FULL OUTER JOIN.
UNION is being used without a UNION ALL to remove duplicates.

#### First-Principles Solution

Let's start with Joins:

Purpose: Joins are used when you need to combine data from two or more tables based on related columns.
Syntax: Common join types include INNER JOIN, LEFT JOIN, RIGHT JOIN, and FULL OUTER JOIN.

#### Example
SQL
SELECT a.column1, b.column2
FROM table_a AS a
INNER JOIN table_b AS b ON a.common_column = b.common_column;
Result: The result set contains only rows where the join condition is met.

Now, let's move on to UNION:

Purpose: UNION combines the results of multiple SELECT statements.
Syntax:
SQL
SELECT column1 FROM table_a
UNION
SELECT column1 FROM table_b;
Result: The result set contains all unique rows from both SELECT statements.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Key Differences:
Joins combine data based on related columns, whereas UNION combines results without considering relationships.
Joins may return fewer rows due to the join condition, while UNION returns all unique rows from both queries.

#### Common Mistakes
Confusing JOIN with UNION ALL, which includes duplicates.
Using joins in situations where a simple union of results would be more appropriate or vice versa.

#### Follow-Ups
Can you explain the difference between INNER JOIN, LEFT JOIN, and RIGHT JOIN?
How do subqueries fit into these operations, and when might they be used instead?

