---
type: "practice"
difficulty: "Deep"
---

# Your autoscaler scales GPU inference replicas on CPU utilization but the workload is actually GPU-memory-bound behind slow batching. Explain why this scaling signal is wrong and what metric (queue depth, batch latency, GPU util) you'd scale on instead.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Workload Isolation on Shared Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Workload_Isolation_on_Shared_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of autoscaling based on CPU utilization in a scenario where the workload is actually GPU-memory-bound due to slow batching, we need to understand why CPU utilization can be misleading and identify more appropriate metrics like queue depth or batch latency. This problem arises because the bottleneck lies in the GPU memory bandwidth rather than the CPU performance.

#### Triaging the Question

The question probes our understanding of how distributed systems operate under various constraints and the importance of choosing relevant metrics for autoscaling. We need to assume that we are dealing with a typical machine learning inference scenario where multiple replicas are running on different nodes, each handling requests from an external service or batch jobs.

#### First-Principles Solution

Understanding the Bottleneck:

In our scenario, slow batching means that data is processed in smaller batches than necessary due to constraints like GPU memory bandwidth.
This results in lower GPU utilization because the GPUs are waiting for new batches of data rather than being fully utilized.

Why CPU Utilization is Misleading:

CPUs often have more cores and higher parallelism compared to GPUs, which means that even if the CPUs are underutilized, the bottleneck might still be on the GPUs.
The autoscaler based on CPU utilization will not detect this issue because it’s looking at the wrong metric. The CPU utilization could remain high while GPU memory is a limiting factor.

Proposed Metrics:

Queue Depth: This measures how many requests are waiting to be processed by the GPUs. High queue depth indicates that there is more data available but not being processed quickly enough.
Batch Latency: This measures the time between when a batch of data arrives and when it is fully processed by the GPU. High latency suggests inefficiencies in the data transfer or processing pipeline.

Choosing the Metric:

Queue Depth: If we observe that the queue depth is consistently high, it indicates that there is an issue with the batching process.
Batch Latency: By monitoring batch latency, we can identify whether the problem lies in slow data transfer (e.g., network issues) or in the GPU processing speed.

#### Closing Summary
and Common Mistakes

In conclusion, the autoscaler should be based on metrics that reflect the actual bottleneck, which is the GPU memory bandwidth due to slow batching. Metrics like queue depth and batch latency provide a better indication of when additional resources are needed. Missteps include relying solely on CPU utilization or other metrics that do not capture the true state of resource utilization in distributed systems.

#### Common Mistakes
Over-reliance on CPU Utilization: Failing to recognize that GPU memory bandwidth is the limiting factor.
Ignoring Queue Depth and Batch Latency: Not monitoring these key performance indicators (KPIs) can lead to suboptimal scaling decisions.
Insufficient Data Collection: Inadequate logging or monitoring can make it difficult to identify the root cause of performance issues.

**Comparison:**

```
Request ID	Data Size (MB)
1	500
2	700
3	600
Metric	Value
Queue Depth	3
Batch Latency	1.5 seconds
```

**Code:**

```python
# Example code snippet for collecting metrics
import time

def monitor_queue_depth():
    queue_size = len(queue)  # Assume queue is a list of incoming requests
    return queue_size

def measure_batch_latency(start_time):
    end_time = time.time()
    batch_processing_time = end_time - start_time
    return batch_processing_time

# Example usage
queue_depth = monitor_queue_depth()
batch_latency = measure_batch_latency(start_time)
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "requests": [
    {"id": 1, "data_size": 500},
    {"id": 2, "data_size": 700},
    {"id": 3, "data_size": 600}
  ]
}

Output:

JSON
Copy
{
  "queue_depth": 3,
  "batch_latency": 1.5
}
```

**Data:**

```json
{
  "requests": [
    {"id": 1, "data_size": 500},
    {"id": 2, "data_size": 700},
    {"id": 3, "data_size": 600}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive Requests
6. Monitor Queue Depth
7. Queue Depth > Threshold?
8. Scale Out GPUs
9. Monitor Batch Latency
10. Batch Latency > Threshold?
11. Scale Out GPUs
12. Do Nothing

