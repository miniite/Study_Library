---
type: "practice"
difficulty: "Expert"
---

# How do you build a capacity model that distinguishes soft limits (tunable) from hard limits (architectural ceilings like max table size, LSN wraparound, partition count)?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Capacity Planning and Throughput](<../../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a robust capacity model in database development and architecture, we need to distinguish between soft limits—tunables that can be adjusted by configuration or optimization—and hard limits, which are architectural ceilings like maximum table size or LSN (Log Sequence Number) wraparound. This distinction helps us manage resource constraints effectively without overburdening the system.

#### Triaging the Question

The core of this question is about differentiating between tunable and non-tunable capacity issues in databases. The interviewer is probing your ability to identify and quantify both types of limits, understand their impact on performance, and design strategies to handle them. Assumptions include that you have a good understanding of database architecture, resource management, and the implications of various limits.

#### First-Principles Solution
Identify Hard Limits: These are architectural ceilings that cannot be easily changed without significant rework or redesign. Examples include maximum table size, LSN wraparound periods, number of partitions, etc.
Quantify Capacity Requirements: Understand the current and projected usage patterns to determine when soft limits might become hard bottlenecks.
Implement Monitoring and Alerting: Set up continuous monitoring to detect approaching capacity thresholds. This helps in proactive management before a critical failure occurs.
Optimize Performance Through Tuning: Use tuning techniques such as query optimization, indexing, and resource allocation strategies to mitigate soft limits.
Detailed Solution

Distinguish Hard Limits

Maximum Table Size: The maximum size a table can grow is often constrained by storage capacity or database version limitations.
LSN Wraparound: This occurs when the log sequence number (LSN) overflows, leading to potential data corruption issues if not managed properly.
Partition Count: The total number of partitions in a sharded system is another hard limit that cannot be easily increased.

Tunable Soft Limits

Query Optimization: Improving query performance can reduce the load on the database and increase throughput.
Indexing Strategy: Adding or refining indexes can speed up read operations, reducing overall resource consumption.
Resource Allocation: Dynamic allocation of CPU, memory, and I/O resources based on demand.

Monitoring and Alerting

Implement a monitoring system that tracks key metrics like disk usage, query latency, and transaction rates.
Set up alerts for when these metrics approach critical thresholds, allowing timely intervention.

Optimization Techniques

Use tools like EXPLAIN plans to identify slow queries and optimize them.
Regularly review and update database configurations to ensure they align with current workloads.

#### Closing Summary

In summary, building a capacity model involves recognizing the difference between tunable soft limits and non-tunable hard limits. By identifying and quantifying both types of constraints, implementing robust monitoring, and continuously optimizing performance, you can effectively manage database resources without facing critical failures.

#### Common Mistakes/Follow-Ups
Ignoring Hard Limits: Failing to recognize architectural ceilings can lead to unforeseen system failures.
Over-Tuning Without Data: Relying solely on tuning without understanding the underlying issues might not solve the root problem.
Lack of Proactive Monitoring: Without continuous monitoring, issues may go unnoticed until they become critical.

Would you like me to elaborate further or provide a specific example?

**Comparison:**

```
Limit Type	Name	Value
Hard	Max Table Size	1024GB
	LSN Wraparound Interval	365 days
Soft	Query Execution Time	1 second
	Disk Usage Alert Level	90%
```

**Code:**

```sql
-- Example SQL query for monitoring disk usage
SELECT
    table_name,
    SUM(data_length + index_length) AS total_size
FROM information_schema.tables
WHERE table_schema = 'your_database'
GROUP BY table_name;
```

**Example input / output:**

```text
Input:
- Current table size: 500GB
- Disk usage: 75%

Output:
- No immediate action required for soft limits.
- Schedule a review of query performance and indexing strategy.
```

**Data:**

```json
{
  "limits": {
    "hard": [
      { "name": "Max Table Size", "value": 1024GB },
      { "name": "LSN Wraparound Interval", "value": 365 days }
    ],
    "soft": [
      { "name": "Query Execution Time Threshold", "value": 1 second },
      { "name": "Disk Usage Alert Level", "value": 90% }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Hard Limits
4. Are Limits Tunable?
5. Optimize PerformanceThrough Tuning
6. Implement Monitoring andAlerting
7. Mitigate Soft Limit Issues
8. Monitor and Act on Thresholds

