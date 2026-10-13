---
type: "practice"
difficulty: "Deep"
---

# Your Trino query fails with 'exceeded memory limit' on the coordinator or a worker. Explain the distributed memory model (query per-node exchange) and how spill-to-disk and resource groups change which queries survive under contention.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address why your Trino query might fail with an "exceeded memory limit" error on the coordinator or a worker, we need to understand the distributed memory model in Trino and how spill-to-disk and resource groups play into the equation.

TL;DR:

The distributed memory model in Trino is designed around per-node exchange, where tasks are executed independently. Spill-to-disk allows queries to temporarily store data on disk when memory limits are exceeded, but this can introduce performance overhead. Resource groups help manage contention by allocating resources more effectively, ensuring that critical queries get the necessary CPU and memory.

#### Triaging the Question
Understanding the Problem: When a query fails due to an "exceeded memory limit" error, it means the query requires more memory than is available on the node or nodes handling the execution.
Key Probes:
The distributed nature of Trino and how tasks are executed across nodes.
How spill-to-disk works and its impact on performance.
The role of resource groups in managing memory and CPU resources.

#### First-Principles Solution
1. Understanding the Distributed Memory Model (Per-Node Exchange)

Trino uses a distributed architecture where each node executes tasks independently, and intermediate data is exchanged between nodes using per-node exchange. This means that each node has its own set of allocated memory for executing tasks. If a query requires more memory than what's available on a single node, it can fail.

2. Spill-to-Disk Mechanism

When the memory limit is exceeded, Trino implements spill-to-disk to temporarily store intermediate data on disk. This mechanism helps queries continue running by freeing up in-memory resources for other tasks. However, this comes with a significant performance overhead because reading and writing to disk are much slower than accessing memory.

3. Resource Groups

Resource groups (also known as query resource limits) allow finer control over how memory and CPU resources are allocated between different queries. By setting appropriate resource group policies, you can ensure that critical or time-sensitive queries get the necessary resources while less important queries might be throttled.

Detailed Explanation:

Per-Node Exchange: Each node in Trino has its own set of memory limits for executing tasks. When a query is split across multiple nodes, each node will try to use as much available memory as possible before spilling data to disk. If the total memory required by all nodes exceeds their combined capacity, the query will fail.

Spill-to-Disk: To handle cases where memory limits are exceeded, Trino uses spill-to-disk mechanisms. These mechanisms allow queries to store intermediate results on disk and read them back when needed. While this prevents immediate failures, it introduces significant performance overhead due to the slower speed of disk I/O compared to in-memory operations.

Resource Groups: Resource groups provide a way to manage how memory and CPU resources are allocated between different queries. By setting resource limits for each group, you can ensure that critical queries get the necessary resources while less important queries might be throttled or even blocked from running if there isn't enough capacity available.

#### Closing Summary

In summary, the distributed memory model in Trino is designed to handle tasks independently on each node with per-node exchange. When memory limits are exceeded, spill-to-disk mechanisms can help but introduce performance overhead. Resource groups provide a way to manage contention and ensure critical queries get the necessary resources, preventing overall system degradation.

#### Common Mistakes
Not Understanding Spill-to-Disk: Failing to recognize that excessive use of disk I/O can significantly impact query performance.
Ignoring Resource Groups: Not setting up appropriate resource limits for different types of queries, leading to suboptimal performance and increased contention.
Improper Memory Allocation: Overestimating the memory requirements of a query without proper profiling or underestimating available resources.
Final Thoughts:

Effective management of these distributed systems components—memory model, spill-to-disk, and resource groups—is crucial for ensuring optimal performance and reliability in Trino. Understanding these aspects will help you diagnose and mitigate issues related to memory limits more effectively.

**Comparison:**

```
Node Name	Memory Limit (MB)
Node1	8192
Node2	6144
Resource Group	CPU Limit (%)	Memory Limit (%)
criticalQueries	20	50
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Query Execution] --> B{Memory Limit?};
    B -->|Yes| C(Execute Spill-to-Disk);
    B -->|No| D[Continue Execution];
    C --> E[Read Data from Disk], F[Free Memory for Other Tasks];
```

**Example input / output:**

```text
Input:
- Query with estimated memory requirements of 10GB.
- Current node configurations with Node1 having 8GB and Node2 having 6GB.

Output:
- Spill-to-Disk is triggered on both nodes as the query exceeds their combined capacity, leading to significant performance degradation due to disk I/O operations.
```

**Data:**

```json
{
  "resourceGroup": {
    "name": "criticalQueries",
    "cpuLimit": "20%",
    "memoryLimit": "50%"
  },
  "nodeMemoryLimits": [
    {"nodeName": "Node1", "memoryLimit": "8GB"},
    {"nodeName": "Node2", "memoryLimit": "6GB"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query Execution
4. Memory Limit?
5. Execute Spill-to-Disk
6. Continue Execution

