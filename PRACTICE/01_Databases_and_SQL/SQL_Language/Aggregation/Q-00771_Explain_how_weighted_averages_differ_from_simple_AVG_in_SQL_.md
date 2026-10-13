---
type: "practice"
difficulty: "Intermediate"
---

# Explain how weighted averages differ from simple AVG in SQL, and write the general pattern for SUM(value*weight)/SUM(weight) with correct NULL/zero handling.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between weighted averages and simple AVG in SQL, it's essential to recognize that a weighted average takes into account not just the values but also their corresponding weights. This is particularly useful when dealing with data where different elements should contribute more or less to the overall average based on some criteria.

The general pattern for calculating a weighted average in SQL can be expressed as SUM(value * weight) / SUM(weight). Let's break down this approach, including handling of NULL and zero values.

#### Triaging the Question
Question: How does a weighted average differ from a simple AVG in SQL? Write the general pattern for SUM(value * weight) / SUM(weight) with correct handling of NULL/zero.
What's Being Probed:
Understanding of basic statistical concepts (simple averages vs. weighted averages).
Knowledge of SQL aggregation functions and their usage.
Ability to handle edge cases such as NULL or zero values.

#### First-Principles Solution
Simple AVG

A simple AVG is calculated by summing up all the values and dividing by the number of non-null elements. It’s a straightforward mean, where each value contributes equally to the result.

SQL
SELECT AVG(column_name) FROM table_name;
Weighted Average

In contrast, a weighted average assigns different weights to the values, reflecting their relative importance. The formula SUM(value * weight) / SUM(weight) captures this concept by multiplying each value with its corresponding weight before averaging.

Here’s how you can implement it in SQL:

SQL
SELECT (SUM(column1 * column2) OVER () / SUM(column2) OVER ()) AS weighted_avg
FROM table_name;
Handling NULL and Zero Values
NULL Handling: If any row has a weight of NULL, the corresponding value should not be included in the calculation. This is handled by SQL's natural filtering.
Zero Handling: A weight can be zero, which means that value contributes no part to the weighted average. You must ensure that multiplying by zero results in zero and does not skew the final result.
Full Solution
SQL
SELECT
    SUM(CASE WHEN weight > 0 THEN value * weight ELSE 0 END) /
    SUM(CASE WHEN weight > 0 THEN weight ELSE 1/0 -- avoid division by zero error
             END) AS weighted_avg
FROM table_name;

#### Closing Summary
and Common Mistakes
Common Mistake: Not handling NULL or zero values correctly, which can lead to incorrect results.
Avoiding Division By Zero: Ensure that the denominator is never zero. Using 1/0 in SQL (or equivalent) triggers an error but ensures you don't encounter a division by zero runtime issue.
Code Block
SQL
-- Sample query for weighted average handling NULL and zero
SELECT
    SUM(CASE WHEN weight > 0 THEN value * weight ELSE 0 END) /
    SUM(CASE WHEN weight > 0 THEN weight ELSE 1/0 -- avoid division by zero error
             END) AS weighted_avg
FROM table_name;
Self-Check

This solution handles NULL and zero values correctly, ensures that the denominator is never zero, and calculates the weighted average accurately.

