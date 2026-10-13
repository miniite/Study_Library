---
type: "practice"
difficulty: "Expert"
---

# Your inference endpoint runs on GPU pods with continuous batching. A naive readiness probe reports healthy while the KV-cache is saturated and queue depth explodes. How do you design health and load-shedding signals that reflect true serving capacity?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to design a comprehensive health check and load-shedding mechanism that accurately reflects the serving capacity of our inference endpoint. The key idea is to create dual signals: one reflecting the overall system health (health signal) and another indicating the current load and potential risk of service degradation (load shedding signal). This solution involves monitoring key metrics, setting thresholds, and dynamically adjusting the system based on these metrics.

#### Triage

The core question revolves around designing robust health checks that go beyond simple liveness/readiness probes. Specifically, we need to ensure that our endpoint is not only responsive but also capable of handling requests efficiently without exhausting resources like GPU memory or queueing up excessive requests.

#### Assumptions
and Key Metrics
We have a continuous batching setup where incoming requests are batched before being sent to the inference engine.
The system uses GPU pods for running inference tasks.
KV-cache is used to store some precomputed results for faster response times.
Queue depth indicates how many requests are pending processing.

#### First-Principles Solution
Health Signal
Overall System Health Check: This checks if the endpoint is responsive and able to process requests without immediate errors or failures.

#### Example
A simple HTTP GET request to a /health endpoint that returns a 200 OK status code when the system is healthy.
Load Shedding Signals
GPU Utilization Monitoring: Monitor GPU usage to ensure it does not exceed a safe threshold, which could indicate resource exhaustion.
Queue Depth Thresholds: Set thresholds for queue depth to trigger alerts or take action if the queue becomes too deep.
KV-Cache Saturation Check: Monitor the cache hit rate and eviction rate to detect when the cache is being overwhelmed.

#### Implementation
Steps

Health Signal

Create a /health endpoint that returns a 200 OK status code if no critical errors are detected.
PYTHON
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health', methods=['GET'])
def health_check():
    return jsonify({"status": "ok"}), 200

Load Shedding Signals

Use Prometheus or similar monitoring tools to collect metrics on GPU utilization and queue depth.
Define thresholds for these metrics:
PLAINTEXT
GPU_UTILIZATION_THRESHOLD = 85%
QUEUE_DEPTH_THRESHOLD = 100 requests
KV_CACHE_HIT_RATE_THRESHOLD = 70%
Implement alerts and actions based on these thresholds.

Alerting and Auto-scaling

Set up alert rules in Prometheus to trigger when any of the above metrics exceed their thresholds.
Use Kubernetes HorizontalPodAutoscaler (HPA) or custom logic to scale GPU pods up or down based on these alerts.
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: inference-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: inference-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Pods
    pods:
      metricName: cpu_utilization
      targetAverageUtilization: 80

Dynamic Load Management

Introduce a circuit breaker pattern to temporarily reject new requests when the system is under heavy load.
PYTHON
from resiliency import CircuitBreaker

circuit_breaker = CircuitBreaker(fail_rate_threshold=50)  # Adjust based on tests

@app.route('/predict', methods=['POST'])
def predict():
    if not circuit_breaker.is_open:
        return process_request()
    else:
        return jsonify({"error": "Service Temporarily Unavailable"}), 503

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, designing health and load-shedding signals requires a multi-faceted approach that includes monitoring key metrics, setting appropriate thresholds, and implementing responsive actions. The common mistakes include relying solely on simple liveness probes, ignoring resource constraints like GPU utilization, and not having robust alerting and auto-scaling mechanisms in place.

#### Follow-up
questions might explore:

How do you fine-tune the load shedding signals to minimize false positives/negatives?
What are some best practices for integrating these health checks with existing monitoring tools like Prometheus or Grafana?
How would you handle cases where scaling is not sufficient, and additional optimization is needed?

**Comparison:**

```
Metric	Threshold	Action
GPU Utilization	>85%	Scale up pods
Queue Depth	>100 requests	Implement circuit breaker
KV Cache Hit Rate	<70%	Alert and optimize cache
```

**Code:**

```

from flask import Flask, jsonify

app = Flask(name)

@app.route('/health', methods=['GET']) def health_check(): return jsonify({"status": "ok"}), 200

CODE
Copy

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: inference-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: inference-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Pods
    pods:
      metricName: cpu_utilization
      targetAverageUtilization: 80
```

**Example input / output:**

```text
Request to /health:
Input: GET http://localhost:5000/health
Output: {"status": "ok"} with status code 200

Request when queue depth exceeds threshold:
Input: POST http://localhost:5000/predict with request payload
Output: {"error": "Service Temporarily Unavailable"} with status code 503
```

**Data:**

```json
{
  "metrics": {
    "gpu_utilization": 83,
    "queue_depth": 95,
    "kv_cache_hit_rate": 72
  },
  "thresholds": {
    "gpu_utilization_threshold": 85,
    "queue_depth_threshold": 100,
    "kv_cache_hit_rate_threshold": 70
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Inference Request
8. Is GPU Utilization <=Threshold?
9. Process Request
10. Is Queue Depth <= Threshold?
11. Is KV Cache Hit Rate >=Threshold?
12. Implement Circuit Breaker
13. Alert Admin

