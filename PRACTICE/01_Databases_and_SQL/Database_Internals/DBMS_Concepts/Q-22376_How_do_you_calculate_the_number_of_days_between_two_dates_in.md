---
type: "practice"
difficulty: "Intermediate"
---

# How do you calculate the number of days between two dates in SQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the number of days between two dates in SQL, we need to use date functions provided by the database management system (DBMS). The key idea is to subtract one date from another and get the difference as an interval value, which can then be converted into the required format. For a fresher or intermediate experience level, I'll walk through this using common DBMS like MySQL and PostgreSQL.

First, let's consider what the question is asking: we need to find out how many days are between two given dates. The core idea is straightforward—subtract one date from another to get the difference in days. However, there are subtle differences depending on the DBMS being used.

Let’s assume our requirement is as follows:

Input: Two dates (e.g., '2023-10-01' and '2023-10-05').
Output: A number representing the days between these two dates.

We'll start with a basic example in MySQL:

SQL
SELECT DATEDIFF('2023-10-05', '2023-10-01') AS DaysBetweenDates;

In this query, DATEDIFF is used to calculate the difference between two dates and returns an integer representing the number of days.

Next, let’s look at how we can achieve the same in PostgreSQL:

SQL
SELECT age('2023-10-05'::date, '2023-10-01'::date) ->> 'days' AS DaysBetweenDates;

Here, age function is used to compute the difference between two dates and then we extract the days part from the resulting interval.

Both of these queries will return an output like:

PLAINTEXT
DaysBetweenDates
-----------------
4

This result indicates that there are 4 days between '2023-10-05' and '2023-10-01'.

