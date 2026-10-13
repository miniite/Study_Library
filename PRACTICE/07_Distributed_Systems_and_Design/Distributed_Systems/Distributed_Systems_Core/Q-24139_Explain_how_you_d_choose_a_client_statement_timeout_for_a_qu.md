---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd choose a client statement_timeout for a query against a distributed database given its p50/p99/p999 latency. Why does setting it at p99 trigger a retry storm the moment the latency distribution shifts?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deadline Propagation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When selecting a statement_timeout for a query in a distributed database, we need to balance performance and reliability while considering the specific characteristics of our workload, such as its latency distribution (p50/p99/p999). For example, if you're working with a distributed database like CockroachDB or PostgreSQL, setting an appropriate statement_timeout is crucial for maintaining the overall health and responsiveness of your application. The choice should be based on understanding where 99% of queries fall in terms of latency, as well as how long it takes to retry failed statements.

To ensure reliability without causing a retry storm, we need to set statement_timeout at a value that is higher than the p99 latency but lower than the worst-case scenarios. This approach prevents unnecessary retries while still handling most queries efficiently.

#### Triage
What's Being Probed

The question is probing your understanding of distributed systems, specifically how latency distribution affects query timeouts and retry mechanisms in a highly available database environment. It also tests your ability to reason about trade-offs between performance and reliability.

#### Assumptions
The database system supports statement_timeout settings.
The workload has well-defined p50/p99/p999 latency metrics.
The database is distributed, meaning there are multiple nodes involved in the query execution.

#### First Principles Solution

To set a statement_timeout, we need to understand the typical response time of our queries and ensure that the timeout value is appropriate for most cases. Here's how you can approach it:

Analyze Latency Distribution:

Identify p50, p99, and p999 latencies from historical query performance data.
For instance, if your p99 latency is 1 second, setting the statement_timeout to just above this value (e.g., 2 seconds) can be a good starting point.

Determine Retries:

A retry storm occurs when many queries fail within a short period, causing an exponential increase in retries.
To avoid this, ensure that the statement_timeout is set to handle typical failures without triggering unnecessary retries.

Set statement_timeout:

Choose a value slightly above p99 latency but not too high to avoid excessive waiting times for slow queries.
For example, if your p99 latency is 1 second, setting the timeout to 2 seconds can be appropriate.

Monitor and Adjust:

Continuously monitor query performance and adjust statement_timeout as needed.
Use A/B testing or gradual rollout strategies to ensure changes don't negatively impact overall system stability.

#### Closing Summary

Setting a statement_timeout in a distributed database involves understanding the latency distribution of your queries. By choosing a value slightly above p99 but not too high, you can balance performance and reliability without triggering retry storms. Regular monitoring is essential to ensure that settings remain appropriate as workloads change over time.

#### Common Mistakes
and Follow-Ups
Mistake 1: Setting statement_timeout too low, leading to frequent retries.
Mistake 2: Setting it too high, causing unnecessary waiting times for slow queries.

#### Follow-Up
Question: How would you handle scenarios with highly variable p90-p99 latencies?

**Comparison:**

```
Metric	p50	p90	p99	p999
Time	100	400	800	2000
```

**Code:**

```sql
-- Example SQL setting statement_timeout in PostgreSQL
SET LOCAL statement_timeout TO '2000ms';
```

**Example input / output:**

```text
Input:
- Latency distribution with p99 = 800ms

Output:
- Set statement_timeout to 1000ms (slightly above p99 latency)
```

**Data:**

```json
{
  "latency_distribution": {
    "p50": 100,
    "p90": 400,
    "p99": 800,
    "p999": 2000
  }
}
```

**Flow:**

1. Yes
2. No
3. Analyze Latency Distribution
4. p99 = 800ms?
5. Set statement_timeout to1000ms
6. Adjust based on p90-p99distribution

