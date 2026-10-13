---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right autoscaling metric for a worker fleet whose real pressure is queue backlog rather than CPU, and what publishing-lag pitfalls would you watch for?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Autoscaling Metrics](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on the right autoscaling metric for a worker fleet that operates based on queue backlog rather than CPU utilization, the primary goal is to ensure efficient resource allocation that directly addresses the workload demand. In this scenario, we need to carefully choose an autoscaling strategy that reacts appropriately to changes in the backlog of tasks waiting to be processed. Additionally, we must be vigilant about potential pitfalls related to publishing lag and how it can affect our scaling decisions.

At a high level, the key idea is to use custom metrics from monitoring tools or directly from the queue system itself as the autoscaling trigger rather than relying on CPU utilization. This allows for more precise resource management that aligns with actual workload needs. We should also monitor publishing lag to avoid false positives and ensure our scaling actions are well-informed.

To begin, let's break down the core of the question: we need an autoscaling metric that reflects queue backlog pressure, not just CPU usage. The most appropriate metrics for this would be:

Queue Length: The number of tasks in the queue.
Queue Latency: Time taken to process a task from the moment it enters the queue until it starts being processed.

These metrics provide direct insight into the real pressure on our worker fleet, which is crucial for accurate autoscaling decisions.

We will now walk through the steps to choose and implement this metric effectively:

Identify Suitable Metrics:

Queue Length: This gives a clear count of tasks waiting to be processed.
Queue Latency: This helps us understand how long it takes to process each task, providing insights into both backlog pressure and processing efficiency.

Set Thresholds for Autoscaling Actions:

Define thresholds based on observed historical data or predefined service level objectives (SLOs). For example, if the queue length exceeds 100 tasks or average latency surpasses 5 minutes, trigger scaling actions.

Monitor and Validate Metrics:

Continuously monitor these metrics using tools like Prometheus, Grafana, or custom dashboards. Ensure that they accurately reflect the true workload state.

Implement Autoscaling Policies:

Use cloud provider services like AWS Auto Scaling, Azure Autoscale, or Google Kubernetes Engine (GKE) to automatically scale based on the chosen metrics.
Configure policies to add or remove worker instances when thresholds are met.

Publishing Lag Considerations:

Be cautious of potential publishing lag issues where data from the queue system might not immediately reflect current state. For example, if there is a delay in updating the metric due to network latency or processing delays.
Implement retries and backoff strategies for metrics updates to ensure consistency.

Evaluate Performance and Adjust:

Regularly review performance metrics after implementing autoscaling policies. Make adjustments as needed based on observed outcomes.

By focusing on queue backlog rather than CPU, we can achieve more accurate and efficient resource management. However, it’s crucial to carefully monitor publishing lag to avoid scaling based on stale data or false positives.

**Comparison:**

```
Metric	Description	Thresholds
Queue Length	Number of tasks in queue	>100 tasks
Average Latency	Time to process a task from queue entry	>5 minutes
```

**Code:**

```python
import boto3

# Initialize an Auto Scaling client
as_client = boto3.client('autoscaling')

def scale_up_queue_length(queue_length_threshold):
    response = as_client.scale_in_auto_scaling_group(
        AutoScalingGroupName='worker-fleet',
        DesiredCapacity=desired_capacity + 1
    )
    print(response)

def scale_down_queue_length(current_queue_length, queue_length_threshold):
    if current_queue_length < queue_length_threshold:
        response = as_client.scale_out_auto_scaling_group(
            AutoScalingGroupName='worker-fleet',
            DesiredCapacity=desired_capacity - 1
        )
        print(response)
```

**Example input / output:**

```

Input:

Current queue length: 210 tasks, Average latency: 5 minutes.

Output:

Auto Scaling group 'worker-fleet' scales up by one instance.
JSON
Copy
{
  "scale_up": {
    "response_code": 200,
    "message": "Scaling group worker-fleet scaled up successfully."
  }
}
```

**Data:**

```json
{
  "queue_metrics": {
    "queue_length": 200,
    "average_latency": 4.5
  }
}
```

**Flow:**

1. Yes
2. No
3. Latency > 5min?
4. Yes
5. No
6. Queue Monitor
7. Queue Length > 100?
8. Scale Up Auto Scaling Group
9. Monitor Latency
10. Scale Down Auto ScalingGroup
11. Adjust Desired Capacity - 1
12. Stay at Current Scale

