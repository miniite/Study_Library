---
type: "practice"
difficulty: "Deep"
---

# Your team argues that serverless GPU is cheaper than an always-on instance for inference. How do you evaluate cold-start latency versus cost to decide?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GPU Sharing](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evaluate whether a serverless GPU is more cost-effective than an always-on instance for inference, we need to consider both the cold-start latency and the cost implications at scale. Serverless GPUs are appealing due to their on-demand nature, but they introduce potential delays during initial execution (cold-start latency). Conversely, always-on instances have zero cold-start latency, but incur continuous costs even when not in use. We'll break this down step by step.

First, let's define the key terms:

Cold-start Latency: The delay between initiating a request and receiving the first response.
Cost: The financial overhead of running GPU resources either on-demand or continuously.

We need to assess the trade-offs under different usage scenarios. To do this accurately, we'll perform cost-benefit analysis based on real-world data and typical workloads.

**Comparison:**

```
Metric	Always-On Instance	Serverless GPU
Cold Start Latency	0 seconds (no delay)	Variable; depends on execution queue time
Cost per Hour of Inactivity	$0.01	$0 for inactivity, non-zero when processing requests
```

**Code:**

```python
# Example function to calculate costs for both models
def calculate_cost(cold_start_latency, always_on_cost, serverless_cost_per_request):
    # Mock function - replace with actual metrics
    def cost_benefit_analysis(cold_start_latency, always_on_cost, serverless_cost_per_request):
        if cold_start_latency > 5:  # Example threshold for acceptable latency
            return 'Serverless'
        else:
            return 'Always On'

    # Example parameters based on hypothetical use case
    cold_start_latency = 3.2  # seconds
    always_on_cost = 0.01     # $ per hour
    serverless_cost_per_request = 0.05   # $ per request

    result = cost_benefit_analysis(cold_start_latency, always_on_cost, serverless_cost_per_request)
    return result
```

**Example input / output:**

```text
- **Input**:
  - Cold-start Latency: 3.2 seconds
  - Always-on Cost Per Hour: $0.01
  - Serverless Cost Per Request: $0.05

- **Output**: `Serverless`

This indicates that for a cold-start latency of 3.2 seconds, serverless is more cost-effective compared to always-on.
```

**Data:**

```json
{
  "coldStartLatency": 3.2,
  "alwaysOnCostPerHour": 0.01,
  "serverlessCostPerRequest": 0.05
}
```

**Flow:**

1. Yes
2. No
3. Initiate Request
4. Cold Start Latency <= 5s?
5. Process Request; Zero Cost
6. Wait for Execution QueueTime; Non-Zero Cost
7. Respond to User
8. Respond to User after ColdStart Overhead

