---
type: "practice"
difficulty: "Basic"
---

# How do you sort a result first by one column ascending and then by another descending?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To sort a result in MySQL, you can use the ORDER BY clause to specify multiple columns with different sorting orders. The query will first sort by the first column in ascending order (ASC) and then by the second column in descending order (DESC). Here’s a breakdown of how this works:

#### Triage

The question is about understanding how to structure a SQL query to sort data using ORDER BY. Specifically, we need to sort based on one column in ascending order followed by another column in descending order. The interviewer might be probing your knowledge of the ORDER BY clause and your ability to combine sorting criteria.

#### First-Principles Solution

To achieve this, you use the ORDER BY clause with multiple columns, specifying the sorting direction for each column. Here's a step-by-step explanation:

Identify the Columns: Determine which two columns need to be sorted.
Specify Sorting Directions: Use ASC for ascending and DESC for descending.
Combine in Order of Preference: Place the first column you want to sort by in ascending order, followed by the second column in descending order.

#### Example
Query

Suppose we have a table named employees with columns last_name, first_name, and salary. We want to sort by last_name (ascending) and then by salary (descending).

SQL
SELECT * FROM employees ORDER BY last_name ASC, salary DESC;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this query:

The last_name column is sorted in ascending order.
The salary column is sorted in descending order.

A common mistake is placing the sorting directions incorrectly or omitting one of them. Always double-check that your ORDER BY clause clearly specifies both columns and their respective directions.

Self-Check

The query structure is correct, and it addresses the sorting requirements effectively.

**Code:**

```sql
SELECT * FROM employees ORDER BY last_name ASC, salary DESC;
```

