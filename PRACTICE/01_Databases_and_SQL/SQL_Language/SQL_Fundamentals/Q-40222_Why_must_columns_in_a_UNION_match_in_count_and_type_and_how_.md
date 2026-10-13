---
type: "practice"
difficulty: "Intermediate"
---

# Why must columns in a UNION match in count and type, and how do you handle a column that only one side has?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, the UNION operation combines the result sets of two or more SELECT statements into a single result set where each row is unique. The core requirement for columns in a UNION clause is that they must have the same number and type across all queries involved. This ensures that the resulting dataset maintains consistency and can be meaningfully combined.

#### Triaging the Question

The question asks about why column counts and types must match in a UNION, particularly when one side of the query has fewer columns than the other. The key points being probed are:

#### Cons
istency: Ensuring that the resulting dataset is well-formed.
Type Safety: Maintaining data integrity by ensuring compatible data types.
Handling Mismatched Columns: Strategies to deal with columns present in one query but not the other.

#### Assumptions
All queries involved in a UNION must have an equal number of columns.
The column order and names do not need to match, only their count and data types should be consistent.
Step-by-Step Solution

Let's start from first principles. A UNION operation combines the results of two or more SELECT statements into a single result set where duplicate rows are removed (using an implicit DISTINCT).

Why Column Counts and Types Must Match

#### Cons
istency: If columns did not match, it would be impossible to ensure that each row in the final output is valid. For example, if one query returns two columns while another returns three, combining them without proper handling leads to ambiguity.
Type Safety: Ensuring that data types are compatible prevents SQL errors and ensures that operations on the combined dataset can proceed smoothly.
Handling a Column That Only One Side Has

When you have mismatched column counts:

The common practice is to add NULL columns or use a UNION ALL which does not enforce uniqueness.
Alternatively, you can rename or alias columns in one of the queries to match the other.
Detailed Example

#### Cons
ider two SELECT statements:

SQL
-- Query 1: Two columns
SELECT column1, column2 FROM table1;

-- Query 2: Three columns
SELECT column3, column4, column5 FROM table2;

To use UNION, we need to make sure both queries return the same number of columns. One approach is to add a NULL column in the first query:

SQL
SELECT column1, column2, NULL AS dummy_column FROM table1
UNION
SELECT column3, column4, column5 FROM table2;

Or, we could use UNION ALL, which does not enforce uniqueness and can handle mismatched columns directly.

#### Closing Summary

In summary, ensuring that the columns in a UNION match in count and type is crucial for maintaining data integrity and avoiding SQL errors. When dealing with mismatched columns, you can add NULL placeholders or use UNION ALL.

#### Common Mistakes/Follow-Ups
Ignoring Column Mismatch: Failing to address column differences can lead to runtime errors.
Using Incorrect Data Types: Mixing incompatible data types in the same UNION can result in type coercion issues.
Not Understanding UNION ALL vs. UNION: Using UNION ALL without knowing its implications (including duplicates) can affect query performance and results.

**Code:**

```sql
-- Example with NULL placeholder
 SELECT column1, column2, NULL AS dummy_column FROM table1
 UNION
 SELECT column3, column4, column5 FROM table2;

 -- Example using UNION ALL
 SELECT column1, column2 FROM table1
 UNION ALL
 SELECT column3, column4 FROM table2;
```

