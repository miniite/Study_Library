---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to swap the values of two columns in every row?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To swap the values of two columns in every row, we need to perform an update operation that dynamically swaps the values based on their column names. This can be achieved by using a SQL query with a CASE statement or temporary aliases.

#### Triaging the Question

The question is asking how to write a PostgreSQL query to swap the values between two specific columns for each row in a table. The interviewer might be probing your understanding of SQL syntax, case statements, and dynamic operations within queries. We need to make some assumptions:

The column names are known.
The table name is also known.
There are no constraints or triggers that would prevent an update.

#### First Principles Solution

We can use a CASE statement in the SET clause of an UPDATE query to swap the values between two columns. Here’s how we can do it:

SQL
UPDATE your_table_name
SET column1 = CASE
                WHEN column1 IS NOT DISTINCT FROM column2 THEN column1  -- keep original value if equal
                ELSE column2
              END,
    column2 = CASE
                WHEN column1 IS NOT DISTINCT FROM column2 THEN column2  -- keep original value if equal
                ELSE column1
              END;
Explanation:
The CASE statement is used within the SET clause of an UPDATE query.
We check if column1 and column2 are not distinct using IS NOT DISTINCT FROM.
If they are not distinct, we keep the original value. Otherwise, we swap them.

#### Closing Summary

This approach ensures that only rows where the columns have different values will be updated. Rows with identical column values in both columns before the operation remain unchanged.

#### Common Mistakes/Follow-Ups
Forgetting to Handle NULL Values: Ensure you handle NULL values appropriately, as they can lead to unexpected behavior.

#### Performance Considerations
Check if the table is large and consider indexes or other optimizations if necessary.

#### Cons
traints or Triggers: Ensure there are no constraints or triggers that would interfere with the update.

**Code:**

```sql
UPDATE your_table_name
SET column1 = CASE
                WHEN column1 IS NOT DISTINCT FROM column2 THEN column1
                ELSE column2
              END,
    column2 = CASE
                WHEN column1 IS NOT DISTINCT FROM column2 THEN column2
                ELSE column1
              END;
```

