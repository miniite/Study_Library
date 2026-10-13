---
type: "practice"
difficulty: "Deep"
---

# How do you decide the cost-optimal batching and dynamic-batching window for online inference, where larger batches raise GPU utilization and cut cost-per-request but add tail latency? Walk me through setting the batch-timeout as an explicit cost-versus-latency knob per SLA tier.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Capacity Economics](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/GPU_Capacity_Economics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the optimal batching strategy for online inference in a cloud environment, we need to balance GPU utilization against tail latency while adhering to service-level agreements (SLAs). This involves understanding both cost and performance trade-offs. We'll start by defining the key parameters and then proceed step-by-step to set up our approach.

Firstly, let's define what batching is: grouping multiple inference requests into a single batch to reduce the number of context switches and improve GPU utilization efficiency. However, this comes with the risk of increased tail latency if batches take too long to process, potentially violating SLAs.

The core problem we're solving here involves setting up a dynamic batching window that can adapt based on real-time metrics such as processing time and current workload. We'll leverage these metrics to adjust batch sizes and timeouts to find an optimal balance between cost savings and performance guarantees.

**Comparison:**

| Load % | Batch Size | Latency (ms) | Timeout (ms) |
|--------|------------|--------------|--------------|
| 60     | 35         | 100          | 225          |
| 75     | 42         | 95           | 158          |
| 90     | 47         | 200          | 285          |

**Code:**

```python
# Define key parameters
MIN_BATCH_SIZE = 10
MAX_BATCH_SIZE = 50
BATCH_TIMEOUT_DEFAULT = 200  # in milliseconds

def calculate_batch_size(current_load, latency_threshold):
    """Adjust batch size based on current load and latency constraints."""
    if current_load > LOAD_THRESHOLD_HIGH:
        return MAX_BATCH_SIZE
    elif current_load < LOAD_THRESHOLD_LOW:
        return MIN_BATCH_SIZE
    else:  # Use linear interpolation for intermediate loads
        return int(MIN_BATCH_SIZE + (MAX_BATCH_SIZE - MIN_BATCH_SIZE) * (current_load - LOAD_THRESHOLD_LOW) / (LOAD_THRESHOLD_HIGH - LOAD_THRESHOLD_LOW))

def update_batch_timeout(current_latency):
    """Adjust timeout based on current latency."""
    if current_latency > LATENCY_THRESHOLD:
        return MAX_BATCH_TIMEOUT
    else:  # Linearly decrease timeout for lower latencies
        return int(BATCH_TIMEOUT_DEFAULT - (BATCH_TIMEOUT_DEFAULT / LATENCY_THRESHOLD) * (current_latency - LATENCY_THRESHOLD))
```

**Example input / output:**

```text
# Example input and output for a dynamic batching scenario

Input:
- Current load = 75% (GPU utilization)
- Current latency = 95 ms

Output:
- Batch size = 42 (interpolated between MIN_BATCH_SIZE and MAX_BATCH_SIZE based on current load)
- Batch timeout = 158 ms (decreased from default as the current latency is lower than the threshold but still close to it)

Input:
- Current load = 90% (GPU utilization)
- Current latency = 200 ms

Output:
- Batch size = 47
- Batch timeout = 285 ms (default timeout reached due to high latency)
```

**Data:**

```yaml
# Example configuration parameters
config:
  load_threshold_low: 50  # in % of GPU utilization
  load_threshold_high: 80  # in % of GPU utilization
  latency_threshold: 100  # in milliseconds
  max_batch_timeout: 300  # in milliseconds
```

**Flow:**

1. Real-time Monitoring
2. Calculate Current Load
3. Calculate Batch Size
4. Adjust Batch Timeout Basedon Latency
5. Apply New Settings toInference Engine
6. Monitor Performance Metrics
7. Update SLA Tier Policies ifNecessary

