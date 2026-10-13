---
type: "practice"
difficulty: "Deep"
---

# What happens if you use HAVING without a GROUP BY clause?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ GROUP BY and HAVING](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using aggregate functions in SQL, it's crucial to understand how HAVING and GROUP BY work together. The HAVING clause is used to filter groups after they have been aggregated, but it can only be used effectively when a GROUP BY clause is present. If you use HAVING without a GROUP BY, the database engine will raise an error because there's no context for which rows to aggregate and then apply the filtering condition.

Let’s dive into this with a practical example:

#### Overview

The core issue here is understanding that HAVING requires the presence of a GROUP BY clause.
Using HAVING without GROUP BY results in an SQL syntax error.

#### Triage
and Assumptions:

The question asks us to explain what happens if you use HAVING without a GROUP BY clause.
We assume the query is being executed on a standard relational database system that adheres to SQL standards (like PostgreSQL, MySQL, or SQL Server).

#### First-Principles Solution

To understand this, let's start with the basics of what HAVING and GROUP BY do.
The GROUP BY clause groups rows that have the same values in specified columns into summary rows.
The HAVING clause filters these summary rows. It works on the result set after a group has been applied.

Here's an example to illustrate:

SQL
-- Incorrect SQL (Causes Syntax Error)
SELECT column1, COUNT(*)
FROM table_name
HAVING COUNT(*) > 5;

This query will fail with an error because there is no GROUP BY clause. The COUNT(*) function needs a context in which rows are grouped together.

To fix this, we need to add the necessary GROUP BY:

SQL
-- Correct SQL
SELECT column1, COUNT(*)
FROM table_name
GROUP BY column1
HAVING COUNT(*) > 5;

#### Closing Summary
and Common Mistakes/Follow-Ups:
Always ensure that you use a GROUP BY clause when using the HAVING clause.
A common mistake is forgetting to include GROUP BY, which can lead to runtime errors.
Another potential pitfall is incorrectly applying aggregate functions in the SELECT clause without matching them with the correct GROUP BY.

In summary, HAVING and GROUP BY are inseparable; HAVING filters groups, but only after they have been created using a GROUP BY. Attempting to use HAVING without GROUP BY is a fundamental SQL syntax error.

**Code:**

```sql
-- Incorrect example (Causes Syntax Error)
SELECT column1, COUNT(*)
FROM table_name
HAVING COUNT(*) > 5;

-- Correct example
SELECT column1, COUNT(*)
FROM table_name
GROUP BY column1
HAVING COUNT(*) > 5;
```

