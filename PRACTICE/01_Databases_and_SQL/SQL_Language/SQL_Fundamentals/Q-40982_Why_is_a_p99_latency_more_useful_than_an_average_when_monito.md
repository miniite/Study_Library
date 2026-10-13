---
type: "practice"
difficulty: "Intermediate"
---

# Why is a p99 latency more useful than an average when monitoring service health, and how do you compute it in SQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why p99 latency is more useful than average latency for monitoring service health, we need to consider the nature of distributed systems and their performance characteristics. The key idea behind using p99 latency lies in its ability to focus on tail events—those extreme cases where the system's performance degrades significantly. In contrast, the average can obscure these issues by masking outliers with normal values.

#### Triage
What the Question is Asking

The question asks two things:

Why p99 latency is more useful than average latency for monitoring service health.
How to compute p99 latency using SQL.

#### Assumptions
and Context
We are dealing with a distributed system where performance metrics like latency can vary widely between requests.
The system's overall health should not be compromised by occasional extreme cases of poor performance.
Step-by-Step Solution
Why p99 Latency is More Useful
Tail Behavior: In many systems, the majority of requests have low latency (e.g., under 10 ms), while a few requests can take much longer due to various factors such as network issues or resource bottlenecks.
User Experience: Users are more concerned with the worst-case scenario; they want fast responses most of the time but cannot tolerate frequent slow responses.
Resource Utilization: Monitoring p99 latency helps in optimizing resources and identifying potential bottlenecks that can be addressed to improve overall service quality.
How to Compute p99 Latency Using SQL

To compute p99 latency, you need to find the 99th percentile value of a dataset. This is often done by sorting the data and finding the appropriate threshold.

Sorting: First, sort the dataset in ascending order.
Index Calculation: Calculate the index for the 99th percentile, which is 0.99 * (number_of_samples).
Interpolation: If this index is not an integer, interpolate between the two closest values.

Here’s how you can compute it using SQL:

SQL
WITH latency_samples AS (
    SELECT latency_ms FROM request_logs -- This is your actual data source
)
SELECT
  -- Interpolate between the 98th and 99th samples if necessary
  (100 * row_number() OVER (ORDER BY latency_ms) - 99) / count(*) FILTER (WHERE latency_ms < 1000) AS p99_latency
FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY latency_ms) as row_num, latency_ms
    FROM latency_samples
    ORDER BY latency_ms DESC -- Ensure the data is sorted in reverse order for correct percentile calculation
);

This query first sorts the latency samples and then calculates the 99th percentile by determining the appropriate index. If the result is not an integer, it interpolates between the values.

#### Closing Summary
+ Common Mistakes

#### Common Mistakes
One common mistake is to directly average all latencies without considering the extreme cases.
Why p99 Latency Matters: By focusing on the 99th percentile, we ensure that service health metrics accurately reflect user experience and help in identifying critical performance issues.
Final Thoughts

Understanding why p99 latency is more useful requires a deep dive into system behavior. Using SQL effectively allows us to implement these metrics efficiently and gain valuable insights into our services' performance characteristics.

