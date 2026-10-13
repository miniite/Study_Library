---
type: "practice"
difficulty: "Deep"
---

# Your inference GPU sits at 15% utilization because requests trickle in one at a time. How would you use dynamic batching and cloud autoscaling to raise utilization without hurting latency SLOs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Autoscaling Metrics](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of low GPU utilization while ensuring service-level objectives (SLOs) on latency, we need to balance between maximizing resource usage and maintaining performance. The key idea is to use dynamic batching in conjunction with cloud autoscaling to dynamically adjust batch sizes based on incoming request patterns.

#### Triaging the Question

The question asks how to improve GPU utilization from 15% without compromising latency SLOs. This involves understanding the current bottlenecks, identifying opportunities for optimization, and implementing a solution that can adapt to varying load conditions.

#### Assumptions
The service is CPU-bound or has other limitations besides GPU constraints.
Latency SLOs are critical and must be maintained.
The cloud environment supports autoscaling and dynamic batching techniques.

#### First Principles Solution
Dynamic Batching: Aggregate multiple requests into a single batch to utilize the GPU more efficiently. This reduces the overhead of starting and stopping inference processes for each individual request.
Cloud Autoscaling: Dynamically adjust the number of instances based on demand to ensure that resources are allocated where needed while keeping costs low during periods of lower load.
Step-by-Step Solution

Analyze Current Load Pattern:

Monitor the request rate and GPU utilization over time to understand the patterns.
Use metrics such as requests per second (RPS), GPU utilization percentage, and latency to gain insights into current performance.

Implement Dynamic Batching:

Configure the inference pipeline to dynamically batch requests based on real-time load conditions. For example, if the RPS is low, batch more requests; if high, reduce the batch size.
Ensure that batching does not significantly increase latency. For instance, if a request takes 10ms individually and batches of 5 requests take 30ms, the effective per-request latency should still be close to 6ms.

Autoscaling Strategy:

Set up autoscaling policies for both CPU and GPU instances. Adjust the number of instances based on metrics such as RPS and resource utilization.
Use a hybrid approach that considers both CPU and memory usage alongside GPU load to make more informed scaling decisions.

Monitoring and Feedback Loops:

Continuously monitor system performance using dashboards like CloudWatch or Prometheus.
Implement feedback loops where the system automatically adjusts batch sizes and instance counts based on real-time data.

#### Closing Summary

By implementing dynamic batching and cloud autoscaling, we can achieve higher GPU utilization while maintaining stringent latency SLOs. This approach involves monitoring current load patterns, configuring systems to adapt dynamically, and continuously optimizing for both performance and cost efficiency.

#### Common Mistakes/Follow-Ups
Over-batching: If batch sizes are too large, it could increase latency significantly.
Under-scaling: Failing to scale down instances during periods of low demand can lead to unnecessary costs.
Latency Compromise: Ensuring that batching and autoscaling do not compromise SLOs is crucial; this might require fine-tuning the batch size thresholds.

Would you like to explore any specific aspects further, such as how to implement dynamic batching or design an autoscaling policy?

**Comparison:**

```
Time	RPS (Requests/Second)	GPU Utilization (%)
09:00	15	12
09:01	25	30
09:02	40	60
09:03	20	45
09:04	10	20
```

**Code:**

```python
# Example of a simple dynamic batching implementation in Python

def dynamic_batching(requests):
    max_batch_size = 50  # Maximum number of requests per batch
    current_batch = []

    for request in requests:
        if len(current_batch) < max_batch_size:
            current_batch.append(request)
        else:
            yield current_batch  # Send the batch to GPU processing
            current_batch = [request]

    if current_batch:  # Yield any remaining batch
        yield current_batch

# Example of an autoscaling policy using AWS Lambda and CloudWatch
def lambda_handler(event, context):
    rps_threshold = event['RPS']
    instance_count = get_instance_count()  # Custom function to fetch current instance count

    if rps_threshold > 100:  # Example threshold for scaling up
        scale_up_instances(3)  # Scale up by 3 instances
    elif rps_threshold < 50 and instance_count > 2:  # Example threshold for scaling down
        scale_down_instances(2)  # Scale down by 2 instances

# Custom functions to interact with CloudWatch metrics
def get_instance_count():
    # Placeholder function to fetch current instance count from CloudWatch
    pass

def scale_up_instances(number_of_instances):
    # Function to scale up AWS EC2 instances
    pass

def scale_down_instances(number_of_instances):
    # Function to scale down AWS EC2 instances
    pass
```

**Example input / output:**

```text
# Example input and output for dynamic batching

Input:
[
    {"request_id": 1, "data": "data_1"},
    {"request_id": 2, "data": "data_2"},
    {"request_id": 3, "data": "data_3"}
]

Output (Batched):
[
    {"batch_id": 1, "requests": [{"request_id": 1, "data": "data_1"}, {"request_id": 2, "data": "data_2"}]},
    {"batch_id": 2, "requests": [{"request_id": 3, "data": "data_3"}]}
]
```

**Data:**

```json
{
  "requests": [
    {"timestamp": "2023-10-01T09:00:00Z", "value": 5},
    {"timestamp": "2023-10-01T09:01:00Z", "value": 8},
    {"timestamp": "2023-10-01T09:02:00Z", "value": 12},
    {"timestamp": "2023-10-01T09:03:00Z", "value": 6},
    {"timestamp": "2023-10-01T09:04:00Z", "value": 4}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monitor RPS and GPUUtilization
6. RPS > Threshold?
7. Scale Up Instances
8. RPS < Threshold?
9. Scale Down Instances
10. Continue Monitoring

