---
type: "practice"
difficulty: "Deep"
---

# Your data platform must support both interactive sub-second dashboards and hour-long batch jobs on the same physical cluster, and they contend for I/O. As principal, how do you architect physical and logical isolation (separate compute pools, storage tiering, query admission control) so neither workload starves the other under peak load?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Data Platform Isolation](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Multi-tenant_Data_Platform_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of coexisting interactive sub-second dashboards and hour-long batch jobs on a shared physical cluster, we need to architect both logical and physical isolation strategies. This involves separating compute resources, optimizing storage tiers, and implementing query admission control mechanisms. By doing so, we ensure that neither workload starves the other during peak load conditions.

#### Triaging the Question

The core of this question is about designing a robust data architecture that can handle multiple workloads efficiently while minimizing contention between them. The key aspects to consider are:

Logical Isolation: Ensuring that different types of queries (interactive and batch) do not interfere with each other.
Physical Isolation: Allocating dedicated resources for each type of workload.
Storage Tiering: Managing data storage effectively to balance access patterns.
Query Admission Control: Controlling the number of concurrent queries based on resource availability.

#### Assumptions
and First Principles
Both workloads require significant I/O operations, which can lead to contention if not managed properly.
The cluster has a fixed set of resources (CPU, memory, storage).
Interactive dashboards have strict SLAs for latency (sub-second response time), while batch jobs allow longer execution times but should complete within hours.

#### Solution
Logical and Physical Isolation
1. Logical Isolation

To achieve logical isolation, we can use different databases or schemas to separate the interactive and batch workloads. For example:

Create a dashboards schema for real-time queries.
Create a batches schema for long-running analytics.

This separation ensures that the query planner does not mix these two types of queries, which helps in managing their execution independently.

2. Physical Isolation

To ensure physical isolation, we can allocate dedicated compute pools:

Allocate a high-performance computing (HPC) pool for interactive dashboards.
Reserve a batch processing pool for long-running jobs.

By doing this, we can control the resources each workload uses and prevent one from overloading another.

3. Storage Tiering

Implement a multi-tier storage strategy to optimize I/O performance:

Use SSDs for the dashboards schema where sub-second response times are critical.
Store less frequently accessed data in HDDs or S3-like object stores within the batches schema.

This ensures that interactive queries get faster access to their required data, while batch jobs can utilize cheaper storage with acceptable performance degradation during I/O-intensive operations.

4. Query Admission Control

To manage the number of concurrent queries and prevent resource starvation:

Implement a query admission control mechanism using queuing systems or API gateways.
Use resource pools and quotas to limit the number of interactive queries running at any given time.
Prioritize critical dashboards over less critical batch jobs during peak times.

#### Closing Summary

By employing logical and physical isolation strategies, we can effectively manage the coexistence of real-time dashboards and long-running batch jobs on a shared cluster. Logical separation ensures that different types of queries do not interfere with each other, while physical isolation allocates dedicated resources for each workload type. Storage tiering optimizes data access patterns to meet performance SLAs, and query admission control mechanisms ensure resource balance during peak load conditions.

#### Common Mistakes/Follow-ups
Ignoring Logical Isolation: Failing to separate queries into different schemas or databases can lead to contention between workloads.
Insufficient Physical Isolation: Not allocating dedicated resources for each type of workload can result in one starving the other, leading to performance degradation.
Neglecting Storage Tiering: Poor storage management can degrade overall system performance and violate SLAs.
Lack of Query Admission Control: Without mechanisms to control query concurrency, both interactive dashboards and batch jobs may suffer from resource contention.

**Comparison:**

| Schema | Storage Tier |
|--------|-------------|
| dashboards | SSD         |
| batches  | HDD         |
| Compute Pool   | Type        | Quota |
|----------------|------------|-------|
| interactivePool | high-performance | 30    |
| batchPool      | standard    | 20    |

**Code:**

```python
# Example code for simple admission control using a queuing mechanism
import threading

class QueryAdmissionControl:
    def __init__(self, max_concurrent_queries):
        self.max_concurrent_queries = max_concurrent_queries
        self.query_count = 0
        self.lock = threading.Lock()

    def admit_query(self):
        with self.lock:
            if self.query_count < self.max_concurrent_queries:
                self.query_count += 1
                return True
            else:
                return False

# Example usage
admission_control = QueryAdmissionControl(max_concurrent_queries=5)
if admission_control.admit_query():
    print("Query admitted")
else:
    print("Query denied due to resource limits")
```

**Example input / output:**

```text
# Example input to the admission control system
>>> admission_control = QueryAdmissionControl(max_concurrent_queries=5)
>>> if admission_control.admit_query():
...     print("Query admitted")
... else:
...     print("Query denied due to resource limits")

Query admitted

# Another example with a higher limit
>>> admission_control2 = QueryAdmissionControl(max_concurrent_queries=10)
>>> if admission_control2.admit_query():
...     print("Query admitted")
... else:
...     print("Query denied due to resource limits")

Query admitted
```

**Data:**

```json
{
  "schemas": {
    "dashboards": {
      "storageTier": "SSD"
    },
    "batches": {
      "storageTier": "HDD"
    }
  },
  "computePools": {
    "interactivePool": {
      "type": "high-performance",
      "quota": 30
    },
    "batchPool": {
      "type": "standard",
      "quota": 20
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Interactive Dashboard
6. Admit Query?
7. Run Query
8. Deny Query
9. Batch Job
10. Admit Query?
11. Run Batch Job
12. Deny Query

