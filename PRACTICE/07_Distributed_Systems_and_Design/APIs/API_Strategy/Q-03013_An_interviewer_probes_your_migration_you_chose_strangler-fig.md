---
type: "practice"
difficulty: "Expert"
---

# An interviewer probes your migration: "you chose strangler-fig to replace the legacy API — what specifically triggers strangling each endpoint, and what's your rollback if the new one is wrong under production load you couldn't simulate?"

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Lifecycle Metrics and Retirement](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Lifecycle_Metrics_and_Retirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a legacy API using the strangler-fig pattern, the primary goal is to incrementally replace the old system while minimizing disruption and risk. This approach involves wrapping the legacy system with new functionality in smaller increments, allowing for controlled changes that can be managed and rolled back if necessary.

The specific triggers for strangling each endpoint involve carefully planned integration points where new features are introduced gradually. Typically, this is done through feature toggles or circuit breakers to isolate risky parts of the API until they are thoroughly tested in a live environment. My rollback strategy focuses on ensuring that any problematic changes can be rapidly reversed without causing broader issues.

Here's how I would approach both:

Triggers for Strangling Each Endpoint
Identify Critical Endpoints: Start by identifying critical endpoints or functionalities within the legacy system that are frequently used and business-critical.
Implement Feature Toggles: Introduce feature toggles at the API level to control which parts of the new implementation should be active in production. This allows for gradual rollout and easy rollback if needed.
Granular Rollout Strategy: Gradually turn on these features based on observed usage patterns or specific business requirements, ensuring that only necessary parts are replaced first.
Rollback Plan
Isolate Problem Areas: Use circuit breakers to quickly isolate problematic areas of the new implementation. This prevents cascading failures and allows for targeted troubleshooting.
Automated Rollback Mechanisms: Implement automated mechanisms to revert changes in case of production issues, such as using feature toggles to disable the new code path and re-enable the legacy API.
Monitoring and Alerts: Set up comprehensive monitoring and alerting to detect performance degradation or other anomalies early on.

#### Example
Workflow
Initial Setup: Wrap the legacy API with a new layer that can handle both old and new requests simultaneously.
Feature Toggle Activation: Enable feature toggles for specific endpoints, e.g., /v1/new-endpoint.
Observation Period: Monitor performance metrics such as response time, error rates, and usage patterns to ensure stability.
Rollback if Necessary: If issues arise, disable the new code path using the feature toggle and revert to the legacy API.

#### Closing Summary

The strangler-fig pattern is effective for migrating complex legacy systems by breaking them down into manageable pieces. Careful planning around triggers and robust rollback strategies are crucial to ensure a smooth transition and minimize risk. Common mistakes include not adequately testing new features in a production-like environment, neglecting comprehensive monitoring, and rushing the deployment process.

**Comparison:**

| Endpoint Path   | Legacy API Logic | New API Logic |
|-----------------|------------------|--------------|
| /v1/getUser     | Basic retrieval  | Enhanced data fetch |
| /v2/createOrder | Simple creation  | Advanced validation and logging |

**Code:**

```python
# Example of using feature toggles with a simple circuit breaker implementation
class FeatureToggle:
    def __init__(self, enabled):
        self.enabled = enabled

def handle_request(path, feature_toggle):
    if path == "/v1/new-endpoint" and not feature_toggle.enabled():
        # Fallback to legacy API logic
        return legacy_api_logic()
    else:
        # New API logic
        return new_api_logic()

# Example usage
feature_toggle = FeatureToggle(enabled=True)  # Set based on monitoring data
response = handle_request("/v1/new-endpoint", feature_toggle)
```

**Example input / output:**

| Input Path | Legacy API Response | New API Response |
|------------|-------------------|-----------------|
| `/v1/getUser` | User details (unoptimized) | Optimized user details |
| `/v2/createOrder` | Order created, no validation | Order created with security checks |

**Data:**

```json
{
  "legacy_api": {
    "url": "https://oldapi.example.com/v1",
    "endpoints": [
      { "path": "/v1/getUser", "description": "Get user details" },
      { "path": "/v1/createOrder", "description": "Create a new order" }
    ]
  },
  "new_api": {
    "url": "https://api.example.com/v2",
    "endpoints": [
      { "path": "/v2/getUser", "description": "Get updated user details" },
      { "path": "/v2/createOrder", "description": "Create a new order with additional validation" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Legacy API Call
4. Feature Toggle Enabled?
5. Execute New API Logic
6. Execute Legacy API Logic
7. Monitor Performance Metrics
8. Rollback if Issues Detected
9. Roll Back to Legacy API

