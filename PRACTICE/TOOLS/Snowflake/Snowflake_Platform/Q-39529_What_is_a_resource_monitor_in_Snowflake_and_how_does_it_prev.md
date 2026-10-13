---
type: "practice"
difficulty: "Intermediate"
---

# What is a resource monitor in Snowflake and how does it prevent runaway credit usage?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Snowflake Compute and Caching](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Compute_and_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A resource monitor in Snowflake is a critical feature designed to detect and mitigate instances of excessive or "runaway" queries that could consume an uncontrolled amount of credits, potentially leading to financial loss or resource exhaustion. In this context, "runaway" refers to queries that are either extremely inefficient or have infinite loops, causing an indefinite consumption of resources.

TL;DR: The primary role of a resource monitor is to identify and limit the execution time and memory usage of problematic queries before they consume all available credits, ensuring balanced resource utilization across your Snowflake account.

To triage this question, we need to understand that Snowflake's resource management system includes multiple layers. A resource monitor operates at one such layer, focusing on detecting anomalies in query behavior and managing those through defined policies or thresholds.

#### Triaging the Question
What is a Resource Monitor?
A resource monitor watches over queries as they execute within Snowflake.
What Does it Prevent?
It prevents "runaway" credit usage by identifying and limiting problematic queries.

#### Assumptions
We assume the interviewer is interested in how this feature works under-the-hood, its configuration options, and potential scenarios where it might be necessary.

#### First-Principles Solution

Understanding Query Execution:

Each query in Snowflake consumes credits based on its complexity and resource usage.
Credits are a billing unit representing the amount of compute resources used by a query.

Identifying Runaway Queries:

A "runaway" query might be one that:
Has an infinite loop or recursive logic.
Performs poorly due to bad join conditions, lack of indexes, etc.
Does not return results in a timely manner and continues to run indefinitely.

Resource Monitor Mechanism:

The resource monitor continuously tracks the progress of running queries.
It uses various metrics such as:
CPU time used
Memory consumption
Duration since query start

Thresholds and Policies:

When a query exceeds predefined thresholds, the resource monitor triggers actions to limit its execution.
These actions include:
Setting a maximum duration for a query (e.g., 30 minutes).
Imposing memory limits on queries.
Allocating time slices to ensure other queries get a fair share of resources.

Action Plan:

If a query exceeds the threshold, it may be paused or terminated.
Notifications are sent via email/webhook to alert administrators about the issue.

#### Closing Summary
+ Common Mistakes/Follow-Ups
A resource monitor ensures that no single query can dominate and exhaust all credits, thus maintaining balanced usage.

#### Common mistakes
include misconfiguration of thresholds, which could either allow runaway queries or prematurely terminate useful ones.

#### Follow-ups
might involve discussing how to configure the resource monitor effectively, trade-offs between cost control and query performance, and best practices for monitoring and managing resources.

**Code:**

```sql
-- Example configuration snippet in Snowflake
ALTER WAREHOUSE my_warehouse SET
    CpuCount = 4,
    DynamicScalingMinimumCpuPercentage = 20,
    MaxConcurrentStatementsPerUser = 5;
```

**Example input / output:**

```text
Input:
- Query: SELECT * FROM large_table WHERE condition;

Output:
- If the query exceeds the configured threshold, it is paused or terminated.
- A notification is sent to the administrator.
```

