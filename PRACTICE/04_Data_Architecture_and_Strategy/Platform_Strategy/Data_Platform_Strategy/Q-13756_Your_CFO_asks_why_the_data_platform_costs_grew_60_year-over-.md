---
type: "practice"
difficulty: "Expert"
---

# Your CFO asks why the data platform costs grew 60% year-over-year while data volume only grew 20% — how do you diagnose the decoupling and what FinOps levers do you pull first?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The CFO's concern about the significant disparity in growth between data volume and platform costs can be diagnosed by examining several key areas within our data platform architecture and operational practices. First, let’s break down what might be driving this cost increase and identify potential FinOps levers to address it.

#### Triaging the Question

The question is probing into understanding why despite a moderate 20% growth in data volume, there was a much larger 60% increase in platform costs. This suggests that something within our current setup or operational practices is inefficient, leading to higher costs per unit of data processed or stored.

#### Assumptions
and Context
Assumption 1: The reported data volumes are accurate.
Assumption 2: Cost metrics include all relevant expenses (e.g., storage, compute, data transfer) without double-counting.
Assumption 3: There have been no significant changes in the cost structure of our services (e.g., new service tiers or pricing).

#### First-Principles Solution
Capacity Utilization Analysis:
Check if we are fully utilizing existing capacity. If not, this suggests underutilized resources leading to higher per-unit costs.
Cost Breakdown:
Categorize the cost increases by components (storage, compute, data transfer). This can reveal which parts of the platform are contributing most to the increased expenses.
Resource Optimization:

#### Cons
ider implementing more efficient resource allocation policies or optimizing query workloads to reduce unnecessary processing and storage costs.
Benchmarking:
Compare our cost metrics with industry benchmarks or historical data from previous years to identify unusual patterns.
Step-by-Step Analysis
Capacity Utilization Check:
Review the current capacity utilization of our data storage and compute resources.
PLAINTEXT
Capacity Utilization:
Storage: 70% utilized out of a 90TB total
Compute: 50% utilized out of a 100 vCPU total
Cost Breakdown Analysis:
Examine the detailed cost report to see where costs are being incurred.
PLAINTEXT
Cost Breakdown:
Storage: $3,000 (45%)
Compute: $2,800 (41%)
Data Transfer: $1,000 (15%)
Other Costs: $700 (10%)
Resource Optimization Plan:
Suggest strategies like:
Implementing auto-scaling to better match demand.
Optimizing data retention policies.
Migrating workloads from more expensive services to cost-effective alternatives.

#### Closing Summary
and Follow-Ups
The high 60% growth in platform costs, despite only a 20% increase in data volume, indicates inefficiencies that need addressing. By analyzing capacity utilization, breaking down costs, and optimizing resources, we can significantly reduce per-unit costs.

#### Common mistakes
to avoid include not thoroughly understanding the cost components or assuming all increases are due to higher data volumes.

#### Common Mistakes
Failing to distinguish between storage and compute costs when looking for optimization opportunities.
Overlooking underutilized resources that could be better allocated.
Ignoring non-data-related costs like network usage, which can also significantly impact total expenses.

**Comparison:**

| Resource | Total Capacity (GB) | Used Capacity (GB) | Utilization (%) |
|----------|---------------------|--------------------|-----------------|
| Storage  | 90,000              | 63,000             | 70.0            |
| Compute  | 100 vCPUs           | 50 vCPUs           | 50.0            |

**Code:**

```python
# Example script to analyze capacity utilization
def analyze_capacity_utilization(storage_total_gb, storage_used_gb, compute_vcpus, compute_used_vcpus):
    storage_util = (storage_used_gb / storage_total_gb) * 100
    compute_util = (compute_used_vcpus / compute_vcpus) * 100
    return storage_util, compute_util

# Example data
storage_total_gb = 90_000
storage_used_gb = 63_000
compute_vcpus = 100
compute_used_vcpus = 50

utilization = analyze_capacity_utilization(storage_total_gb, storage_used_gb, compute_vcpus, compute_used_vcpus)
print(f"Storage Utilization: {utilization[0]:.2f}%")
print(f"Compute Utilization: {utilization[1]:.2f}%")
```

**Example input / output:**

```text
Input:
- Storage total capacity: 90,000 GB
- Storage used capacity: 63,000 GB
- Compute vCPUs total: 100
- Compute vCPUs used: 50

Output:
Storage Utilization: 70.0%
Compute Utilization: 50.0%
```

**Data:**

```json
{
  "storage": {
    "total_capacity_gb": 90_000,
    "used_capacity_gb": 63_000
  },
  "compute": {
    "vcpus_total": 100,
    "vcpus_used": 50
  }
}
```

**Flow:**

1. Yes
2. No
3. Capacity Utilization Analysis
4. Is Utilization High?
5. Optimize Resource Allocation
6. Check for UnderutilizedResources
7. Implement OptimizationStrategies
8. Consider Scaling or MigratingWorkloads

