---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a deterministic and a non-deterministic function? Give an example of each.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Scalar vs Table-valued Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Scalar_vs_Table-valued_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, especially with SQL, it's crucial to understand the distinction between deterministic and non-deterministic functions. A deterministic function always returns the same result when given a specific set of input values, whereas a non-deterministic function may return different results even if provided with identical input parameters.

Let’s explore each type in detail:

Deterministic Functions: These functions are predictable and consistent. They produce the same output every time they are called with the same inputs.
Non-Deterministic Functions: These functions can return a different result for the same set of input values, making their behavior less predictable.

#### Triage

The question is asking us to define deterministic and non-deterministic SQL functions and provide examples of each. The interviewer may be probing our understanding of function properties and how they behave in SQL environments.

#### First-Principles Solution
Deterministic Functions

Deterministic functions are straightforward as they always return the same result for a given set of input values. Examples include built-in arithmetic operations like SUM, AVG, or even custom functions that use only data from their parameters and do not rely on any external factors.

For instance, consider a simple SQL function:

SQL
CREATE FUNCTION dbo.CalculateTotalSales(@SalesData TABLE (Amount DECIMAL(10,2)))
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @Total DECIMAL(10,2) = 0.0;

    SELECT @Total += Amount FROM @SalesData;

    RETURN @Total;
END;

This function CalculateTotalSales is deterministic because the total sum will always be the same given the same set of sales data.

Non-Deterministic Functions

Non-deterministic functions, however, can return different results even with the same input values. SQL Server has a few built-in non-deterministic functions like NEWID(), which generates a new unique identifier every time it is called.

An example would be:

SQL
CREATE FUNCTION dbo.GenerateRandomNumber(@Seed INT)
RETURNS TABLE
AS
RETURN (
    SELECT NEWID(), @Seed + (ABS(CHECKSUM(NewId())) % 100) AS RandomNumber
);

Here, NEWID() creates a new unique identifier each time it is called, making this function non-deterministic. Even with the same seed value, the result will vary.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, deterministic functions ensure predictability and consistency in their results, while non-deterministic functions allow for variability due to external factors or randomization within SQL. The key takeaway is understanding how these differences impact query performance, indexing strategies, and overall database architecture.

#### Common mistakes
include:

Confusing deterministic with non-deterministic functions.
Overlooking the implications of using non-deterministic functions in production environments where predictability is critical.

#### Follow-ups
might involve discussing how to handle these functions in different contexts, such as during data warehousing or real-time analytics scenarios.

**Code:**

```sql
-- Example of deterministic function
CREATE FUNCTION dbo.CalculateTotalSales(@SalesData TABLE (Amount DECIMAL(10,2)))
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @Total DECIMAL(10,2) = 0.0;

    SELECT @Total += Amount FROM @SalesData;

    RETURN @Total;
END;

-- Example of non-deterministic function
CREATE FUNCTION dbo.GenerateRandomNumber(@Seed INT)
RETURNS TABLE
AS
RETURN (
    SELECT NEWID(), @Seed + (ABS(CHECKSUM(NewId())) % 100) AS RandomNumber
);
```

