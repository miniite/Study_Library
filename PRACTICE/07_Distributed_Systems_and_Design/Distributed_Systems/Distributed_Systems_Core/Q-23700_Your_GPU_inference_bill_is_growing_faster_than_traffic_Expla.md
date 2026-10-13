---
type: "practice"
difficulty: "Deep"
---

# Your GPU inference bill is growing faster than traffic. Explain where you'd attack cost first: batching efficiency, quantization, right-sizing replicas, spot/preemptible GPUs, and how you'd make cost a first-class SLO without wrecking latency.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Workload Isolation on Shared Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Workload_Isolation_on_Shared_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the growing GPU inference costs while maintaining or even improving latency, we need to consider multiple dimensions of our system's architecture. The key areas to attack are batching efficiency, quantization, right-sizing replicas, and using spot/preemptible GPUs. We will also ensure that cost is treated as a first-class Service Level Objective (SLO) by monitoring it closely and adjusting strategies dynamically.

#### Triage

The core of the question revolves around optimizing GPU utilization to reduce costs while maintaining acceptable latency for inference tasks. The interviewer wants you to demonstrate your understanding of distributed systems, resource management, and how to balance cost with performance metrics. You should assume a typical cloud infrastructure setup where we can leverage various strategies to optimize.

#### First-Principles Solution
Batching Efficiency: By increasing the batch size, we reduce the overhead per inference request. This means that each GPU run handles more requests at once, thus amortizing the fixed costs over more tasks.
Quantization: Reducing the precision of model weights can significantly decrease memory and compute requirements without significant loss in accuracy. This allows us to use less powerful GPUs or achieve higher throughput with the same hardware.
Right-Sizing Replicas: Ensuring that we have the right number of replicas (both active and reserved) to handle peak loads efficiently is crucial. Over-provisioning can lead to wasted resources, while under-provisioning may cause excessive latency.
Spot/Preemptible GPUs: Utilizing spot instances for non-critical tasks or during off-peak hours can drastically reduce costs by leveraging cheaper GPU options that are available at a moment's notice.
Cost as an SLO

Cost should be treated like any other service-level objective (SLO). We need to monitor it closely and ensure it stays within acceptable bounds. Here’s how we would implement this:

Monitoring: Set up continuous monitoring of both cost and latency using tools like Prometheus, Grafana, or AWS CloudWatch.
Dynamic Adjustment: Use automated scripts or cloud-native services to dynamically adjust batch sizes, replica counts, and GPU types based on current load and cost metrics.
Cost Thresholds: Define thresholds for acceptable cost levels and alerting mechanisms when these are breached. This ensures proactive management of costs.
Step-by-Step Solution
Initial Baseline: Start by profiling the current system to understand baseline performance and resource usage.
Batch Size Optimization:
Increase batch sizes gradually while monitoring latency.
Use techniques like dynamic batching where requests are grouped based on similar inputs to maximize efficiency.
Quantization Evaluation:
Test different levels of quantization (e.g., INT8, INT4) and evaluate their impact on model performance and resource usage.
Employ techniques like knowledge distillation or post-training quantization to fine-tune models for better efficiency.
Right-Sizing Replicas:
Use auto-scaling policies to adjust the number of replicas based on real-time traffic patterns.
Implement reserved instances where possible to balance cost and availability.
Spot Instance Utilization:
Reserve spot instances during non-peak hours for non-critical tasks or background processing.
Ensure that critical tasks always use on-demand instances to avoid service interruptions.

#### Closing Summary

By focusing on batching efficiency, quantization, right-sizing replicas, and using cost-effective GPU options, we can significantly reduce the overall inference costs without compromising performance. Treating cost as an SLO ensures that our optimizations are balanced against business requirements and maintain high availability.

#### Common Mistakes
and Follow-Ups
Ignoring Latency: Over-optimizing for cost might lead to unacceptable latency, especially in time-sensitive applications.
Not Monitoring Effectively: Without proper monitoring, it’s hard to gauge the impact of optimizations and ensure they are effective.
Over-Reliance on Spot Instances: Relying solely on spot instances can introduce unpredictability and service interruptions.

**Comparison:**

```
Task Type	Usage of Spot Instances
Background	Yes
Critical	No
```

**Code:**

```python
# Example code for dynamic batch size adjustment
import time

def adjust_batch_size(current_requests, max_latency):
    if len(current_requests) > (max_latency * 10):  # Simplified logic
        return current_requests // 5  # Decrease batch size by factor of 5
    else:
        return max(len(current_requests), 32)  # Ensure minimum batch size

# Example function to simulate auto-scaling replicas
def auto_scale_replicas(current_load, target_load):
    if current_load > (target_load * 1.2):  # Increase replicas when load is high
        print("Scaling out: Adding 5 more replicas")
    elif current_load < (target_load * 0.8):  # Decrease replicas when load is low
        print("Scaling in: Removing 3 replicas")

# Example of using spot instances for non-peak tasks
def use_spot_instances(task_type):
    if task_type == "background":
        return True  # Use spot instance for background processing
    else:
        return False  # Use on-demand instance for critical tasks
```

**Example input / output:**

```text
Input:
- Current number of requests = 20
- Max allowed latency = 5 seconds

Output:
- Batch size after adjustment = 6 (adjusted by factor of 5)
- Auto-scaling recommendation: No change in replicas

Request for background task type "background":
- Use spot instance? Yes
```

**Data:**

```json
{
  "current_requests": 20,
  "max_latency": 5.0,
  "batch_size": 32,
  "target_load": 100
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Monitor Current Load
8. Is current load > 120% oftarget?
9. Scale out by adding 5 replicas
10. Is current load < 80% oftarget?
11. Scale in by removing 3 replicas
12. No change in number ofreplicas
13. Monitor Batch Size
14. Current batch size > 200requests?
15. Reduce batch size to 50% ofcurrent
16. Keep current batch sizeunchanged

