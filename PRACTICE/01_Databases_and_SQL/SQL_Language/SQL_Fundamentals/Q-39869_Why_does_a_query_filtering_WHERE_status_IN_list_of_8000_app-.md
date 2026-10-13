---
type: "practice"
difficulty: "Deep"
---

# Why does a query filtering WHERE status IN (list of 8000 app-generated literals) produce an awful plan, and what rewrites (VALUES join, temp table, unnest array) stabilize it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you run a query with WHERE status IN ('value1', 'value2', ..., 'value8000'), the database optimizer often generates an inefficient execution plan because it assumes that the list is randomly distributed and not indexed. This leads to poor performance, especially when dealing with such a large number of values.

To understand this better, let's break down the issue from first principles:

Optimizer Assumptions: The SQL query optimizer usually makes assumptions about data distribution based on statistics. When you use IN clauses with many literals, it cannot efficiently utilize indexes or other optimization strategies.
Execution Plan Efficiency: With 8000 values, the optimizer might choose a nested loop join or full table scan, which is highly inefficient and can take a significant amount of time.

To address this issue, we need to explore alternative query rewrites that can help the database use indexes more effectively. Here are three possible solutions:

#### Solution
1: VALUES Join

One effective approach is to transform the IN clause into an EXISTS or JOIN with a temporary table containing the values.

Step-by-Step Process:
Create a Temporary Table: Create a temporary table with the 8000 values.
Use JOIN: Modify the query to use a JOIN operation instead of the IN clause.
SQL
CREATE TEMPORARY TABLE temp_statuses AS
SELECT 'value1' AS status UNION ALL SELECT 'value2' AS status ... UNION ALL SELECT 'value8000' AS status;

-- Original Query with IN Clause:
SELECT * FROM app_data WHERE status IN ('value1', 'value2', ..., 'value8000');

-- Re-written Query with JOIN:
SELECT * FROM app_data WHERE EXISTS (SELECT 1 FROM temp_statuses WHERE temp_statuses.status = app_data.status);

#### Solution
2: UNNEST Array

Another approach is to use the UNNEST function, which can be particularly useful in PostgreSQL and other databases that support it.

Step-by-Step Process:
Create an Array: Store the 8000 values in a single array.
Use UNNEST: Modify the query to use the UNNEST function.
SQL
-- Create an array of statuses:
CREATE OR REPLACE TEMPORARY FUNCTION get_statuses AS
SELECT ARRAY['value1', 'value2', ..., 'value8000'];

-- Original Query with IN Clause:
SELECT * FROM app_data WHERE status IN ('value1', 'value2', ..., 'value8000');

-- Re-written Query with UNNEST:
WITH statuses AS (SELECT get_statuses()::TEXT[])
SELECT * FROM app_data WHERE status = ANY (UNNEST(statuses));

#### Solution
3: Temp Table

Using a temporary table as mentioned in the VALUES join solution can also help. This approach ensures that the database has more control over how to handle the values.

Step-by-Step Process:
Create and Populate Temp Table: Create a temp table with the 8000 values.
Join Query:
SQL
CREATE TEMPORARY TABLE temp_statuses (status TEXT);
INSERT INTO temp_statuses VALUES ('value1'), ('value2'), ..., ('value8000');

-- Original Query with IN Clause:
SELECT * FROM app_data WHERE status IN ('value1', 'value2', ..., 'value8000');

-- Re-written Query with JOIN:
SELECT * FROM app_data
JOIN temp_statuses ON app_data.status = temp_statuses.status;

#### Closing Summary

By using VALUES join, UNNEST, or creating a temporary table, you can improve the performance of queries involving large IN clauses. These approaches allow the database to utilize indexes more effectively and generate better execution plans.

#### Common Mistakes/Follow-ups
Ignoring Indexes: Not recognizing that large IN clauses often lead to poor plan choices.
Not Using Temporary Tables or Arrays: Failing to leverage functions like UNNEST or creating temporary tables for better performance.
Overlooking Optimizer Statistics: Ignoring the importance of accurate statistics and schema changes.

**Comparison:**

| Query Type | Steps |
|------------|-------|
| VALUES Join | 1. Create a temporary table with values<br>2. Use JOIN to match statuses |
| UNNEST Array | 1. Create an array of statuses<br>2. Use UNNEST and ANY for comparison |
| Temp Table  | 1. Create and populate temp table with values<br>2. Perform JOIN operation |

**Code:**

```sql
CREATE TEMPORARY TABLE temp_statuses AS
SELECT 'value1' AS status UNION ALL SELECT 'value2' AS status ... UNION ALL SELECT 'value8000' AS status;

WITH statuses AS (SELECT get_statuses()::TEXT[])
SELECT * FROM app_data WHERE status = ANY (UNNEST(statuses));
```

**Example input / output:**

```text
Original Query:
SELECT * FROM app_data WHERE status IN ('value1', 'value2', ..., 'value8000');

Re-written Queries:
VALUES Join:
SELECT * FROM app_data WHERE EXISTS (SELECT 1 FROM temp_statuses WHERE temp_statuses.status = app_data.status);

UNNEST Array:
WITH statuses AS (SELECT get_statuses()::TEXT[])
SELECT * FROM app_data WHERE status = ANY (UNNEST(statuses));

Temp Table:
SELECT * FROM app_data
JOIN temp_statuses ON app_data.status = temp_statuses.status;
```

**Data:**

```sql
-- Example of creating and populating a temp table:
CREATE TEMPORARY TABLE temp_statuses (status TEXT);
INSERT INTO temp_statuses VALUES ('value1'), ('value2'), ..., ('value8000');
```

**Flow:**

1. Create Temporary Table
2. Populate With Values
3. Use EXISTS to Match Statuses
4. Use UNNEST and ANY forComparison
5. Perform JOIN Operation withTemp Table

