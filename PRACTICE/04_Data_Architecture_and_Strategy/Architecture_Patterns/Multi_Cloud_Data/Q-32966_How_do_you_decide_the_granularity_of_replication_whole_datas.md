---
type: "practice"
difficulty: "Expert"
---

# How do you decide the granularity of replication — whole datasets, changed partitions, or CDC row-level — for a cross-cloud analytics copy, and what does each cost you to keep consistent?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Hub-and-spoke Cross-cloud Networking](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Hub-and-spoke_Cross-cloud_Networking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on the granularity of replication in a multi-cloud data architecture, we need to balance the trade-offs between consistency, performance, and cost. The options typically considered are whole dataset replication, changed partitions, or CDC (Change Data Capture) row-level changes. Each approach has its own implications for consistency and resource usage.

The core idea is to choose a replication strategy that meets the business requirements while minimizing operational overhead. For example, in a large-scale data warehouse scenario, you might prioritize minimal latency at the cost of higher compute resources, whereas in a near-real-time stream processing use case, every change needs immediate propagation across clouds.

**Comparison:**

```
Data Replication Options
Whole Dataset
High Cost, High Consistency
Changed Partitions
Medium Cost, Medium Latency
CDC Row-Level Changes
Low Cost, Minimal Latency butIncreased Complexity
```

**Code:**

```python
# Example pseudo-code for setting replication granularity based on business rules

def determine_replication_granularity(data_size, real_time_requirements):
    if data_size > 100GB and real_time_requirements:
        return "CDC"
    elif data_size < 5GB and not real_time_requirements:
        return "changed_partitions"
    else:
        return "whole_dataset"

# Example usage
data_size = 75GB
real_time_requirements = True
granularity = determine_replication_granularity(data_size, real_time_requirements)
print(f"Recommended granularity: {granularity}")
```

**Example input / output:**

| Input (Whole Dataset) | Output (CDC Row-Level Change) |
|-----------------------|------------------------------|
| Data Warehouse Update | CDC Event                    |
| Timestamp: 2023-10-01T15:00:00Z | Changed Product A price to $14.99 on 2023-10-01T15:00:00Z |

**Data:**

```yaml
# Sample data schema for a cross-cloud analytics dataset

data:
  - id: 123456
    name: "Product A"
    price: 9.99
    timestamp: "2023-10-01T09:00:00Z"

changes:
  - id: 789012
    operation: update
    field: "price"
    new_value: 14.99
    old_value: 9.99
    timestamp: "2023-10-01T15:00:00Z"
```

**Flow:**

1. High Consistency, High Cost
2. Medium Consistency,Medium Cost
3. Low Cost, Minimal Latencybut Increased Complexity
4. Whole Dataset
5. (High Resource Usage)
6. Changed Partitions
7. (Reduced Compute Overhead)
8. CDC Row-Level Changes
9. (Real-Time Processing)

