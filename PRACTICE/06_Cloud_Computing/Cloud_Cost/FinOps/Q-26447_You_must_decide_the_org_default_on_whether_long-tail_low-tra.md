---
type: "practice"
difficulty: "Deep"
---

# You must decide the org default on whether long-tail, low-traffic GenAI models get scale-to-zero serving (idle savings, cold-start latency of loading large weights) versus a warm floor. How do you set the policy per model class and meter the cost-versus-latency trade?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Model Routing Cost vs Quality](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Model_Routing_Cost_vs_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In deciding whether long-tail, low-traffic GenAI models should operate with scale-to-zero serving or a warm floor, we must balance idle savings against cold-start latency and cost-versus-latency trade-offs. This involves setting policies based on model class characteristics and metering these costs to ensure efficient resource utilization.

#### Triage

The question is essentially about determining the optimal operational strategy for GenAI models with low traffic—specifically whether they should be configured for scale-to-zero (to save idle resources) or a warm floor (to maintain a minimal level of readiness). The core issue here is balancing cost efficiency and user experience. We need to consider the model's use case, resource requirements, and the impact on latency.

#### First-Principles Solution

Define Model Classes:

Identify different classes based on traffic volume and criticality.

Evaluate Cost vs. Latency Trade-offs:

For scale-to-zero models, the goal is to minimize idle costs by scaling down or completely stopping service when not in use.
For warm floor models, there's a trade-off between maintaining readiness (reducing cold-start latency) and keeping resources active even during periods of low traffic.

Metering and Monitoring:

Implement mechanisms to monitor usage patterns and dynamically adjust the strategy based on real-time data.
Use cost metrics as primary indicators for policy enforcement, but balance with user experience requirements.

Policies by Model Class:

High-Criticality Models: These might be better suited for a warm floor due to their critical nature or frequent use spikes (e.g., emergency response models).
Low-Criticality Models: Scale-to-zero could provide significant savings, but may introduce cold-start latency that can be mitigated with caching and pre-warming strategies.

#### Implementation
Strategy:

Leverage cloud-native tools like AWS Lambda, Google Cloud Functions, or Azure Functions for scale-to-zero.
Use auto-scaling groups in Kubernetes or managed services to maintain a warm floor state without manual intervention.

#### Closing Summary

By setting policies based on model class characteristics and metering the cost-versus-latency trade-offs, we can optimize resource usage while ensuring that critical applications are not affected. This approach involves defining clear criteria for each model class, implementing automated scaling strategies, and continuously monitoring performance metrics to ensure efficient operation.

#### Common Mistakes
Overgeneralization: Assuming a single policy fits all models without considering individual use cases.
Ignoring User Experience: Prioritizing cost savings over user experience, leading to unacceptable cold-start latencies for users.
Inadequate Monitoring: Failing to implement robust monitoring and feedback mechanisms that can adapt policies in real-time.

**Comparison:**

```
Model Class	Traffic Thresholds	Latency Thresholds
High-Criticality	> 100 requests/min	< 1 sec
Low-Traffic	<= 5 requests/min	>= 2 sec
```

**Code:**

```python
# Example policy implementation using AWS Lambda
def lambda_handler(event, context):
    # Check if the request is for a low-traffic model
    if event['model_class'] == 'low_traffic':
        # Scale to zero during idle periods
        return {
            "statusCode": 200,
            "body": json.dumps({"status": "scaled_to_zero"})
        }
    else:
        # Maintain warm floor state for high-criticality models
        return {
            "statusCode": 200,
            "body": json.dumps({"status": "warm_floored"})
        }
```

**Example input / output:**

```

Request: { "model_class": "high_criticality", "timestamp": "2023-10-01T12:00:00Z" }

Response: { "status": "warm_floored" }

CODE
Copy

```plaintext
Request:
{
  "model_class": "low_traffic",
  "timestamp": "2023-10-01T18:00:00Z"
}

Response:
{
  "status": "scaled_to_zero"
}
```

**Data:**

```json
{
  "model_classes": [
    {"name": "high_criticality", "thresholds": {"traffic": 100, "latency": 1}},
    {"name": "low_traffic", "thresholds": {"traffic": 5, "latency": 2}}
  ]
}
```

**Flow:**

1. Scale-to-Zero Policies
2. Warm Floor Policies
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Model Class
10. Is Model Critical?
11. Warm Floor Policy
12. Scale-to-Zero Policy
13. Monitor Traffic
14. Traffic Spike?
15. Pre-warm Model
16. Keep Warm
17. Monitor Idle Time
18. Idle Period Exceeded?
19. Suspend Service
20. Resume Service

