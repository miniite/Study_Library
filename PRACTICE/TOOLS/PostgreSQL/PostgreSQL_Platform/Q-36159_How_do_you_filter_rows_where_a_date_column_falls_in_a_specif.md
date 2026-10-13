---
type: "practice"
difficulty: "Basic"
---

# How do you filter rows where a date column falls in a specific year?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To filter rows in PostgreSQL based on a specific year from a date column, we need to extract the year part of the date and compare it with our desired year. This involves using SQL functions like EXTRACT or DATE_TRUNC to isolate the year component and then applying a WHERE clause to match our criteria.

First, let's understand what the question is asking: We want to retrieve records where the year in a specific date column matches a given year. The key idea here is to manipulate the date to get its year part and compare it with the desired value.

We will assume you have a table named employees with a column join_date, which contains the dates when employees joined the company. We need to filter this table for employees who joined in 2015.

