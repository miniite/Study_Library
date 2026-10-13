---
type: "practice"
difficulty: "Basic"
---

# How does the MOD function (or % operator) work and what does it compute?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The MOD function, also known as the modulo operation or the % operator in SQL, computes the remainder when one number is divided by another. Essentially, it tells us how much "left over" there is after dividing a number (the dividend) by another number (the divisor). This function is incredibly useful for tasks such as determining if a number is even or odd, calculating cycles, and filtering data.

Let's break down the question from first principles:

What does it compute? The MOD function calculates the remainder of a division operation.
How does it work? It takes two inputs: the dividend (the number to be divided) and the divisor (the number by which we divide). If you use the % operator, it works similarly but can be slightly different in some SQL dialects depending on how it's implemented.

Now let's look at a practical example:

SQL
SELECT 15 % 4;

In this case, 15 is the dividend and 4 is the divisor. The result of the operation would be 3, because when you divide 15 by 4, you get 3 as the quotient with a remainder of 3.

Triaging
What's being asked? We need to explain how the modulo operation works and what it computes.

#### Assumptions
Typically, we assume that both inputs are integers. In some cases, floating-point numbers can also be used, but they might not always return accurate results due to precision issues.

#### First Principles Solution

Understand Division: Start by understanding the concept of division in mathematics. When you divide a number a by another number b, there are three components:

Quotient: The result of how many times b can fully fit into a.
Remainder: The leftover part that is less than b.

Implement the Operation: In SQL, you can use the % operator to get this remainder.

#### Example
Dry Run:

SQL
SELECT 17 % 5;

Here, 17 divided by 5 gives a quotient of 3 with a remainder of 2. Thus, the result is 2.

Negative Numbers: It's important to note that when using negative numbers, different SQL dialects handle them differently. Some return positive remainders, while others return negative ones.

SQL
SELECT -17 % 5; -- Different results in different databases: MySQL returns -2, PostgreSQL returns 3

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Forgetting that the result can vary based on the SQL dialect. Always test in your specific environment.

#### Follow-Up
Questions: Interviewers might ask about edge cases or operations with floating-point numbers, or how to use this function for more complex logic like filtering rows.

In summary, the MOD function calculates the remainder of a division operation, which is useful for various data processing tasks. Understanding its behavior and knowing how different SQL databases handle it can help in writing robust queries.

**Code:**

```sql
SELECT 15 % 4; -- Returns 3
SELECT -17 % 5; -- Result may vary between -2 or 3, depending on the database
```

