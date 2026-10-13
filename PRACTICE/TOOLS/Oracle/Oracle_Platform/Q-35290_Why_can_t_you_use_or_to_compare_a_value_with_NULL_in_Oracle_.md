---
type: "practice"
difficulty: "Intermediate"
---

# Why can't you use = or != to compare a value with NULL in Oracle and what should you use instead?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Data Types](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Data_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Oracle, using = or != to compare a value with NULL is problematic because NULL represents an unknown value, not a specific nullity condition. To accurately handle comparisons involving NULL, you should use the IS NULL and IS NOT NULL operators.

#### Triage

The question asks why Oracle doesn't allow direct comparison using = or != with NULL. It also seeks what alternative operators to use for such comparisons. The key here is understanding how NULL is handled in SQL, particularly the nuances of comparison operations involving it.

Step-by-Step Solution from First Principles
Understanding NULL: In relational databases, NULL is a special value used to represent an unknown or missing data. Unlike other values like integers, strings, etc., NULL does not have any inherent value; it signifies absence of information.
Comparison Operations with NULL:
Using = (equal) or != (not equal): These operators compare two operands for equality or inequality. When you use these operations with NULL, the result is always false because there's no specific value to equate or not equate. However, this behavior can be misleading.
IS NULL and IS NOT NULL:
To accurately check if a column has a NULL value, you should use IS NULL. For example: SELECT * FROM table WHERE column IS NULL.
Similarly, to find rows where the column is not NULL, use IS NOT NULL: SELECT * FROM table WHERE column IS NOT NULL.

#### Closing Summary

In Oracle and most SQL databases, direct comparison using = or != with NULL will always return false. Instead, you should use the IS NULL and IS NOT NULL operators to handle comparisons involving NULL. This approach ensures that your queries correctly identify rows where a column has no value.

#### Common Mistakes/Follow-Ups
Mistake: Using = or != with NULL.

#### Example
of Correct Usage: Instead of SELECT * FROM table WHERE column = NULL, use SELECT * FROM table WHERE column IS NULL.

#### Follow-Up
Question: In what scenarios might you want to check for NULL values in a query?
Answer: Common scenarios include filtering out rows where data is missing or ensuring completeness before performing operations.

