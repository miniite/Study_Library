---
type: "practice"
difficulty: "Basic"
---

# How do you sort query results by two columns, one ascending and one descending?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To sort SQL query results by multiple columns with different order directions, you can use the ORDER BY clause in your SQL statement. Specifically, you'll want to specify which column should be sorted in ascending order and which in descending order.

#### Triaging the Question

The question is asking how to perform a multi-column sort in an SQL query where one of the columns sorts results in ascending order (ASC), while the other sorts them in descending order (DESC). The main concepts involved are understanding the ORDER BY clause and its syntax.

#### First-Principles Solution

To achieve this, you would write your SQL query like so:

SQL
SELECT column1, column2, ...
FROM table_name
ORDER BY column1 ASC, column2 DESC;

Here's a breakdown of what each part does:

SELECT column1, column2, ...: This specifies which columns you want to retrieve from the database.
FROM table_name: This indicates the name of the table from which to fetch data.
ORDER BY column1 ASC, column2 DESC;: The ORDER BY clause sorts the results based on the specified columns. In this case, column1 is sorted in ascending order (ASC), and column2 is sorted in descending order (DESC).

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, when sorting by multiple columns with different order directions, you can use the ORDER BY clause directly within your SQL query. The key is to separate each column you want to sort by with a comma and specify the direction of each sort using ASC or DESC.

A common mistake is forgetting to include the table name in your query. Always ensure that all referenced columns belong to an existing table.

For follow-ups, consider discussing performance implications if sorting on large datasets, the impact of indexes, and whether other sorting strategies might be more efficient depending on the data distribution.

**Code:**

```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column1 ASC, column2 DESC;
```

