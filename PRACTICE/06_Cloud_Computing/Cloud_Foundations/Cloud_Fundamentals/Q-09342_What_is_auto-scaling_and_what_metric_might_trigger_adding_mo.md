---
type: "practice"
difficulty: "Basic"
---

# What is auto-scaling and what metric might trigger adding more model-serving instances during peak traffic?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Autoscaling Metrics](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Auto-scaling is a crucial feature in cloud infrastructure that automatically adjusts the number of running instances based on demand to ensure optimal resource utilization and performance. For ML model serving, auto-scaling might trigger additional instance deployment when there's an increase in request volume or latency surpasses a certain threshold.

#### Triage
The question asks about the concept of auto-scaling and what metrics could trigger scaling up (increasing instances) for an ML model serving system during peak traffic. The interviewer is probing your understanding of cloud infrastructure, resource management, and how to handle varying workloads efficiently.

Step-by-First Principles Solution:

Understanding Auto-Scaling:

Auto-scaling is a feature in cloud platforms like AWS (Elastic Load Balancing), Azure (Auto Scaling), or Google Cloud (Autoscaler) that dynamically adjusts the number of running instances based on predefined metrics.

Metrics for Triggering Scaling Up:

Request Volume: If the number of incoming requests per second exceeds a certain threshold, it might trigger additional instance deployment to handle the load.
Latency: If the response time (latency) starts increasing beyond an acceptable level due to high traffic, it can signal the need for more resources to maintain performance.
CPU Utilization: High CPU usage could indicate that the current instances are overloaded and cannot process requests efficiently, necessitating additional capacity.

#### Example Scenario

Imagine a scenario where during a holiday season, an e-commerce website experiences a surge in traffic. The ML model serving this data would need to handle more requests for product recommendations. If auto-scaling is enabled, the system will automatically add more instances to distribute the load and maintain performance.

#### Trade-offs

While auto-scaling ensures that resources are allocated based on demand, it also involves costs associated with maintaining additional instances during periods of low traffic.
Monitoring and setting up these scaling policies require careful consideration of cost-effectiveness and user experience.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, auto-scaling is a key aspect of cloud infrastructure management that helps in efficiently managing resources. For ML model serving, it's crucial to set appropriate metrics like request volume or latency thresholds to ensure optimal performance under varying loads. A common mistake is setting overly aggressive scaling policies without proper monitoring, which can lead to unnecessary costs.

#### Follow-Ups

How do you balance the trade-offs between cost and performance in auto-scaling?
Can you explain how different cloud providers implement auto-scaling for ML model serving?

**Comparison:**

```
Metric	Threshold	Action Triggers
Request Volume	> 5000 RPS	Scale up
CPU Utilization	> 80%	Scale up if request volume also high
```

**Code:**

```python
# Example code snippet for setting up an autoscaling policy (pseudocode)
autoscaler_config = {
    "min_instances": 2,
    "max_instances": 10,
    "cpu_threshold": 80,  # 80% CPU utilization triggers scaling
    "request_volume_threshold": 5000  # Scale up if more than 5000 requests/sec
}
```

**Example input / output:**

| Input | Output |
|-------|--------|
| 1500 requests/second, CPU utilization at 70% | No scaling action taken (current instances are sufficient) |
| 6000 requests/second, CPU utilization at 90% | Scaling up to max_instances (10 instances) triggered |
| 4000 requests/second, CPU utilization at 85% | Scaling down to min_instances (2 instances) triggered |

**Data:**

```json
{
  "autoscaler_config": {
    "min_instances": 2,
    "max_instances": 10,
    "cpu_threshold": 80,
    "request_volume_threshold": 5000
  }
}
```

**Flow:**

1. Yes
2. No
3. High Request Volume
4. CPU Utilization High
5. Scale Up?
6. Add Instances
7. Monitor CPU and Requests

