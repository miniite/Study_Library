---
type: "practice"
difficulty: "Deep"
---

# You need cross-AZ GPU-to-GPU communication for training but the data-transfer cost is huge. Explain how AZ placement affects both training-network cost and fault tolerance, and how you'd co-locate a training job in one AZ without losing resilience.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of cross-AZ GPU-to-GPU communication for distributed machine learning (ML) training, we need to carefully consider both the cost of data transfer and fault tolerance. The placement of GPUs across Availability Zones (AZs) will have significant implications on these factors. This response will outline how AZ placement affects network costs and resilience, and propose strategies to co-locate training jobs while maintaining high availability.

In essence, placing GPUs in different AZs can provide better fault tolerance but at the cost of increased data transfer latency and higher network bandwidth requirements. Conversely, colocating GPUs within a single AZ reduces network overhead but may compromise fault isolation. We will explore how to leverage these trade-offs effectively.

**Comparison:**

```
AZ	GPUs	Network Cost (per 2GB)
AZ1-2	4+5=9	$0.02
AZ1-3	4+6=10	$0.10
AZ2-3	5+6=11	$1.10
```

**Example input / output:**

```text
# Example input-output for network cost calculation

def calculate_network_cost(data_size, latency):
    return data_size * latency * Cost_per_GB

# Assuming training job size is 2GB and requires 3 GPU-to-GPU transfers between AZ1 and AZ3
transfer_cost_AZ1_to_AZ3 = calculate_network_cost(2, Latency_between_AZs)
print(f"Network cost for transfer: {transfer_cost_AZ1_to_AZ3} units")
```

**Data:**

```json
{
  "AZs": ["AZ1", "AZ2", "AZ3"],
  "GPUs_per_AZ": [4, 5, 6],
  "Cost_per_GB": 0.01,
  "Latency_AZ_within": 0.1,
  "Latency_between_AZs": 1.0
}
```

**Flow:**

1. Yes
2. No
3. AZ Placement
4. GPUs in Same AZ?
5. Risk of Single Point Failure,Low Network Cost
6. HIGH NETWORK COST, HighFault Tolerance
7. Colocate Training Jobs withinSame AZ for Efficiency andResilience
8. Ensure Multi-AZ Redundancy,Use Network OptimizationTechniques

