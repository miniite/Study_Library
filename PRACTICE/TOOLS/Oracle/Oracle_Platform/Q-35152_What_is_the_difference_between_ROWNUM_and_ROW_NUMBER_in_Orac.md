---
type: "practice"
difficulty: "Deep"
---

# What is the difference between ROWNUM and ROW_NUMBER() in Oracle?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Aggregate Functions](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL queries, especially those involving sorting or pagination, understanding the differences between ROWNUM and ROW_NUMBER() functions in Oracle databases is crucial. While both can be used to limit the number of rows returned by a query, they operate differently under the hood. In this answer, I'll explain these differences from first principles and provide examples to illustrate their usage.

#### Triage
Understanding ROWNUM and ROW_NUMBER()

Question Summary: The core question here is to understand the functional and operational differences between ROWNUM and ROW_NUMBER() in Oracle databases.

Key Assumptions:

We are dealing with SQL queries that require row limitation or pagination.
Both functions can be used to return a subset of rows from a query result set, but their mechanisms differ.

#### First Principles Solution
ROWNUM

ROWNUM is a pseudocolumn in Oracle that assigns a unique sequential integer (starting from 1) to each row returned by a query. The assignment happens as the rows are retrieved and before any sorting or filtering occurs. Therefore, ROWNUM might not always return the intended results if you use it for pagination or sorting.

Here's an example:

SQL
SELECT * FROM (
    SELECT ROWNUM AS rn, emp_id, name FROM employees ORDER BY salary DESC
)
WHERE rn > 10 AND rn < 20;

In this query, ROWNUM is assigned as the rows are fetched from the base query. This means that if you sort by salary and then apply a filter on ROWNUM, it might not return exactly 10 to 20 employees in terms of their original order.

ROW_NUMBER()

On the other hand, ROW_NUMBER() is an analytic function that assigns a unique integer value starting at 1 within each partition. It ensures that sorting and filtering are applied before row numbering. This makes it more predictable for pagination purposes as it respects the actual sorting or filtering conditions specified in your query.

Here's how you would use ROW_NUMBER():

SQL
SELECT emp_id, name
FROM (
    SELECT emp_id, name, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn FROM employees
)
WHERE rn BETWEEN 10 AND 20;

In this example, the inner query first sorts all rows by salary in descending order. Then ROW_NUMBER() is applied to assign a unique number starting from 1 for each row based on that sorting. Finally, we filter to get the rows where the assigned rn value is between 10 and 20.

#### Closing Summary

In summary, while both ROWNUM and ROW_NUMBER() are used to limit the number of rows returned by a query, they differ in their application and reliability for pagination or sorting. ROWNUM assigns numbers based on row retrieval order, which may not align with your desired sort order, whereas ROW_NUMBER() ensures that numbering is based on specified conditions before any filtering.

#### Common Mistakes/Follow-Ups
Misunderstanding Sorting: Many beginners mistakenly use ROWNUM for pagination without realizing the order in which rows are processed.
Performance Concerns: Using subqueries with large datasets and ROWNUM can be performance-intensive due to its row-by-row processing.
Alternative Solutions: Consider using other analytic functions like RANK() or DENSE_RANK() if you need more complex ranking logic.

**Comparison:**

```
Employee ID	Name
2	Bob
3	Carol
4	Dave
<!-- ROW_NUMBER() example -->
SQL
Copy
-- Input for ROW_NUMBER() example
SELECT emp_id, name
FROM (
    SELECT emp_id, name, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn FROM employees
)
WHERE rn BETWEEN 10 AND 20;

-- Output for ROW_NUMBER() example (may vary based on actual data and sorting)
```

**Code:**

```sql
-- Example of ROWNUM usage
SELECT * FROM (
    SELECT ROWNUM AS rn, emp_id, name FROM employees ORDER BY salary DESC
)
WHERE rn > 10 AND rn < 20;

-- Example of ROW_NUMBER() usage
SELECT emp_id, name
FROM (
    SELECT emp_id, name, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn FROM employees
)
WHERE rn BETWEEN 10 AND 20;
```

**Example input / output:**

```text
-- Input for ROWNUM example
SELECT * FROM (
    SELECT ROWNUM AS rn, emp_id, name FROM employees ORDER BY salary DESC
)
WHERE rn > 10 AND rn < 20;

-- Output for ROWNUM example (may vary based on actual data and sorting)
```

**Data:**

```sql
-- Sample table structure for demonstration
CREATE TABLE employees (
    emp_id NUMBER,
    name VARCHAR2(50),
    salary NUMBER
);

INSERT INTO employees (emp_id, name, salary) VALUES (1, 'Alice', 70000);
INSERT INTO employees (emp_id, name, salary) VALUES (2, 'Bob', 80000);
-- More insert statements...
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Is the query sorted?
9. RN is assigned as rows arefetched
10. Is RN used for pagination?
11. Use of ROWNUM can beunpredictable
12. RN respects sorting andfiltering before numbering
13. Does the query useROW_NUMBER()?
14. RN is based on conditionsapplied first
15. Consider other analyticfunctions if needed forcomplex ranking logic

