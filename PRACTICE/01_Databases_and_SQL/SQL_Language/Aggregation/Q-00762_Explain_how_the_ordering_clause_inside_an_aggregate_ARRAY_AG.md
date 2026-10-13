---
type: "practice"
difficulty: "Deep"
---

# Explain how the ordering clause inside an aggregate — ARRAY_AGG(x ORDER BY y) or STRING_AGG(x, ',' ORDER BY y) — works and why forgetting it produces unstable output.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ GROUP BY and HAVING](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL databases, especially PostgreSQL, you might encounter scenarios where you need to collect a list of values grouped by some criteria, possibly ordered in a specific way. The ARRAY_AGG and STRING_AGG functions are powerful tools for this purpose. However, understanding how the ORDER BY clause inside these aggregate functions works is crucial, as forgetting it can lead to inconsistent results.

#### Triaging the Question

The question asks us to explain the behavior of the ORDER BY clause within ARRAY_AGG(x ORDER BY y) and STRING_AGG(x, ',' ORDER BY y), specifically focusing on why omitting this clause leads to unstable output. The key points we need to cover are:

Behavior without an Order By Clause: Without a specified order, the results can be unpredictable.
How ORDER BY Works Inside Aggregate Functions: It applies the ordering after grouping but before aggregation.
Stability and Unpredictable Behavior: Understanding why forgetting ORDER BY leads to unstable output.

#### First-Principles Solution

Let's start with a brief overview: When using aggregate functions like ARRAY_AGG or STRING_AGG, it’s essential to specify an ORDER BY clause. This is because the order in which rows are processed can affect the result, especially when grouping data.

1. Behavior Without ORDER BY Clause

When you use a function like ARRAY_AGG(x), PostgreSQL doesn’t know how to order the elements unless you explicitly tell it. For example, consider this query:

SQL
SELECT id, ARRAY_AGG(name)
FROM employees
GROUP BY id;

Without an ORDER BY clause, the resulting array might look something like this:

For id = 1: ['Alice', 'Bob']
For id = 2: ['Charlie', 'David']

Notice that the order of names is not preserved. This behavior can be quite surprising and problematic if you need consistent ordering.

2. How ORDER BY Works Inside Aggregate Functions

The ORDER BY clause inside aggregate functions works by first grouping the data (if necessary) and then applying the order to the aggregated result. Consider this query:

SQL
SELECT id, ARRAY_AGG(name ORDER BY name)
FROM employees
GROUP BY id;

Here’s what happens step-by-step:

Grouping: The id column is used for grouping.
Sorting Within Each Group: For each group, the names are sorted alphabetically (or based on the specified order).
Aggregation: The sorted names are then aggregated into an array.

Using this approach, you get a consistent and predictable result:

For id = 1: ['Alice', 'Bob']
For id = 2: ['Charlie', 'David']
3. Stability and Unpredictable Behavior

The key reason why forgetting the ORDER BY clause can lead to unstable output is that SQL engines are not required to maintain a consistent order for rows unless explicitly specified with an ORDER BY. For example, consider this query without any sorting:

SQL
SELECT id, ARRAY_AGG(name)
FROM employees
GROUP BY id;

Due to the lack of ORDER BY, the output might be:

For id = 1: ['Bob', 'Alice']
For id = 2: ['David', 'Charlie']

As you can see, the order of names changes between different executions or even within a single execution if the underlying data is updated. This unpredictability makes it challenging to rely on the results for critical applications.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using ORDER BY inside aggregate functions like ARRAY_AGG and STRING_AGG ensures that your output remains consistent and stable. Omitting this clause can lead to unpredictable results because SQL engines do not guarantee the order of rows unless explicitly instructed.

#### Common mistakes
include:

Forgetting to specify an ORDER BY clause in queries where ordering is critical.
Assuming that the default processing order provided by SQL engines will meet your needs without explicit sorting.

#### Follow-ups
might involve discussing how different database systems handle this behavior or exploring alternative methods for achieving stable results when grouping and ordering data.

