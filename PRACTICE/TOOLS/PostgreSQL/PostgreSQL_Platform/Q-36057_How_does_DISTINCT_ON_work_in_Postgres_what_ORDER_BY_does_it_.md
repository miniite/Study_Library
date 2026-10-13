---
type: "practice"
difficulty: "Intermediate"
---

# How does DISTINCT ON work in Postgres, what ORDER BY does it require to be deterministic, and why do people get flaky results when they forget the tiebreaker column?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Distinct On is a powerful feature in PostgreSQL that allows you to retrieve unique rows from a query result set based on a specific ordering. The key idea behind DISTINCT ON is that it selects one row from each "group" of identical values defined by the ORDER BY clause, but only if there are no other groups with different values for the columns specified in the SELECT statement.

#### Triage

This question asks you to explain how DISTINCT ON works and why order matters. The core aspect here involves understanding the relationship between DISTINCT ON and ORDER BY. Specifically, you need to know that DISTINCT ON requires an ORDER BY clause to work deterministically, otherwise, results can be unpredictable or "flaky."

#### First-Principles Solution

Distinct On operates by first sorting the rows according to the ORDER BY clause and then picking one row from each set of identical values defined by the selected columns. Here’s a step-by-step breakdown:

Sorting: The table is logically sorted based on the ORDER BY clause.
Selection: Rows are then selected starting from the top of the sorted result, keeping only the first occurrence for each group identified by the SELECT statement.
Why ORDER BY Is Crucial

The ORDER BY clause is essential because it ensures that rows are processed in a consistent order. Without this order, PostgreSQL does not know how to select one row per unique value since multiple rows might have the same values but appear at different positions without an explicit ordering.

#### Example
Flaky Results Due to Missing Tiebreaker Column

#### Cons
ider the following example:

SQL
SELECT id, name FROM users ORDER BY name;

This query alone does not provide a deterministic result for DISTINCT ON because it doesn’t specify which row to return when multiple rows have the same name. You might get different results each time you run the query if there are duplicates in the name column.

To fix this, you need to include an additional column that breaks ties. For instance:

SQL
SELECT id, name FROM users ORDER BY name, age DESC;

In this case, age serves as a tiebreaker for rows with the same name. PostgreSQL will first sort by name, and then within each group of identical names, it will further sort by age DESC, ensuring that only one row is selected per unique name.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, DISTINCT ON requires an ORDER BY clause to ensure deterministic results. Without a proper tiebreaker in the ORDER BY clause, you can get inconsistent and unpredictable outcomes. Always include columns in your ORDER BY that are relevant to the uniqueness of your result set.

#### Common mistakes
people make when using DISTINCT ON include:

Failing to include any ORDER BY clause.
Including only a single column in the ORDER BY that doesn’t provide sufficient uniqueness, leading to flaky results.

#### Follow-up
questions might involve discussing performance considerations, or how DISTINCT ON behaves with subqueries or common table expressions (CTEs).

**Code:**

```sql
-- Example of using DISTINCT ON with a tiebreaker
SELECT id, name FROM users ORDER BY name, age DESC;
```

