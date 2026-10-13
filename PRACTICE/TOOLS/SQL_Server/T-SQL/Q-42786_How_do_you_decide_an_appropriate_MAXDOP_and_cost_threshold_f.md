---
type: "practice"
difficulty: "Expert"
---

# How do you decide an appropriate MAXDOP and cost threshold for parallelism for a mixed OLTP and reporting workload?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up MAXDOP (Maximum Degree of Parallelism) and the Cost Threshold for Parallelism in SQL Server, especially for a database handling both transactional processing (OLTP) and reporting workloads, it's crucial to balance performance efficiency with resource utilization. Here’s how you can approach this:

Understanding the Goal: The primary goal is to ensure that parallel queries run efficiently without overwhelming your CPU or other critical resources.

#### Assumptions
You have a mixed workload consisting of both OLTP and reporting queries.
Your server has multiple cores and is adequately powerful to handle concurrent operations.

#### Triaging the Question
The question is essentially asking how to configure MAXDOP and Cost Threshold for Parallelism in SQL Server, given your workload characteristics.
The interviewer is probing your understanding of concurrency control, resource management, and query performance tuning.
Step-by-Step Solution from First Principles

Determine the Right MAXDOP:

OLTP Workload: OLTP queries are typically short-lived and involve frequent reads and writes. Setting a low MAXDOP (e.g., 2 or 4) can help reduce contention on shared resources.
Reporting Workload: Reporting queries often deal with larger datasets, and using higher MAXDOP values (up to the number of CPU cores minus one, if applicable) can improve query execution speed.

Cost Threshold for Parallelism:

This parameter controls at what point SQL Server decides whether to start a parallel plan.
For OLTP workloads, setting this threshold lower might be better since short-running queries may not benefit from parallelism and could even cause performance issues if they compete with longer running queries.

Dynamic Adjustments:

#### Cons
ider using Dynamic Management Views (DMVs) such as sys.dm_exec_query_stats to monitor the execution plans of different types of queries.
Use Performance Monitor counters like % Processor Time, SQL Server: Batch Requests/sec, and SQL Server: User Connections to observe the server's state.

Testing and Iteration:

Perform load testing under realistic conditions using tools like SQLIO, Idera Load Generator, or custom scripts.
Analyze query performance and adjust settings based on observed behavior.

#### Closing Summary

In summary, for a mixed OLTP and reporting workload, setting an appropriate MAXDOP and Cost Threshold for Parallelism involves understanding the nature of your queries, leveraging monitoring tools, and iteratively tuning these settings. Too high or too low values can lead to suboptimal performance, so it's crucial to strike the right balance.

#### Common Mistakes
and Follow-Ups
Mistake: Setting MAXDOP too high for OLTP workloads can lead to resource contention.

#### Follow-Up
Discussing how to use DMVs to identify long-running queries that might benefit from parallelism could be a good follow-up topic.

**Comparison:**

```
Setting	OLTP Workload	Reporting Workload
MAXDOP	2-4	CPU Core Count - 1
Cost Threshold for Parallelism	<50	>50
```

**Code:**

```sql
-- Example of setting MAXDOP and Cost Threshold in T-SQL

ALTER DATABASE YourDatabaseName SET MAXDOP = 4;
ALTER DATABASE YourDatabaseName SET cost_threshold_for_parallelism = 50;
```

