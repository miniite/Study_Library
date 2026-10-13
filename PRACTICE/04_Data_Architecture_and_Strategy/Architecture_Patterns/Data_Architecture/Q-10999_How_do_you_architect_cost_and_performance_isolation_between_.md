---
type: "practice"
difficulty: "Deep"
---

# How do you architect cost and performance isolation between ad-hoc analyst queries and scheduled production pipelines sharing one warehouse, when a runaway analyst query can starve a revenue-critical pipeline? Compare separate virtual warehouses, workload management queues, and query priority, and where each fails.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect cost and performance isolation between ad-hoc analyst queries and scheduled production pipelines sharing one warehouse while mitigating the risk of a runaway analyst query starving a revenue-critical pipeline, we need to carefully balance cost-efficiency with performance guarantees. The key concepts involve leveraging separate virtual warehouses, workload management queues, and dynamic query priority systems.

This problem is fundamentally about resource contention management in a shared environment where different types of queries have varying needs and importance. Ad-hoc analyst queries can be unpredictable, potentially using excessive resources, whereas production pipelines are mission-critical and require consistent performance levels. Our goal is to ensure that the critical production pipelines always receive the necessary resources.

Firstly, let's consider separate virtual warehouses (VWs). VWs allow us to create isolated environments with distinct compute, storage, and memory configurations. However, this approach has limitations:

It increases infrastructure costs since we need multiple dedicated warehouses.
Management complexity is higher as we must maintain multiple instances.
There might be inefficient use of resources if not all VWs are fully utilized.

Secondly, workload management queues can be a more efficient solution. By organizing queries into different queues with varying priority levels, we can ensure that critical pipelines always get the first chance at processing:

High-priority production pipelines would be given precedence.
Analyst queries could be placed in lower priority queues.
This approach minimizes resource contention and ensures that critical operations are not disrupted.

Thirdly, query prioritization mechanisms can further enhance this setup. By dynamically adjusting the priority of queries based on their current resource usage and historical patterns, we can fine-tune performance:

Queries can be marked with labels or tags to indicate their priority.
A scheduler can monitor real-time metrics (e.g., CPU, memory) and adjust priorities accordingly.

Comparing these approaches:

Virtual Warehouses:

#### Pros
Clear isolation, easy management of dedicated resources.

#### Cons
Increased costs, higher complexity in deployment and maintenance, potential underutilization of resources.

Workload Management Queues:

#### Pros
Cost-effective, dynamic resource allocation based on priority, lower management overhead compared to multiple VWs.

#### Cons
May not provide as clear isolation, requires robust monitoring and scheduling mechanisms.

Query Priority Mechanisms:

#### Pros
Balances cost and performance effectively by dynamically adjusting priorities in real-time.

#### Cons
Requires sophisticated monitoring tools and might still face challenges during peak load times.

In conclusion, the best approach depends on specific use cases, budget constraints, and operational requirements. For a balance between cost and performance, workload management queues with dynamic query priority are often the most practical solution. They offer flexibility and efficiency while minimizing the risk of resource contention affecting critical operations.

#### Common mistakes
include:

Over-relying on separate VWs without considering their added costs.
Ignoring real-time monitoring and failing to implement effective queue management.
Underestimating the complexity involved in dynamic query priority systems, particularly during high load periods.

**Comparison:**

```
Approach	Pros	Cons
Virtual Warehouses	Clear isolation, easy management	Higher costs, complexity
Workload Management Queues	Cost-effective, dynamic resource allocation	Potential for lower isolation
Query Priority Mechanisms	Flexibility, real-time adjustments	Requires sophisticated monitoring
```

**Code:**

```python
# Example of implementing workload-based scheduling with dynamic priorities

class QueryScheduler:
    def __init__(self):
        self.priority_queues = {1: [], 2: [], 3: []}  # Priorities 1-3, lower numbers are higher priority

    def add_query(self, query, priority):
        if priority in self.priority_queues:
            self.priority_queues[priority].append(query)

    def process_queries(self):
        for i in range(1, 4):  # Process queues from highest to lowest priority
            while self.priority_queues[i]:
                query = self.priority_queues[i].pop(0)
                if monitor_resource_usage():  # Simulate resource monitoring
                    execute_query(query)
```

**Example input / output:**

```text
Input:
- Queries with varying priorities and durations

Output:
- Queries processed in order of priority, ensuring high-priority queries are executed first.
- Dynamic adjustment of resource allocation based on real-time monitoring.
```

**Data:**

```json
{
    "queries": [
        {"id": 1, "priority": 3, "duration": 5},
        {"id": 2, "priority": 1, "duration": 2},
        {"id": 3, "priority": 2, "duration": 4}
    ]
}
```

**Flow:**

1. Query Scheduler
2. Monitor Resource Usage
3. Adjust Priorities
4. Process High-Priority Queues
5. Process Lower Priority Queues

