---
type: "practice"
difficulty: "Deep"
---

# You want a canary rollout of a new model version behind the same /generate endpoint, routing 5% of traffic to it. How do you design the routing, response tagging, and metric split so you can compare quality and latency safely?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a canary rollout for a new model version behind the /generate endpoint, we need to ensure that only 5% of the traffic is routed to the new version while the remaining 95% continues to use the existing version. This process must be seamless and allow us to collect metrics on both versions so we can compare their performance safely.

#### Triage

The core of this question revolves around:

Routing Traffic: Designing a mechanism that routes exactly 5% of traffic to the new model.
Response Tagging: Ensuring responses from different versions are tagged appropriately for easy differentiation in logging and metrics.
Metric Split: Implementing a system to collect and compare metrics from both versions.

#### Assumptions
We have access to load balancers or network-level tools that can route traffic based on specific criteria.
The API gateway or server-side logic is capable of distinguishing between different requests and responses.
Metrics are available at the request and response level.

#### Solution
Step 1: Traffic Splitting

To achieve a 5% canary rollout, we need to implement a mechanism that splits traffic such that only 5% gets routed to the new model. This can be done using load balancers or custom server-side logic.

Load Balancer Approach:

If you are using a cloud-based load balancer like AWS ALB, Google LB, or Azure Front Door, these services often provide built-in mechanisms for traffic splitting via rules and policies.

#### Example
In AWS ALB, create a rule that sends 5% of the requests to a target group associated with the new model version.

Server-Side Logic Approach:

Implement a custom solution where each incoming request has an identifier or header that indicates its origin. Use this information in your API gateway or server-side logic to decide whether to route the traffic to the old or new model.

#### Example
Add a canary query parameter or a header like X-Canary-Version. If present and set to new, send the request to the new version.
Step 2: Response Tagging

To ensure clear differentiation of responses, we need to tag each response with metadata indicating which model was used. This can be done through HTTP headers or custom JSON tags in the response body.

Response Header Approach:

Add a header like X-Model-Version that contains the version information for the model used.
PYTHON
if request.headers.get('X-Canary-Version') == 'new':
    new_model_response = call_new_model()
    return {'result': new_model_response, 'version': 'new'}
else:
    old_model_response = call_old_model()
    return {'result': old_model_response, 'version': 'old'}

Response Body Approach:

Modify the JSON response to include a version field.
JSON
{
  "result": "response from new model",
  "version": "new"
}
Step 3: Metric Collection and Comparison

To compare the quality and latency of both models, we need a robust metric collection system.

Metrics Collection:

Use a metrics service like Prometheus or Grafana to collect and aggregate performance metrics. Ensure that each version's metrics are tagged with its corresponding version.
PYTHON
from prometheus_client import start_http_server, Counter

new_model_counter = Counter('model_new_response_time', 'Response time for new model')
old_model_counter = Counter('model_old_response_time', 'Response time for old model')

if request.headers.get('X-Canary-Version') == 'new':
    response_time = call_new_model()
    new_model_counter.inc(response_time)
else:
    response_time = call_old_model()
    old_model_counter.inc(response_time)

Metrics Aggregation:

Use a data aggregation tool like Grafana or CloudWatch to visualize the metrics and perform comparative analysis.
GRAFANA
# Query for new model response time
SELECT mean("model_new_response_time") FROM "model_new_response_time" WHERE $time_range GROUP BY time(1m)

# Query for old model response time
SELECT mean("model_old_response_time") FROM "model_old_response_time" WHERE $time_range GROUP BY time(1m)

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to properly tag responses, leading to confusion in metrics collection.
Not having a clear rollback plan if the new model performs poorly.
Overlooking edge cases where traffic might not be correctly split.

#### Follow-Ups

Discussing how to automate the canary rollout process using CI/CD pipelines.
Exploring advanced techniques like gradual rollouts or A/B testing in production environments.

**Comparison:**

| Time (UTC) | Model New Response Time (ms) | Model Old Response Time (ms) |
|------------|-----------------------------|----------------------------|
| 2022-01-01 |                          200 |                           300 |
| 2022-01-02 |                          250 |                           280 |

**Code:**

```python
from prometheus_client import start_http_server, Counter

new_model_counter = Counter('model_new_response_time', 'Response time for new model')
old_model_counter = Counter('model_old_response_time', 'Response time for old model')

def handle_request(request):
    if request.headers.get('X-Canary-Version') == 'new':
        response_time = call_new_model()
        new_model_counter.inc(response_time)
        return {'result': "response from new model", 'version': 'new'}
    else:
        response_time = call_old_model()
        old_model_counter.inc(response_time)
        return {'result': "response from old model", 'version': 'old'}

def call_new_model():
    # Logic to call the new model
    pass

def call_old_model():
    # Logic to call the old model
    pass
```

**Example input / output:**

```text
Input:
GET /generate?X-Canary-Version=new

Output (with metrics):
{
  "result": "response from new model",
  "version": "new"
}

Metrics Collection:
- `model_new_response_time`: Mean response time for the new model
- `model_old_response_time`: Mean response time for the old model
```

**Data:**

```json
{
  "model_new_response_time": [
    [1640995200, 200],
    [1640998800, 250]
  ],
  "model_old_response_time": [
    [1640995200, 300],
    [1640998800, 280]
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is Canary Version?
5. Call New Model
6. Record Metrics
7. Return Response
8. Call Old Model
9. Record Metrics
10. Return Response
11. Canary Rollout Complete
12. Canary Rollout Complete

