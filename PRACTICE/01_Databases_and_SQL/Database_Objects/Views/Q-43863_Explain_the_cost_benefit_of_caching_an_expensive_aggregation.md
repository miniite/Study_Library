---
type: "practice"
difficulty: "Deep"
---

# Explain the cost/benefit of caching an expensive aggregation as a materialized view versus a summary table you maintain via triggers; when does each win?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Trade-offs](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between caching an expensive aggregation as a materialized view (MV) or maintaining a summary table through triggers, we need to weigh several factors including performance benefits, maintenance overhead, and data consistency.

#### Overview
Materialized views offer a precomputed result set that can be used for fast query responses. On the other hand, summary tables are typically created via triggers on write operations, ensuring real-time updates but with additional complexity and potential issues around concurrency and consistency.

#### Triage

Question Core: Assessing when materialized views or summary tables provide better performance and maintainability.
Interviewer's Probing: Understanding the trade-offs between read-heavy and write-heavy workloads; exploring the implications of real-time vs. precomputed data.

Step-by-Step Solution:

Identify Use Case:

Read-Heavy Workload: Materialized views are ideal for scenarios where queries are read-intensive, as they significantly reduce query execution time by avoiding expensive computations.
Write-Heavy Workload: Summary tables maintained via triggers can handle real-time updates but may introduce latency during inserts and updates.

#### Performance Considerations

Materialized View: Reduces the computational load on database servers, improving overall read performance. The MV is updated either manually or using a scheduler.
Summary Table: Real-time updates ensure data accuracy at the cost of potentially slower write operations due to trigger overhead.

Maintenance Overhead:

Materialized View: Requires regular maintenance, such as refreshing the view when underlying data changes. This can be automated but may need manual intervention if the schema evolves.
Summary Table: Continuously updated by triggers, which might lead to increased complexity in handling concurrency issues and transactional integrity.

Data Consistency:

Materialized View: Can fall behind real-time data as updates are batched or scheduled. Suitable for scenarios where a slight delay is acceptable.
Summary Table: Ensures near-instantaneous consistency with the base tables but may introduce complexity in managing concurrent writes.

Scalability and Cost:

Materialized View: Can scale by refreshing more frequently, reducing the refresh interval to minimize staleness but potentially increasing resource usage.
Summary Table: Scalable through efficient trigger design but may require additional storage for the summary table.

#### Closing Summary

Materialized views excel in read-heavy environments where performance is critical and a slight delay in data freshness can be tolerated.
Summary tables are preferable in real-time applications requiring immediate updates, albeit with increased complexity and potential concurrency issues to manage.

#### Common Mistakes/Follow-Ups

Overlooking the maintenance costs of materialized views.
Ignoring the trigger overhead in summary table implementations.
Failing to consider the trade-off between data freshness and performance when choosing a solution.

