---
type: "practice"
difficulty: "Deep"
---

# What is a histogram in database statistics and how does it help estimate skewed data distributions?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database terms, a histogram is a statistical tool used to summarize large datasets by dividing them into ranges or bins, where each bin represents the frequency of values within that range. This summary helps databases make more informed decisions about how to execute queries efficiently, especially when dealing with skewed data distributions.

#### Triaging the Question

The core of the question revolves around understanding what a histogram is and how it aids in query tuning, particularly for datasets with skewed distributions. The interviewer might be probing your knowledge of:

Basic concepts related to histograms.
How histograms are used in database management systems (DBMS) for query optimization.
Your ability to explain the practical implications of skewed data on performance and how histograms can mitigate these issues.

#### First Principles Solution

Understanding Histograms:

A histogram is a representation of the distribution of values in a dataset.
In databases, histograms are used by query optimizers to estimate the selectivity (the proportion of rows that match certain criteria) and cardinality (the number of distinct values) of queries.

How Histograms Work:

When creating a histogram, the database system analyzes the data and bins it into ranges based on the distribution.
For skewed distributions, more frequent value ranges are given more detail to ensure accurate estimates.

Practical Application in Query Optimization:

By using histograms, the query optimizer can make better decisions about which index to use, how many rows to return, and other execution strategies.
This is particularly useful for queries involving filters or sorting that rely on understanding the distribution of data.
Detailed Explanation
Histogram Construction: When a database indexes a column, it may also create a histogram to capture the frequency distribution. For example, if we have a column salary with values ranging from 10,000 to 200,000 and most employees earn around 50,000, a histogram might show this concentration of values in one bin.
Query Estimation: When a query like SELECT * FROM employees WHERE salary > 100000 is run, the database can use the histogram to estimate how many rows will match. If the histogram shows that only a few high-salary bins are relevant, fewer rows need to be scanned.
Skew Handling: For skewed data like salary, histograms ensure that even less frequent but important ranges (like very high salaries) are still represented adequately.

#### Closing Summary
and Common Mistakes

In summary, histograms in databases help by providing an accurate representation of the distribution of values, which is crucial for efficient query execution. Common mistakes include:

Overlooking Skew: Not understanding that skewed data requires more detailed binning.
Ignoring Cardinality Estimation: Failing to recognize how cardinality estimates influence query plans.
Misinterpreting Histograms: Misunderstanding the purpose and limitations of histograms, such as their dependency on the sample size.

**Comparison:**

| min_salary | max_salary | frequency |
|-----------|-----------|----------|
| 10000     | 20000     | 50        |
| 20001     | 40000     | 75        |
| 40001     | 60000     | 100       |
| 60001     | 80000     | 125       |
| 80001     | 100000    | 150       |
| 100001    | 150000    | 75        |
| 150001    | 200000    | 50        |

**Code:**

```sql
-- Example SQL Query for Creating a Histogram
CREATE STATISTICS salary_histogram ON employees(salary);
```

**Example input / output:**

```text
Input Query: SELECT * FROM employees WHERE salary > 80000;
Estimated Rows by Histogram: ~325 (out of total 400)

Input Query: SELECT AVG(salary) FROM employees WHERE department = 'Sales';
Estimated Cards by Histogram: ~150 (actual count might be slightly different)
```

**Data:**

```json
{
  "salary_distribution": [
    { "min_salary": 10000, "max_salary": 20000, "frequency": 50 },
    { "min_salary": 20001, "max_salary": 40000, "frequency": 75 },
    { "min_salary": 40001, "max_salary": 60000, "frequency": 100 },
    { "min_salary": 60001, "max_salary": 80000, "frequency": 125 },
    { "min_salary": 80001, "max_salary": 100000, "frequency": 150 },
    { "min_salary": 100001, "max_salary": 150000, "frequency": 75 },
    { "min_salary": 150001, "max_salary": 200000, "frequency": 50 }
  ]
}
```

**Flow:**

1. No
2. Database receives query
3. Is the column indexed?
4. Create histogram
5. Use histogram for estimation
6. Estimate selectivity andcardinality
7. Choose optimal execution plan
8. Execute query

