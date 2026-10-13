---
type: "practice"
difficulty: "Deep"
---

# Why is PERCENTILE_CONT an ordered-set aggregate using WITHIN GROUP rather than a normal aggregate, and how does that change its syntax?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Window Functions Overview](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why PERCENTILE_CONT in SQL is an ordered-set aggregate with WITHIN GROUP, we need to first grasp the core difference between regular aggregates and cumulative distributions, and how they relate to the syntax of functions like PERCENTILE_CONT. This function calculates a continuous percentile value across a set of rows based on their order, making it fundamentally different from standard aggregations.

#### Triaging the Question

The question is probing your understanding of SQL window functions and their implementation in aggregate calculations. Specifically, we need to explain why PERCENTILE_CONT requires an ordering within its group using the WITHIN GROUP (ORDER BY ...) clause, and how this impacts its syntax compared to other aggregates.

#### First-Principles Solution

PERCENTILE_CONT is a window function that calculates a continuous percentile value. Unlike standard SQL aggregations such as SUM, COUNT, or MAX, which return a single scalar value for the entire result set, PERCENTILE_CONT operates on an ordered subset of rows and returns a value based on that order.

Key Concepts
Continuous Percentile Calculation: This function doesn't simply pick out one row from your data but rather interpolates between values to calculate a continuous percentile.
Ordering: To perform this interpolation, the rows must be ordered in some meaningful way, which is why PERCENTILE_CONT requires an ORDER BY clause within its WITHIN GROUP.
Syntax and Example

The syntax for PERCENTILE_CONT is:

SQL
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER () AS median_salary
FROM employees;

Here, the OVER () clause is required to indicate that we are performing a window function over all rows in the table. The WITHIN GROUP (ORDER BY salary) part specifies how the rows should be ordered before calculating the continuous percentile.

Detailed Explanation
Why Ordered Sets: Standard aggregations like COUNT(*) or SUM(column_name) can operate on unordered sets of data because they are summary statistics that don't depend on order. However, a percentile requires ordering to determine the position within the distribution.
Cumulative Distribution: The cumulative distribution function (CDF) is used in statistical analysis to provide an ordered view of your data. For example, if you have 10 salaries sorted from lowest to highest, the CDF at any point will give you the proportion of salaries that are less than or equal to that value.
Interpolation: PERCENTILE_CONT uses linear interpolation between values in the ordered set to calculate a continuous percentile. This means it can return fractional positions within the dataset.

#### Example

Let's consider an example with sample data:

SQL
CREATE TABLE salaries (
    id INT,
    salary DECIMAL(10, 2)
);

INSERT INTO salaries (id, salary) VALUES
(1, 50000),
(2, 60000),
(3, 70000),
(4, 80000),
(5, 90000);

To find the median salary:

SQL
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER () AS median_salary
FROM salaries;

This query will return a value that represents the middle point in the ordered set of salaries.

#### Closing Summary
and Common Mistakes

In summary, PERCENTILE_CONT is an ordered-set aggregate because it calculates a continuous percentile by interpolating between values based on their order. This requires using the WITHIN GROUP (ORDER BY ...) clause to specify how rows should be sorted before calculation.

#### Common mistakes
include forgetting the WITHIN GROUP (ORDER BY ...) clause or assuming that PERCENTILE_CONT works like other standard SQL aggregates without needing an ordering. Another common mistake is misunderstanding the continuous nature of percentile values and expecting integer results.

Common Follow-Ups
How does this function handle ties in the data?
What happens if no rows match the specified percentile?
Can you explain the difference between PERCENTILE_CONT and PERCENTILE_DISC?

**Code:**

```

CREATE TABLE salaries ( id INT, salary DECIMAL(10, 2) );

INSERT INTO salaries (id, salary) VALUES (1, 50000), (2, 60000), (3, 70000), (4, 80000), (5, 90000);

CODE
Copy

```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER () AS median_salary
FROM salaries;
```

