---
type: "practice"
difficulty: "Expert"
---

# Your inference fleet could run cheaper on spot GPUs but a preemption mid-generation ruins user experience. Design the hybrid spot/on-demand GPU serving architecture and the cost-versus-UX policy that governs the mix.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Capacity Economics](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/GPU_Capacity_Economics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to design an architecture that balances cost efficiency with user experience by intelligently managing a hybrid fleet of on-demand and spot GPUs. The core idea is to use spot instances when the application can tolerate preemption, but switch to on-demand instances during critical times to ensure consistent performance.

#### Triage
Understanding the Problem: We need an architecture that allows dynamic switching between spot and on-demand GPU resources based on user experience requirements.
Key Components:
GPU Resources: Spot and On-Demand instances.
Cost Management System: To monitor costs and enforce cost policies.
User Experience Metrics: To determine when to switch resource types.

#### Assumptions
Users have defined SLAs (Service Level Agreements) for minimum performance requirements.
The system can dynamically detect preemption events on spot instances.

#### Solution
Step 1: Architecture Design

We will design a hybrid GPU serving architecture that includes:

A load balancer to distribute requests across available GPUs.
An intelligent scheduler to manage the allocation of resources based on cost and user experience metrics.
A monitoring system to track resource usage, costs, and SLA compliance.
Step 2: Intelligent Scheduler

The scheduler will make decisions based on real-time metrics such as GPU utilization, preemption events, and current cost. It should:

Monitor Preemption Events: Detect when a spot instance is preempted.
Switch to On-Demand: Immediately switch the workload to an on-demand instance if a critical SLA is at risk.
Restore Cost Savings: If preemption does not impact user experience, allow the system to use spot instances again.
Step 3: Cost Versus UX Policy

We will implement policies that balance cost and user experience:

Baseline Cost Monitoring: Track historical costs and set a baseline for acceptable spend.
Dynamic Thresholds: Set thresholds based on SLA violations. If GPU performance drops below a certain threshold, prioritize on-demand instances.
Cost-Optimization Algorithm: Implement an algorithm that determines the optimal mix of spot and on-demand instances based on current cost and user experience.
Step 4: User Experience Metrics

Define metrics to measure user experience:

Response Time: Average response time for requests.
Error Rate: Number of errors or failed requests.
Latency: Latency between request and response.
Throughput: Requests per second handled by the system.

#### Closing Summary

By implementing a hybrid GPU serving architecture with an intelligent scheduler and cost-versus-UX policies, we can achieve significant cost savings while maintaining high user experience standards. The key is to dynamically switch resources based on real-time metrics to ensure that critical SLAs are met without unnecessary expense.

#### Common Mistakes/Follow-Ups
Failing to Monitor Preemption Events: Not properly detecting when spot instances are preempted can lead to prolonged service disruptions.
Ignoring User Experience Metrics: Overemphasizing cost savings at the expense of user experience can result in customer dissatisfaction.
Lack of Dynamic Policy Adjustment: Static policies may not adapt well to changing conditions, leading to suboptimal resource utilization.

**Comparison:**

```
SLA Metrics	Thresholds	Consequences
Latency	> 200 ms	Service Interruption
Response Time	> 500 ms	Increased Error Rate
Cost	> 1.75 * Baseline	Prioritize On-Demand Instances
```

**Code:**

```python
# Example policy implementation for dynamic threshold adjustment
class CostUXPolicy:
    def __init__(self, baseline_cost, max_latency=50):
        self.baseline_cost = baseline_cost
        self.max_latency = max_latency

    def evaluate(self, current_cost, response_time):
        if (current_cost > 1.5 * self.baseline_cost) or (response_time > self.max_latency):
            return "Switch to on-demand"
        else:
            return "Use spot instances"

# Example scheduler decision logic
def schedule_resources(policy, request_type):
    decision = policy.evaluate(current_cost(request_type), response_time(request_type))
    if decision == "Switch to on-demand":
        use_on_demand_instance()
    else:
        use_spot_instance()
```

**Example input / output:**

```text
Request Type: Inference Request
- Current Cost:
  - Spot Instance: $0.4 per hour
  - On-Demand Instance: $2.0 per hour
- Response Time:
  - Spot Instance: 80ms
  - On-Demand Instance: 30ms

Policy Evaluation:
- Baseline Cost (1.5 * $0.5) = $0.75
- Current Cost ($0.4) < $0.75 and Response Time (80ms) < 100ms

Decision: Use Spot Instances
```

**Data:**

```json
{
  "baseline_cost": 0.5,
  "max_latency": 100,
  "current_cost": {
    "spot": 0.4,
    "on-demand": 2.0
  },
  "response_time": {
    "spot": 80,
    "on-demand": 30
  }
}
```

**Flow:**

1. No
2. Yes
3. Cost Exceeds Threshold
4. Cost Below Threshold
5. Monitor GPU Utilization andPreemption
6. Is Latency Exceeded?
7. Use Spot Instances
8. Switch to On-Demand Instance
9. Evaluate Cost and ResponseTime
10. Prioritize On-Demand
11. Monitor Continuously

