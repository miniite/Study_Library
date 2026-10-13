---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether to add a NOLOCK-style hint to speed up a reporting procedure, and what would you warn stakeholders about?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Stored Procedure Performance and Design](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When optimizing a stored procedure for a reporting task, one common technique is to use hints like NOLOCK to improve read performance by reducing locking contention. However, deciding whether to add such a hint requires careful consideration of several factors and clear communication with stakeholders about potential trade-offs.

Firstly, let's triage the question: The key aspects we need to address are:

Identifying when a reporting procedure might benefit from NOLOCK.
Evaluating the performance impact.
Discussing the risks associated with using non-transactional consistency hints.
Explaining these considerations to stakeholders.

#### Assumptions
include that the database is under heavy read workload, and the reporting task does not require transactional consistency or a high level of data accuracy.

To decide whether to add NOLOCK, we should follow these steps:

Analyze Query Performance: Start by profiling the stored procedure to identify which queries are causing performance bottlenecks.
Understand Data Update Frequency: Determine how often data is updated, as this impacts the likelihood of non-transactional consistency issues.
Evaluate Read-Heavy vs Write-Heavy Workloads: In highly read-heavy workloads, NOLOCK can provide significant performance benefits.

Now, let's go through each step in detail:

Profiling and Identifying Bottlenecks:

Use SQL Server Profiler or Dynamic Management Views (DMVs) to identify slow-running queries.
Check for long wait times on lock waits or latch waits.

Assessing Data Update Frequency:

If data is updated infrequently, NOLOCK might be safe to use.
If updates are frequent and critical, using NOLOCK could lead to reading stale data.

Testing with NOLOCK:

Introduce a WITH (NOLOCK) hint in the suspect queries.
Monitor the performance impact using execution plans and actual query times.
Ensure no critical data is being missed by the procedure after adding the hint.

Communication to Stakeholders:

Explain that while NOLOCK can improve read speed, it might also introduce non-transactional consistency issues.
Discuss trade-offs: better read performance vs. potential for stale data.
Recommend using other techniques like partitioning or indexing to balance these concerns.

#### Closing Summary
and Common Mistakes/Follow-Ups: In conclusion, adding NOLOCK can be effective in speeding up reporting procedures by reducing locking contention. However, it's crucial to thoroughly test and communicate with stakeholders about the potential risks, especially regarding data integrity and consistency. Regular monitoring is essential to ensure that performance improvements do not compromise critical business requirements.

#### Common mistakes
include:

Overusing NOLOCK without proper profiling.
Failing to monitor the impact on overall system behavior.
Not considering alternative optimizations like indexing or partitioning.

