---
type: "practice"
difficulty: "Basic"
---

# What is the BETWEEN operator in Oracle and is it inclusive or exclusive of its boundaries?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle SQL and Indexes](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_SQL_and_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The BETWEEN operator in Oracle SQL is used to select a range of values, ensuring that queries can be written more concisely. The key aspect here is understanding whether the boundaries are inclusive or exclusive.

To answer the question directly, the BETWEEN operator in Oracle is inclusive of its boundaries. This means that if you use BETWEEN 10 AND 20, it will include both 10 and 20 in the result set.

Here’s a step-by-step explanation from first principles:

#### Triage
The question asks about the behavior of the BETWEEN operator, specifically whether its boundaries are inclusive or exclusive.
The core concept involves understanding how SQL ranges work with the BETWEEN operator and its syntax in Oracle.

#### First Principles Solution

The BETWEEN operator in SQL is designed to include both boundary values. This means that if you use a query like:

SQL
SELECT * FROM employees WHERE salary BETWEEN 1000 AND 2000;

it will return all records where the salary is either exactly 1000, between 1000 and 2000 (excluding 2000), or exactly 2000.

To reason through this, consider a simple example:

Suppose you have an employees table with salary values ranging from 950 to 2100.
Using BETWEEN 1000 AND 2000 will return all salaries that are at least 1000 and no more than 2000.

#### Common Mistakes
and Follow-Ups
Exclusive Boundaries: A common mistake is assuming the boundaries are exclusive, as in many other programming languages where BETWEEN works differently (e.g., Python's slicing).
Edge Cases: Ensure you understand how it behaves at the very edges of a range. For example:
SQL
SELECT * FROM employees WHERE salary BETWEEN 1000 AND NULL; -- This is valid, and will include all records with a non-null salary.
Inclusive vs Exclusive Range: Another follow-up could be to compare BETWEEN with other range operators like >= and < in Oracle, which are more explicit about exclusivity:
SQL
SELECT * FROM employees WHERE salary >= 1000 AND salary < 2000; -- This would exclude the boundary values.

#### Closing Summary

The BETWEEN operator in Oracle SQL is inclusive of its boundaries. It means that when using a query like BETWEEN X AND Y, both X and Y are included in the results.

**Code:**

```sql
-- Example usage of BETWEEN
SELECT * FROM employees WHERE salary BETWEEN 1000 AND 2000;
```

