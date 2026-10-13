---
type: "practice"
difficulty: "Deep"
---

# Explain how TRUNCATE versus ROUND interact with negative numbers and how a mismatch caused an off-by-one in a billing calculation.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TRUNCATE and ROUND are both SQL functions used to modify numerical values, but they operate quite differently when dealing with negative numbers. TRUNCATE removes digits after a certain point without rounding, whereas ROUND rounds a number to the nearest integer or specified decimal place. A mismatch between these two functions can lead to unexpected results, especially in critical calculations like billing.

The core issue lies in how each function handles negative numbers and how a difference in their behavior can cause an off-by-one error. For instance, when you truncate -3.6 using TRUNCATE, it becomes -3, whereas rounding the same number with ROUND gives you -4. This discrepancy can be subtle but significant in financial calculations where accuracy is paramount.

To understand this interaction more deeply, let's break down how each function works and then see an example of a potential off-by-one error.

#### Triage

The question asks us to explain the interaction between TRUNCATE and ROUND with negative numbers, focusing on why a mismatch might lead to an off-by-one error in a billing calculation. The interviewer is probing your understanding of SQL functions and their behavior across different numerical values, particularly negatives.

#### First-Principles Solution

Let's start by examining how both functions work with negative numbers:

TRUNCATE Function:

TRUNCATE(n, d) truncates the number n to d decimal places without rounding.
For example, TRUNC(-3.6) results in -3, as it simply cuts off any digits after the decimal point.

ROUND Function:

ROUND(n, d) rounds the number n to d decimal places according to standard rules.
For instance, ROUND(-3.6) would result in -4 because -3.6 is closer to -4 than it is to -3.

Now let's consider a practical example where these functions might be used in a billing system:

SQL
-- Example of TRUNCATE and ROUND with negative numbers
SELECT
    TRUNC(-3.6) AS truncated_value,  -- This will output -3
    ROUND(-3.6) AS rounded_value;    -- This will output -4
Practical Scenario: Billing Calculation

Imagine a scenario where you are calculating the total cost of items in a store, and each item has a price that needs to be adjusted based on quantity and possibly tax rates.

Let's say we have an item with a base price of -3.6 (this could represent a discount or negative pricing due to returns). The quantity is 2, and there's no need for rounding in this case:

SQL
-- Correct calculation using TRUNCATE without further adjustments
SELECT
    -3.6 * 2 AS total_cost;           -- This results in -7.2

However, if we introduce a tax rate or another factor that requires rounding, the issue might arise:

SQL
-- Incorrect calculation with TRUNCATE followed by ROUND
SELECT
    (TRUNC(-3.6) + 0.1) * 2 AS total_cost; -- This results in -5.8 due to TRUNC(-3.6) = -3 and then rounded to -4

-- Correct calculation using ROUND directly
SELECT
    ROUND((-3.6 + 0.1) * 2) AS total_cost; -- This correctly results in -7, avoiding the off-by-one error

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, TRUNCATE and ROUND handle negative numbers differently, leading to potential discrepancies that can cause off-by-one errors in calculations. In financial systems, it's crucial to use the appropriate function based on the desired outcome.

#### Common Mistakes

Forgetting about the difference between truncation and rounding.
Assuming both functions will behave identically with negative inputs.

#### Follow-Ups

How would this affect performance or memory usage in a large dataset?
Are there other SQL functions you might consider using for financial calculations to avoid such issues?

**Code:**

```sql
-- Example of TRUNCATE and ROUND with negative numbers
SELECT
    TRUNC(-3.6) AS truncated_value,  -- This will output -3
    ROUND(-3.6) AS rounded_value;    -- This will output -4
```

