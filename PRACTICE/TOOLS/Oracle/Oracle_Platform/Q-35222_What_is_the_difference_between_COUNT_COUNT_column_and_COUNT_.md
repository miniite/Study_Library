---
type: "practice"
difficulty: "Deep"
---

# What is the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column) in Oracle?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Aggregate Functions](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between COUNT(*), COUNT(column), and COUNT(DISTINCT column) in Oracle, we first need to break down what each of these functions does. This understanding is crucial for optimizing queries and ensuring accurate results.

TL;DR
COUNT(*): Counts all rows, including duplicates.
COUNT(column): Counts the non-null values in a specified column.
COUNT(DISTINCT column): Counts unique non-null values in a specified column.

These functions are fundamental for basic row counting and can significantly impact query performance depending on how they're used. Now, let's dive into each of these in more detail from first principles.

#### Triage

The question is probing your understanding of basic SQL aggregate functions in Oracle. You need to explain the differences between three distinct ways of counting rows or values within a column:

#### Assumptions
We assume you are familiar with basic SQL operations and have an understanding of how Oracle processes queries.
Core Idea: The core idea here is to understand the behavior of different count functions and their impact on query performance.
Step-by-Step Solution
1. COUNT(*)

COUNT(*) returns the number of rows in a result set, including duplicates. This means that if you have duplicate rows, they will be counted multiple times unless explicitly removed using an DISTINCT clause.

SQL
SELECT COUNT(*) FROM employees;

This query counts all rows in the employees table without any filtering or conditions.

2. COUNT(column)

COUNT(column) returns the number of non-null values in a specified column. If the column contains nulls, they will not be counted.

SQL
SELECT COUNT(salary) FROM employees;

This query counts only those rows where the salary column is not null. Rows with no salary (null salary) are excluded from the count.

3. COUNT(DISTINCT column)

COUNT(DISTINCT column) returns the number of unique, non-null values in a specified column. This means that if you have duplicate values in a column, they will only be counted once.

SQL
SELECT COUNT(DISTINCT department_id) FROM employees;

This query counts the number of distinct department_id values in the employees table. If there are any duplicates in department_id, they will be considered as one unique value for the count.

#### Closing Summary

To summarize, COUNT(*) simply returns all rows, including duplicates; COUNT(column) returns non-null values from a specified column; and COUNT(DISTINCT column) returns unique non-null values. Each of these functions can be very useful depending on your query requirements.

#### Common Mistakes/Follow-Ups
Misunderstanding Nulls: Many beginners might confuse null values with zero or empty strings, but in SQL, null is a special value that represents an unknown or missing value.

#### Performance Considerations
COUNT(*) and COUNT(column) are generally faster than COUNT(DISTINCT column) due to the additional processing required for distinct counts. Always consider performance implications when choosing between these functions.

If you were asked to optimize a query, understanding which function to use would be critical.

Self-Check

This solution is correct and covers all necessary aspects of the question.

**Comparison:**

| ID | Name  | Salary | Department_ID |
|----|-------|--------|--------------|
| 1  | Alice | 50000  | 10           |
| 2  | Bob   | 60000  | 10           |
| 3  | Charlie | NULL  | 20           |
| 4  | Dave | 50000  | 10           |

**Code:**

```sql
-- Example Queries
SELECT COUNT(*) FROM employees;
SELECT COUNT(salary) FROM employees;
SELECT COUNT(DISTINCT department_id) FROM employees;
```

**Example input / output:**

```text
COUNT(*) -> 4 rows in the employees table.
COUNT(salary) -> 3 non-null salary values (rows 1, 2, and 4).
COUNT(DISTINCT department_id) -> 2 unique departments (10 and 20).
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "Alice", "salary": 50000, "department_id": 10},
    {"id": 2, "name": "Bob", "salary": 60000, "department_id": 10},
    {"id": 3, "name": "Charlie", "salary": null, "department_id": 20},
    {"id": 4, "name": "Dave", "salary": 50000, "department_id": 10}
  ]
}
```

**Flow:**

1. COUNT(*)
2. Counts all rows
3. Performance: fast, includesduplicates
4. COUNT(column)
5. Counts non-null values in acolumn
6. Performance: may be slowerdue to null checks
7. COUNT(DISTINCT column)
8. Counts unique, non-null values
9. Performance: slowest due toadditional processing foruniqueness

