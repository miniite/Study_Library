---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute a growth-rate column that safely handles a zero or null previous value from LAG without dividing by zero?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a growth rate while safely handling zeros or nulls in SQL using window functions, we need to calculate the percentage change between consecutive rows, but avoid division by zero. Here's how you can approach this problem step-by-step:

Understand the Problem: We are looking for a growth rate (percentage increase) from one row to another, but we must handle cases where the previous value is null or zero.

#### Triage
The interviewer likely wants to see your understanding of SQL window functions and how to gracefully deal with edge cases in data processing.

#### First-Principles Solution

Use the LAG function to get the previous value.
Handle nulls by checking if they are not null before performing division.
Use a conditional check or a common SQL trick (COALESCE) to avoid division by zero.

Step-by-Step Solution:

First, select the necessary columns and apply LAG on the previous value.
Then, use a conditional expression to safely perform the division.

Here’s an example using PostgreSQL syntax:

SQL
SELECT
    id,
    current_value,
    LAG(current_value) OVER (ORDER BY date) AS prev_value,
    CASE
        WHEN LAG(current_value) OVER (ORDER BY date) = 0 THEN NULL -- Avoid division by zero for previous value
        ELSE (current_value - LAG(current_value) OVER (ORDER BY date)) / LAG(current_value) OVER (ORDER BY date)
    END AS growth_rate
FROM
    sales_data;

#### Closing Summary
+ Common Mistakes:
The key idea is to use the LAG function with a conditional check to prevent division by zero.
A common mistake is not handling null values properly, leading to potential errors or incorrect results.

