---
type: "practice"
difficulty: "Basic"
---

# What happens if you forget ORDER BY inside OVER when using ROW_NUMBER()?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, window functions like ROW_NUMBER() are used to assign a unique row number within a partition of a result set. However, forgetting the ORDER BY clause within the OVER() clause can lead to unpredictable results. Let's explore why this happens and understand the implications.

#### Triage
What’s Being Probed

The question is probing your understanding of how SQL window functions work, specifically focusing on the importance of using an ORDER BY clause in conjunction with ROW_NUMBER(). The interviewer might be assessing whether you are aware that omitting ORDER BY can cause issues and if you understand why.

#### Assumptions
and Core Solution

When writing a query involving ROW_NUMBER() without specifying an ORDER BY, the SQL engine will use some default or arbitrary order to assign row numbers. This order is typically based on the table's internal storage structure, which may not align with any meaningful order intended by the user.

Step-by-Step Explanation from First Principles
Understanding Window Functions: A window function processes a set of rows that are related to the current row within a partition defined by a PARTITION BY clause.
ROW_NUMBER() Function: This assigns a unique sequential integer starting at 1 for each row within the partition, and this number is ordered in ascending or descending order based on the ORDER BY clause specified within the OVER() clause.
The Problem with Omitting ORDER BY:
Without an ORDER BY, SQL engines often use a default ordering mechanism (e.g., primary key order) which may not reflect any meaningful sequence of data.
This can lead to row numbers that are meaningless and inconsistent, leading to incorrect results.

#### Example Scenario

#### Cons
ider the following table:

id	name
1	Alice
2	Bob
3	Carol

A query using ROW_NUMBER() without an ORDER BY would look like this:

SQL
SELECT id, name, ROW_NUMBER() OVER () AS row_num
FROM table_name;

In this case, the SQL engine might use the primary key or another default ordering method. The output could be:

id	name	row_num
3	Carol	1
2	Bob	2
1	Alice	3

Notice how the row numbers are assigned based on the engine’s internal order, which may not be as expected.

#### Closing Summary
and Common Mistakes

Remember that using ROW_NUMBER() without an ORDER BY can lead to non-deterministic results. It is crucial to specify an ORDER BY clause to ensure consistent and meaningful row numbering. The common mistake here is assuming that the query will behave as if there were an implicit ordering, which is not guaranteed by SQL standards.

Common Follow-Ups
What happens with different window functions?
How can you handle non-deterministic results in production code?
Can you give an example where omitting ORDER BY might lead to issues?

**Comparison:**

| id | name  | row_num |
|----|-------|---------|

**Code:**

```sql
-- Example of correct usage with ORDER BY
SELECT id, name, ROW_NUMBER() OVER (ORDER BY id) AS row_num
FROM table_name;

-- Example without ORDER BY
SELECT id, name, ROW_NUMBER() OVER () AS row_num
FROM table_name;
```

**Example input / output:**

| Input                | Output                           |
|----------------------|----------------------------------|
| Correct Usage        | [id, name, row_num]              |
| Without ORDER BY     | [id, name, row_num] (non-deterministic) |

**Data:**

| id | name  |
|----|-------|
| 1  | Alice |
| 2  | Bob   |
| 3  | Carol |

**Flow:**

1. Define Window Function
2. Choose ROW_NUMBER()
3. Specify ORDER BY Clause
4. Execute Query withDeterministic Results
5. Execute Query without ORDERBY, Non-deterministic Results

