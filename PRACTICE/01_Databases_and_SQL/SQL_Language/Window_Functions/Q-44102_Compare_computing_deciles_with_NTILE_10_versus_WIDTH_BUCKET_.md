---
type: "practice"
difficulty: "Deep"
---

# Compare computing deciles with NTILE(10) versus WIDTH_BUCKET / PERCENT_RANK; when does NTILE give unequal buckets and does that matter for reporting?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare computing deciles using NTILE(10) versus WIDTH_BUCKET or PERCENT_RANK in SQL, we need to understand how these functions work and their implications on data distribution and reporting accuracy. This comparison will help us decide which method is more suitable for different use cases.

#### Triage

The question asks about the differences between using NTILE(10) and WIDTH_BUCKET or PERCENT_RANK to compute deciles in SQL. Specifically, it explores whether NTILE can produce unequal buckets and if this matters for reporting purposes.

#### Assumptions
We are working with a numeric column that we want to divide into 10 groups (deciles).
The data distribution is not uniform.
We need to understand the implications of using each method on the final report.

#### First Principles Solution
NTILE(10)

NTILE(n) assigns rows into n evenly sized buckets. If the number of rows cannot be perfectly divided by n, some buckets will have one more row than others. This means that with 100 rows, we might end up with nine buckets having 10 rows and one bucket having 10 extra rows (making it 20 rows).

SQL
SELECT NTILE(10) OVER (ORDER BY salary) AS decile
FROM employees;

Output Example:

Bucket 1: 9 rows
Bucket 2: 10 rows
...
Bucket 10: 10 rows
WIDTH_BUCKET and PERCENT_RANK

WIDTH_BUCKET(column, min_value, max_value, num_buckets) divides the range of column values into num_buckets equally spaced buckets. This method ensures that each bucket has a fixed size based on the value range.

SQL
SELECT
    (salary - MIN(salary) OVER ()) /
    ((MAX(salary) OVER () - MIN(salary) OVER ()) / 9) AS bucket,
    salary
FROM employees;

PERCENT_RANK() computes the relative rank of each row within a group, which can be used to determine deciles by setting thresholds.

SQL
SELECT PERCENT_RANK() OVER (ORDER BY salary) AS decile
FROM employees;

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake: Using NTILE for precise bucketing when a uniform distribution is needed, which can lead to unexpected results due to uneven bucket sizes.

#### Follow-Up
If the data distribution significantly affects the decision between these methods, consider using WIDTH_BUCKET or adding conditional logic to handle unequal buckets in NTILE.
When to Use NTILE(10): When a simple method is needed and slight imbalances are acceptable. It’s easier to implement for basic reporting.
When to Use WIDTH_BUCKET/PERCENT_RANK: For more precise and uniform bucketing, especially when the data distribution matters significantly.

#### Example Scenario

Suppose we have an employees table with salaries ranging from 50k to 200k. Using NTILE(10) might result in some buckets having slightly different sizes based on the number of rows. However, using WIDTH_BUCKET ensures that each salary range is divided into roughly equal intervals, which can be more desirable for precise reporting.

Depth + Explanation
NTILE Limitations: NTILE only guarantees that the buckets are evenly distributed in terms of count, not value. This means that while bucket 1 might have fewer high salaries compared to other buckets, it will still contain a similar number of rows.
Width_Bucket Advantages: WIDTH_BUCKET and PERCENT_RANK ensure that each interval covers roughly the same range of values, making it easier to interpret results in terms of salary ranges.

#### Conclusion

Choosing between NTILE, WIDTH_BUCKET, or PERCENT_RANK depends on the specific requirements of your reporting. For precise value-based deciles, WIDTH_BUCKET is often a better choice due to its uniform distribution approach. However, if simplicity and slight imbalances in bucket size are acceptable, NTILE(10) can be sufficient.

**Comparison:**

| Decile Bucketing (NTILE) Example |
| --- |
| id | decile |
| 1  | 2     |
| 2  | 3     |
| 3  | 4     |
| Value-Based Bucketing (WIDTH_BUCKET) Example |
| --- |
| id | bucket | salary |
| 1  | 0      | 50000  |
| 2  | 1      | 60000  |
| 3  | 2      | 70000  |

**Code:**

```sql
-- Example NTILE SQL
SELECT NTILE(10) OVER (ORDER BY salary) AS decile
FROM employees;

-- Example WIDTH_BUCKET SQL
SELECT
    (salary - MIN(salary) OVER ()) /
    ((MAX(salary) OVER () - MIN(salary) OVER ()) / 9) AS bucket,
    salary
FROM employees;
```

**Example input / output:**

```text
Input:
SELECT NTILE(10) OVER (ORDER BY salary) AS decile FROM employees

Output:
id | decile
-- | ------
1  | 2
2  | 3
3  | 4
// ... more data

Input:
SELECT
    (salary - MIN(salary) OVER ()) /
    ((MAX(salary) OVER () - MIN(salary) OVER ()) / 9) AS bucket,
    salary
FROM employees

Output:
id | bucket | salary
-- | ------ | ------
1  | 0      | 50000
2  | 1      | 60000
3  | 2      | 70000
// ... more data
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "Alice", "salary": 50000},
    {"id": 2, "name": "Bob", "salary": 60000},
    {"id": 3, "name": "Charlie", "salary": 70000}
    // ... more data
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Input: SELECT NTILE(10) OVER(ORDER BY salary)
6. Are rows evenly distributed?
7. Output: Equal count of rowsper bucket
8. Output: Unequal count of rowsper bucket
9. Input: WIDTH_BUCKET(salary,50000, 200000, 10)
10. Are value ranges uniform?
11. Output: Uniform value rangedistribution
12. Output: Non-uniform valuerange distribution

