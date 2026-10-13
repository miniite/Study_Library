---
type: "practice"
difficulty: "Deep"
---

# What is the role of percentiles versus averages when reporting query latency, and why do averages mislead?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the role of percentiles versus averages in reporting query latency, we need to first grasp how both metrics are calculated and their respective strengths and weaknesses. Averages can be misleading because they don't capture the full distribution of response times. Percentiles provide a more nuanced view by breaking down the data into segments, allowing us to see where most of the queries fall within the latency spectrum.

TL;DR: Use percentiles for accurate query latency reporting as they reveal the true performance across different parts of your workload. Averages can be deceptive because they may hide long tail issues that significantly impact user experience.

#### Triaging the Question

The question is probing our understanding of statistical metrics in monitoring and their implications on decision-making. We need to clarify a key assumption: we are focusing on query latency, which measures how quickly database operations complete.

Averages are straightforward—summing up all response times and dividing by the number of queries.
Percentiles divide the data into segments based on time spent, such as 95th percentile (the slowest 5% of queries).
Step-by-Step Solution

Let's start with a basic example to illustrate why averages can be misleading.

#### Example Scenario

#### Cons
ider an application with two types of database queries:

Type A: Common and quick; average response time is 10 ms.
Type B: Rare but very slow; average response time is 3 seconds.

If we calculate the average query latency, it would be skewed by the few Type B queries. However, for most users, the performance is actually much better than this average suggests.

Calculating Percentiles

Now let's look at how percentiles can give us a more accurate picture:

95th Percentile: This means 95% of all queries are faster than or equal to the value at the 95th percentile.
If 95% of Type A queries take less than 20 ms, then the 95th percentile might be around 20 ms.

In this case, reporting a 95th percentile latency of 20 ms gives us a much clearer picture of typical user experience compared to an average of 16 ms (assuming one slow query at 3 seconds).

Edge Cases
Noisy Data: Outliers can significantly skew the average but not as much in percentiles.
Skewed Distributions: In highly skewed distributions, averages can be heavily influenced by outliers, making percentiles more reliable.

#### Closing Summary

Percentiles are crucial for understanding query latency because they provide a clearer picture of typical performance. Averages can mask significant issues, such as the presence of slow queries that affect user satisfaction and system reliability. By using percentiles, we can make informed decisions about optimizing database performance and ensuring a better experience for end-users.

#### Common Mistakes/Follow-Ups
Misinterpreting Percentiles: Percentiles should not be confused with averages; they answer different questions.
Neglecting Averages: While percentiles are valuable, they shouldn't replace the use of averages entirely. Both metrics together give a more complete picture.
Ignoring Outliers: Outliers can still affect percentiles but less so compared to averages. Understanding their impact is important.

Do you have any specific scenarios or follow-up questions related to this topic? I'd be happy to discuss them further.

**Comparison:**

```
Query Type	Response Time (ms)
A	10
B	3000
C	15
Average Latency	95th Percentile
16 ms	28 ms
```

**Code:**

```sql
-- Example SQL query for calculating 95th percentile
SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY response_time) AS p95
FROM query_latency_table;
```

**Example input / output:**

```text
Input:
95th percentile calculation for query latency

Output:
p95 = 28 ms
```

**Data:**

```json
{
  "query_latency_table": [
    {"id": 1, "response_time": 10},
    {"id": 2, "response_time": 3000}, // 3 seconds in milliseconds
    {"id": 3, "response_time": 5}
  ]
}
```

**Flow:**

1. 3 seconds :::long
2. Query Type
3. Type A
4. Type B
5. Response Time
6. F
7. Average Latency
8. 16 ms
9. 95th Percentile
10. 28 ms

