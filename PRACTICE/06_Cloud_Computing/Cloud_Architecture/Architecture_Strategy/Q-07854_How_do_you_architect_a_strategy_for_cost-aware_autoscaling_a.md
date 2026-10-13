---
type: "practice"
difficulty: "Expert"
---

# How do you architect a strategy for cost-aware autoscaling and capacity across spot, on-demand, and committed capacity for a spiky workload like ticketing or IPL streaming?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compute Placement Strategy](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Compute_Placement_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a strategy for cost-aware autoscaling and capacity management in a spiky workload like ticketing or IPL streaming, we need to balance between ensuring service availability at all times while keeping costs under control. The core idea is to leverage different types of cloud instances (spot, on-demand, committed) based on their respective costs and reliability trade-offs.

First, let's triage the question: The interviewer is probing how you would design a hybrid solution that optimizes for cost without sacrificing service availability. You'll need to consider multiple factors including instance pricing models, auto-scaling policies, and monitoring mechanisms.

We start from first principles:

Understand the Workload: Spiky workloads have varying demand over time, with peaks and troughs.
Instance Types:
On-Demand Instances: High reliability but higher cost.
Spot Instances: Lower cost but lower reliability due to potential forced termination.
Committed Instances (Reserved Instances): Lower cost with long-term commitment.
Auto-Scaling Strategy: Implement policies that dynamically adjust capacity based on real-time demand.
Monitoring and Alerting: Set up comprehensive monitoring to detect anomalies and ensure service availability.

Let's break down each step:

Determine Baseline Capacity:
Identify the minimum number of instances required at low-demand times (base load).
Spot Instance Utilization for Peak Handling:
Configure auto-scaling groups that utilize spot instances during peak demand when on-demand instances are too expensive.
On-Demand and Committed Instances as a Buffer:
Use on-demand or committed instances to handle the core service, ensuring high availability even if spot instances get terminated.
Auto-Scaling Policies:
Define scaling policies based on metrics like CPU usage, network traffic, and queue lengths.
Cost Optimization Strategy:
Implement a cost optimization strategy that prioritizes using spot instances during off-peak times while ensuring service availability.

To illustrate with an example scenario:

Suppose we have a ticketing application where peak demand can spike to 10x the normal load. We start by setting up a minimum of 5 on-demand instances for baseline capacity. During off-peak hours, we allow these instances to scale down. When demand spikes, auto-scaling triggers additional spot instances (up to a limit) while maintaining at least two on-demand and one committed instance as buffers.

Now, let's consider the architecture:

Spiky Workload
On-Demand Instances
Spot Instances (Auto-Scaling)
Committed Instances(Reserved)
Baseline Capacity
Peak Handling
Buffer Capacity

For monitoring, we set up dashboards that track key metrics:

YAML
metrics:
  - metric: CPUUtilization
    threshold: 80%
    action: ScaleUp
  - metric: NetworkTraffic
    threshold: 90% of max bandwidth
    action: ScaleDown

Finally, common mistakes to avoid include:

Over-relying on spot instances without sufficient buffer.
Ignoring cost optimization entirely.
Not setting up proper monitoring and alerting mechanisms.

**Comparison:**

```
Metric	Threshold	Action
CPUUtilization	80%	ScaleUp
NetworkTraffic	90% of max bandwidth	ScaleDown
```

**Code:**

```python
# Example Python code for auto-scaling policy
def scale_instances(demand_metric):
    if demand_metric > 90:
        return 'ScaleUp'
    elif demand_metric < 40:
        return 'ScaleDown'
    else:
        return 'Maintain'
```

**Example input / output:**

```

Input (Demand Metric): 95% Output (Action): ScaleUp

Input (Demand Metric): 30% Output (Action): ScaleDown

Request:

JSON
Copy
{
  "demand_metric": 70
}

Response:

JSON
Copy
{
  "action": "Maintain"
}
```

**Data:**

```yaml
instance_types:
  - type: on-demand
    count: 5
  - type: spot
    max_count: 10
  - type: reserved
    count: 3
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Spiky Workload
6. Auto-Scaling
7. Demand Metric > 80%
8. ScaleUp
9. Demand Metric < 40%
10. ScaleDown
11. Maintain

