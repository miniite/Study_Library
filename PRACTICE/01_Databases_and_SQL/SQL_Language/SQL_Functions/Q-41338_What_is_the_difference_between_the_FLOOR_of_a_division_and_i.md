---
type: "practice"
difficulty: "Deep"
---

# What is the difference between the FLOOR of a division and integer division in SQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, both FLOOR and integer division are used to handle numeric operations but serve different purposes. The core idea here is understanding how each handles rounding and truncation.

Firstly, let's clarify what we're dealing with:

Integer Division: This operation divides two numbers and rounds the result towards zero (also known as truncating). For example, 10 / 3 results in 3.
FLOOR Function: This function rounds a number down to the nearest integer. The key point is that it always rounds down regardless of the decimal part.

Now, let's dive into the differences with a step-by-step explanation:

#### Triage
What’s Being Probed

The question is fundamentally probing your understanding of:

Different rounding behaviors: Understanding how FLOOR works compared to integer division.
Use cases and implications: How these operations can affect your queries, especially in scenarios involving negative numbers or non-integer results.

#### Assumptions
and Reasoning from First Principles

Let's assume we are working with a simple SQL database environment where both operations could be used interchangeably but yield different results due to their inherent differences.

#### Cons
ider the following example:

SQL
SELECT FLOOR(10.8), 10 / 3, FLOOR(-10.8), -10 / 3;
FLOOR(10.8): This will return 10 because it rounds down to the nearest integer.
10 / 3: This results in 3 as it truncates towards zero.

Now, for negative numbers:

FLOOR(-10.8): This returns -11 since it always rounds down.
-10 / 3: This also results in -3, again due to integer division’s truncation behavior.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, FLOOR provides a consistent rounding method that always moves towards negative infinity for positive numbers and towards zero for negative numbers. Integer division, on the other hand, simply truncates towards zero.

#### Common Mistakes
Misunderstanding Truncation: Sometimes developers confuse integer division with simple subtraction of the decimal part.
Failing to Account for Negative Numbers: The behavior can be counterintuitive if you're not aware that FLOOR rounds down, while integer division truncates.

#### Follow-Ups
How do these differences affect query performance or result accuracy in complex operations?
In what scenarios might one choose over the other?

**Code:**

```sql
-- Example SQL Statements
SELECT FLOOR(10.8), 10 / 3, FLOOR(-10.8), -10 / 3;
```

