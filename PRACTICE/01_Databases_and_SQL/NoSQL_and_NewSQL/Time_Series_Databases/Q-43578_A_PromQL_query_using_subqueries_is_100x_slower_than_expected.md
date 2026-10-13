---
type: "practice"
difficulty: "Deep"
---

# A PromQL query using subqueries is 100x slower than expected; how do you reason about the evaluation cost and rewrite it for an alerting rule?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When dealing with slow PromQL queries, particularly those involving subqueries, understanding the evaluation cost is crucial. The primary issue here is that subqueries can significantly increase the number of samples processed, leading to performance degradation. To address this, we need to reason about the query structure and identify optimization opportunities by rewriting it using more efficient techniques such as vectorization.

#### Triage

The question asks us to investigate why a PromQL alerting rule using subqueries is 100x slower than expected. The goal is to understand the evaluation cost and propose an optimized solution. We need to consider factors like the number of samples being processed, the complexity of the query, and potential bottlenecks.

Reasoning from First Principles

Prometheus uses a pull-based model where exporters push metrics into Prometheus which then stores them in time series databases like InfluxDB. When writing PromQL queries, especially those with subqueries, it's essential to minimize the number of samples that are processed.

1. Understanding Evaluation Cost
Subquery Impact: Subqueries can dramatically increase the evaluation cost because they often lead to Cartesian product operations between different time series.
Sample Processing: Each sample in a time series can be part of multiple subqueries, leading to repeated processing and increased latency.
2. Identifying the Problem

Let's assume our original query looks something like this:

PROMQL
alert {
  expr: (vector(1) without(*) * vector(2)) > 0
}

Here, vector(1) and vector(2) are time series from different sources or with different labels. The without(*) ensures we're multiplying all samples of one series with all samples of the other, leading to a Cartesian product.

3. Rewriting for Efficiency

To avoid this Cartesian product, we can use vectorized operations and reduce the sample size before performing any complex operations. Here’s a revised approach:

PROMQL
alert {
  expr: (rate(vector(1)[5m]) * rate(vector(2)[5m])) > 0
}

Here, rate is used to compute the rate of change over the last 5 minutes for each time series, reducing the sample size before multiplication.

#### Closing Summary
and Common Mistakes
Optimization Techniques:
Vectorization: Use functions like rate, increase, or movingAverage to reduce the number of samples.
Subquery Avoidance: Minimize subqueries by aggregating data first.
Sample Reduction: Use functions that operate on a subset of samples, reducing overall processing.

#### Common Mistakes
Overusing Subqueries: Failing to recognize how subqueries can lead to Cartesian products and increased evaluation cost.
Ignoring Sample Size: Not considering the number of samples being processed, which directly impacts performance.
Lack of Vectorization: Using complex operations without vectorizing them first.

#### Example
Dry-Run

Let's consider a small example where we have two time series vector(1) and vector(2), each with 1000 samples over the last hour.

Original Query (Slower):
PROMQL
alert {
  expr: (vector(1) without(*) * vector(2)) > 0
}

This results in a Cartesian product of 1,000,000 operations.

Optimized Query (Faster):
PROMQL
alert {
  expr: (rate(vector(1)[5m]) * rate(vector(2)[5m])) > 0
}

Here, rate reduces each series to a moving average over the last 5 minutes, effectively reducing the sample size from 1000 to around 60. The multiplication now involves only 3600 operations.

Final Thoughts

Understanding and optimizing PromQL queries is critical for efficient alerting in AIOps environments. By minimizing subqueries and using vectorization techniques, we can significantly reduce evaluation costs and improve query performance.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
  title PromQL Query Optimization
  dateFormat YYYY-MM-DD
  section Initial Query
      2023-08-01T05:00:00,1000000,Original Subquery
  section Optimized Query
      after 2023-08-01T05:00:00,60000,Reduced Sample Size
```

**Code:**

```
PROMQL
Copy
alert {
  expr: (rate(vector(1)[5m]) * rate(vector(2)[5m])) > 0
}
```

**Example input / output:**

```

Input:

PROMQL
Copy
vector(1) = [1, 2, 3, 4, 5]
vector(2) = [6, 7, 8, 9, 10]

Output:

Original (Slower):
PLAINTEXT
Copy
Cartesian product: 1 * 6, 1 * 7, ..., 5 * 9, 5 * 10 (1000 samples)
Optimized (Faster):
PLAINTEXT
Copy
Rate calculation: [2.4, 3.2] for vector(1) over last 5 minutes
Rate calculation: [7.6, 8.6] for vector(2) over last 5 minutes
Multiplication: 2.4 * 7.6, 3.2 * 8.6 (6 samples)

Dry Run:

PLAINTEXT
Copy
Original: 1000 operations
Optimized: 6 operations
```

**Data:**

```json
{
  "vector(1)": [1, 2, 3, 4, 5],
  "vector(2)": [6, 7, 8, 9, 10]
}
```

**Flow:**

1. Yes
2. No
3. Identify Slow Query
4. Is Subquery Used?
5. Vectorize and Aggregate
6. Optimize Other Aspects
7. Reduce Sample Size
8. Implement Efficient Operations

