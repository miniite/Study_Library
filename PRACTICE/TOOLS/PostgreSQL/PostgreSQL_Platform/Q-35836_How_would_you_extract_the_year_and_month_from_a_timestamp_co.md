---
type: "practice"
difficulty: "Intermediate"
---

# How would you extract the year and month from a timestamp column using EXTRACT or DATE_TRUNC?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract the year and month from a timestamp column in PostgreSQL, we can use either EXTRACT or DATE_TRUNC. These functions allow us to break down the timestamp into its component parts such as years, months, days, etc., making it easier to perform calculations or filtering on specific date components.

First, let's understand what these functions do:

EXTRACT: This function extracts a specified part of a timestamp. For example, you can use EXTRACT(YEAR FROM ...) to get the year and EXTRACT(MONTH FROM ...) for the month.

DATE_TRUNC: This function truncates a date or timestamp to a specified unit (year, month, day) and returns it as a date. Using this, we can effectively isolate the year and month from a full timestamp.

#### Triage

The question is asking how to extract specific date components (year and month) from a timestamp column using PostgreSQL functions. The interviewer might be probing your understanding of date manipulation in SQL databases and your ability to use built-in functions correctly. Assumptions here include familiarity with common SQL functions and the syntax for working with timestamp data types.

#### First-Principles Solution

Let's start by examining a simple example:

Assume we have a table orders with a timestamp column called order_time.

Using EXTRACT:
SQL
SELECT
    order_id,
    EXTRACT(YEAR FROM order_time) AS order_year,
    EXTRACT(MONTH FROM order_time) AS order_month
FROM orders;

In this query, we use the EXTRACT function to get the year and month from the order_time column. The result will be a list of order IDs with their respective years and months.

Using DATE_TRUNC:
SQL
SELECT
    order_id,
    DATE_TRUNC('year', order_time) AS order_year,
    DATE_TRUNC('month', order_time) AS order_month
FROM orders;

Here, DATE_TRUNC is used to truncate the timestamp to the year level (or month level), which effectively gives us the year and month components. The result will be a list of order IDs with their respective years and months.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Both methods work well depending on your specific use case:

EXTRACT is more precise as it returns integer values for year and month.
DATE_TRUNC can also provide the desired result but might not be as straightforward if you need to combine the results with other date calculations.

#### Common mistakes
include:

Not understanding the difference between EXTRACT and DATE_TRUNC.
Forgetting to cast the output of DATE_TRUNC to a numeric type when performing arithmetic operations.
Using the wrong function name or syntax, which can lead to errors.

#### Follow-ups
might involve discussing edge cases, such as handling time zones, or performance considerations if working with large datasets.

Self-Check

This solution covers both methods and addresses common mistakes, ensuring completeness and correctness.

