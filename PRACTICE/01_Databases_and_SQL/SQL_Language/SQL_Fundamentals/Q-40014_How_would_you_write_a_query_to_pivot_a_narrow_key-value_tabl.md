---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to pivot a narrow key-value table (attribute, value) into a wide table for reporting?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Expanding Date Ranges](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Expanding_Date_Ranges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to transform a narrow key-value table into a wide table format suitable for reporting purposes. This is commonly known as pivoting in SQL. The key challenge here is that each row might have multiple attributes (keys), and we want to move these keys to columns while retaining the associated values.

#### Triage

The question is essentially asking how to convert data from a long, narrow table format into a wide table format. This involves using aggregate functions like SUM, MAX, or even CASE statements to transform rows into columns. The interviewer might be probing your understanding of SQL and your ability to handle complex queries.

#### Solution

Let’s assume we have the following schema for our key-value table:

SQL
CREATE TABLE attributes (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    value VARCHAR(255)
);

The goal is to pivot this table so that each name becomes a column, and the corresponding value appears in these new columns. For example, if we have:

id	name	value
1	color	red
1	size	medium
2	color	blue
2	size	large

We want to transform it into:

id	color	size
1	red	medium
2	blue	large
Step-by-Step Solution
Identify Unique Keys: First, identify the unique keys (attributes) in your data.
Use Aggregate Functions: Use aggregate functions to group and summarize the values based on these keys.

Here is a SQL query that accomplishes this:

SQL
SELECT id,
       MAX(CASE WHEN name = 'color' THEN value END) AS color,
       MAX(CASE WHEN name = 'size' THEN value END) AS size
FROM attributes
GROUP BY id;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query uses CASE statements to conditionally include the values in the resulting columns. It dynamically creates new columns based on the distinct keys in the name column. The MAX function is used here because it handles any non-null value for each key, but other aggregate functions like MIN, AVG, or even custom aggregation logic could be used depending on your specific use case.

#### Common mistakes
include:

Not considering all possible attribute values.
Misusing the GROUP BY clause, leading to incorrect aggregations.
Forgetting to handle NULL values properly in the CASE statements.

#### Follow-ups
might involve discussing performance considerations, handling missing attributes, or dealing with a very large number of unique keys.

