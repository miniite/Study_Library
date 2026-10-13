---
type: "practice"
difficulty: "Deep"
---

# What is the difference between VAR/VARIANCE and STDDEV aggregate functions?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Window Functions Overview](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between VAR (Variance) and STDDEV (Standard Deviation) in SQL, we need to grasp their mathematical definitions and how they are used in statistical analysis. These functions help us measure the dispersion or spread of a dataset around its mean value.

#### Triage

The question asks for the distinction between VAR and STDDEV. The interviewer is likely probing your understanding of basic statistics and SQL aggregation functions, as well as how to interpret their results. We'll assume you're familiar with basic concepts like variance and standard deviation from a statistical standpoint before delving into SQL specifics.

#### First-Principles Solution

Variance (VAR):

Definition: Variance is the average of the squared differences from the Mean.
Formula: If x represents each value in the dataset, then the population variance ( \sigma^2 ) is given by: [ \sigma^2 = \frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2 ] where ( \mu ) is the mean of the dataset.
Interpretation: It measures how far each number in the set is from the mean and thus gives an idea of the spread.

Standard Deviation (STDDEV):

Definition: Standard Deviation is the square root of Variance, which provides a measure of dispersion that is easier to interpret because it's in the same units as the original data.
Formula: The population standard deviation ( \sigma ) is: [ \sigma = \sqrt{\sigma^2} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2} ]
Interpretation: It's the square root of variance and provides a more intuitive measure of spread.
SQL Implementation

In SQL, these functions are implemented as follows:

Variance (VAR)
In PostgreSQL:
SQL
SELECT var_samp(column_name) FROM table_name;
var_samp is an unbiased estimator.
In MySQL:
SQL
SELECT variance(column_name) FROM table_name;
Standard Deviation (STDDEV)
In PostgreSQL:
SQL
SELECT stddev_samp(column_name) FROM table_name;
stddev_samp is also an unbiased estimator.
In MySQL:
SQL
SELECT stddev(column_name) FROM table_name;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, while both functions are used to measure the dispersion in a dataset, Variance provides the average squared difference from the mean, whereas Standard Deviation gives the square root of this value. You should choose STDDEV when you want a more interpretable measure of spread.

#### Common mistakes

Using VAR and VARIANCE interchangeably.
Assuming they both follow the same syntax across different database systems without checking documentation.

If the interviewer probes further, they might ask about scenarios where one function is preferred over the other, or how these functions handle null values. Always consider the context of your dataset and the insights you need to draw from it when choosing between Variance and Standard Deviation.

**Comparison:**

| Variance and Standard Deviation of Salaries |
|--------------------------------------------|
| Salary | Variance (Var) | Std Dev (Stddev) |
|--------|----------------|-----------------|
| 5000   | 1000           | 31.62           |
| 6000   |                |                 |
| 7000   |                |                 |

**Code:**

```sql
-- Example in PostgreSQL for variance and standard deviation

SELECT var_samp(salary) AS variance_salary,
       stddev_samp(salary) AS std_dev_salary
FROM employees;
```

**Example input / output:**

```text
-- Sample input and output

Input:
SELECT var_samp(salary) AS variance_salary,
       stddev_samp(salary) AS std_dev_salary
FROM (VALUES (5000), (6000), (7000)) AS salaries(salary);

Output:
| variance_salary | std_dev_salary |
|-----------------|---------------|
| 1000            | 31.62         |
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "Alice", "salary": 5000},
    {"id": 2, "name": "Bob", "salary": 6000},
    {"id": 3, "name": "Charlie", "salary": 7000}
  ]
}
```

**Flow:**

1. Input Data
2. Calculate Mean
3. Subtract Mean from EachValue
4. Square Differences
5. Average Squared Differences(Variance)
6. Square Root of Variance (StdDev)

