---
type: "practice"
difficulty: "Intermediate"
---

# What is integer division in SQL and why might SELECT 5/2 return 2 instead of 2.5?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Integer division in SQL refers to a type of arithmetic operation where both operands are integers, and the result is also an integer. The primary characteristic of integer division is that it truncates any fractional part of the result, effectively rounding down towards zero.

To understand why SELECT 5/2 returns 2 instead of 2.5, we need to delve into how SQL handles numeric types and arithmetic operations. This behavior stems from the fact that integer division discards the decimal portion, providing a whole number result.

#### Triage
Restating the Question

The question is asking about the concept of integer division in SQL and why it behaves differently compared to standard floating-point division. Specifically, we need to explain why an operation like SELECT 5/2 results in 2 rather than 2.5.

#### Assumptions
and Context
We are dealing with a scenario where both operands (5 and 2) are integers.
The SQL dialect being used supports integer division, which is common but not universal across all databases (e.g., MySQL, PostgreSQL).

#### First Principles Solution
Step-by-Step Reasoning

Integer Division Definition: Integer division in SQL involves dividing two integers and returning the result as an integer by truncating any fractional part.

#### Example
Calculation:

When you perform 5 / 2, both numbers are integers.
The mathematical result of this operation is 2.5.
However, since the result needs to be an integer in SQL's context of integer division, it gets truncated down to 2.

Alternative Example: If we were using floating-point division (SELECT 5.0 / 2), the result would naturally be a float, and you'd get 2.5.

Database-Specific Behavior:

Different SQL databases might have slightly different behaviors for integer division. For instance, in PostgreSQL, the behavior is consistent with our example.
In MySQL, an implicit conversion might occur if one operand is a decimal or float.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Assuming that SELECT 5/2 will always yield 2.5. The key takeaway is to understand the behavior of integer division in SQL.

#### Follow-Up
Question: How can one ensure they get a floating-point result? One way is by casting one or both operands as floats using SELECT CAST(5 AS REAL) / 2.
Practical Example
SQL
-- Integer Division
SELECT 5 / 2;  -- Result: 2

-- Floating-Point Division
SELECT 5.0 / 2;  -- Result: 2.5

**Code:**

```sql
-- Integer Division
SELECT 5 / 2;

-- Floating-Point Division
SELECT 5.0 / 2;
```

