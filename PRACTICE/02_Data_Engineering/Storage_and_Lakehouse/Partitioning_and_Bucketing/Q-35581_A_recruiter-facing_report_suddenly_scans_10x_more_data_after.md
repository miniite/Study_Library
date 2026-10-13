---
type: "practice"
difficulty: "Deep"
---

# A recruiter-facing report suddenly scans 10x more data after a partition column type changed from DATE to STRING — how does that break pruning and how do you catch it in CI?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Partitioning and bucketing are essential techniques for optimizing query performance on large datasets, especially when dealing with time-series data. However, a recent change from using a DATE type to a STRING type as a partition column in our recruiter-facing report has led to the report scanning 10x more data than before. This issue stems from how SQL engines handle partition pruning based on column types and data distribution. To fully understand and resolve this problem, we need to delve into the concept of partition pruning and explore potential CI solutions.

Partition pruning works by efficiently selecting only those partitions that are relevant to a query. For example, if our report queries for data within a specific date range, the engine would ideally skip scanning irrelevant partitions outside that range. The change from DATE to STRING type can cause this pruning logic to fail because SQL engines may not be able to effectively filter out irrelevant partitions when the column values are stored as strings.

#### Triage
What's Really Being Probed

The question is probing our understanding of how partitioning works in SQL databases, specifically focusing on the implications of using different data types for partition columns. It also touches on best practices for CI/CD pipelines to ensure that changes like this do not break critical functionality without detection.

#### First-Principles Solution
Understanding Partition Pruning

Partition Column Type Impact: The DATE type is natively understood by the SQL engine, allowing it to perform range queries efficiently. In contrast, a STRING type column does not provide this same level of optimization because the engine cannot use its indexing or filtering capabilities as effectively.

Pruning Logic Failure: When using a DATE partition, the SQL engine can apply filters directly on the partition keys, leading to reduced scan times and lower data volume. With a STRING partition, these filters become less effective since the engine must perform more comprehensive scans across partitions.

Data Distribution Considerations: Even if the data distribution is uniform for both types, the failure of pruning logic means that potentially all partitions may be scanned in the absence of proper indexing or optimization strategies.

Implementing a CI Solution

To catch such issues during the CI process, we can take several steps:

Data Type Validation: Integrate validation checks into our CI pipeline to ensure that any changes involving partition columns are accompanied by appropriate data type annotations and metadata updates.

Query Performance Tests: Develop automated tests that simulate queries using different date ranges and verify whether the correct partitions are being scanned. This can be done by comparing actual query execution plans with expected ones.

Code Reviews and Linting: Ensure that code reviews include checks for partition column usage to prevent incorrect data types from being introduced unintentionally.

Documentation Updates: Maintain up-to-date documentation on best practices for partition columns, including the use of DATE types over STRING types for optimal performance.

#### Closing Summary
Key Takeaways
Changing a partition column type from DATE to STRING can significantly impact query optimization and pruning.
CI pipelines should include checks for partitioning logic correctness and performance testing.
Best practices involve using appropriate data types for partitions and ensuring thorough code reviews.

#### Common Mistakes
and Follow-Ups
Ignoring Data Type Changes: Not recognizing the importance of maintaining proper data types can lead to significant performance issues.
Inadequate CI Coverage: Failing to include comprehensive tests in the CI process may result in undetected pruning failures.
Lack of Documentation: Insufficient documentation on partitioning best practices can lead to repeated mistakes and suboptimal query performance.

**Comparison:**

| Partition Column | Expected Behavior with DATE | Actual Behavior with STRING |
|------------------|----------------------------|----------------------------|
| Pruning          | Effective                   | Ineffective                |
| Scan Volume      | Reduced                     | Increased                  |

**Code:**

```

-- Example SQL query for testing partition pruning with a DATE column SELECT * FROM job_history WHERE start_date BETWEEN '2021-01-01' AND '2023-12-31';

CODE
Copy

```sql
-- Example SQL query for testing partition pruning with a STRING column
SELECT * FROM job_history WHERE start_date_str BETWEEN '2021-01-01' AND '2023-12-31';
```

**Example input / output:**

```text
Input Query:
SELECT * FROM job_history WHERE start_date BETWEEN '2021-01-01' AND '2023-12-31';

Expected Output (with DATE):
Scanned partitions: [2021, 2022]

Actual Output (with STRING):
Scanned partitions: All
```

**Data:**

```json
{
  "job_history": {
    "start_date": "DATE",
    "start_date_str": "STRING"
  }
}
```

**Flow:**

1. Yes
2. No
3. Query Received
4. Check Partition Column Type
5. Is Type DATE?
6. Pruning and OptimizationApplied
7. Ineffective Pruning, Full ScanPossible

