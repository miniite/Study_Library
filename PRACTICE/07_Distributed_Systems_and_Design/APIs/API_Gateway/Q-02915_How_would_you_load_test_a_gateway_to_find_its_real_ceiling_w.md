---
type: "practice"
difficulty: "Expert"
---

# How would you load test a gateway to find its real ceiling; what metrics tell you it is the gateway (not upstreams) that saturates, and how do you find the limiting resource?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Scaling for Traffic Spikes](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/Gateway_Scaling_for_Traffic_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively load test an API Gateway and determine if it's reaching its true ceiling, we need to focus on several key metrics and perform a series of steps from first principles. The goal is to identify whether the gateway itself becomes the bottleneck or if the upstream services are causing issues. This involves setting up a controlled environment for testing, analyzing performance metrics, and understanding resource limitations.

The core idea here is to isolate and test the API Gateway under varying loads to see where it starts to show signs of degradation or saturation. By carefully monitoring specific metrics such as response time, error rates, and concurrency limits, we can pinpoint whether the gateway itself is the limiting factor. We'll use these insights to optimize our gateway's architecture and configuration.

**Comparison:**

```
Metric	Description
Response Time (Avg)	Average time taken to process a request.
Response Time (P50, P90, Max)	Percentile response times indicating performance under load.
Error Rate (Gateway vs Upstream)	Percentage of requests that failed at the gateway level versus upstream services.
Concurrency Limit	Maximum number of concurrent connections the gateway can handle.
```

**Code:**

```python
# Example code snippet for setting up a load test with k6 (a popular performance testing tool)
import k6
from k6 import http, sleep

options = {
    "title": "API Gateway Load Test",
    "vus": 100,
    "duration": "3m"
}

def setup():
    pass

def teardown():
    pass

def test():
    res = http.get("http://api-gateway-url", headers={"Authorization": "Bearer your-token"})
    if res.status != 200:
        k6.error(f"Request failed with status {res.status}")

k6.export Scenarios(test)
```

**Example input / output:**

```text
- Input:
  - k6 test configuration with 100 virtual users and a 3-minute duration.
  - API Gateway URL: http://api-gateway-url

- Output:
  - Response time metrics (e.g., avg, p50, p90, max).
  - Error rates for the gateway and upstream services.
  - Concurrency limits observed during testing.
```

**Data:**

```json
{
  "testConfig": {
    "title": "API Gateway Load Test",
    "vus": 100,
    "duration": "3m"
  },
  "headers": [
    {"name": "Authorization", "value": "Bearer your-token"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Setup Load Test
4. Is Gateway Saturated?
5. Identify Gateway Bottleneck
6. Check Upstream Services
7. Optimize GatewayConfiguration
8. Debug and Optimize UpstreamServices
9. Proceed with OptimizationPlan
10. Proceed with OptimizationPlan

