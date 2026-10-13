---
type: "practice"
difficulty: "Basic"
---

# How do you fetch only the first 10 rows of a query result in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To fetch only the first 10 rows of a query result in PostgreSQL, you can use the LIMIT clause. This is a straightforward operation that allows you to retrieve a limited number of records from your database based on certain conditions or simply by row count. Here’s how it works and what you need to consider.

#### Triaging the Question

The question asks about using the LIMIT clause in PostgreSQL to fetch only the first 10 rows. The interviewer might be probing for an understanding of basic SQL syntax, familiarity with PostgreSQL's query language, or awareness of common pitfalls when working with database queries.

#### First-Principles Solution

In PostgreSQL (and many other SQL databases), you can use the LIMIT clause to restrict the number of returned rows from a query. The general format for using LIMIT is:

SQL
SELECT * FROM table_name LIMIT n;

Here, n represents the number of rows you want to fetch. For example, if you want the first 10 rows, your SQL statement would be:

SQL
SELECT * FROM employees LIMIT 10;

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, the correct way to fetch only the first 10 rows in PostgreSQL is by using the LIMIT clause. However, it’s important to note that if you don’t specify an ORDER BY clause, the results are not guaranteed to be consistent across different executions unless there's a unique primary key or some other deterministic order.

#### Common mistakes
include:

Forgetting to use an ORDER BY clause, which can lead to unpredictable results.
Using incorrect syntax for the LIMIT statement (e.g., forgetting to specify the exact number of rows).

#### Follow-ups
could involve discussing scenarios where pagination is necessary and exploring how to handle larger result sets efficiently.

**Code:**

```sql
-- Example query
SELECT * FROM employees LIMIT 10;
```

