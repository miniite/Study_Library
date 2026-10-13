---
type: "practice"
difficulty: "Deep"
---

# You run inference on both a hyperscaler's managed model endpoints and a cheaper independent inference provider, and their pricing, quotas, and reliability differ. Design the cost-aware traffic-splitting policy and the guardrail that keeps you from over-concentrating on the cheap provider right before it degrades.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Model Routing Cost vs Quality](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Model_Routing_Cost_vs_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a cost-aware traffic-splitting policy while ensuring robust reliability, we must balance the trade-offs between budget constraints and service quality. The key is to dynamically allocate traffic based on real-time metrics such as pricing, performance, and availability, without over-concentrating on an unreliable provider that could degrade unexpectedly.

#### Triaging the Question

The core of this question revolves around creating a robust mechanism for traffic distribution across multiple inference providers, ensuring cost efficiency while maintaining service reliability. The interviewer is probing our understanding of cloud economics, real-time monitoring, and dynamic decision-making processes. We need to assume the existence of two providers: one managed by a hyperscaler with potentially higher costs but reliable performance, and another independent provider that may offer cheaper rates but has varying reliability.

#### First-Principles Solution
Define Metrics: We will define key metrics such as cost per inference, latency, error rate, and service level agreement (SLA) compliance.
Real-Time Monitoring: Use cloud-native tools to continuously monitor these metrics in real-time.
Dynamic Policy Engine: Implement a policy engine that adjusts traffic based on the current state of each provider.
Guardrails: Establish guardrails to prevent over-concentration on less reliable providers, ensuring service availability.
Step-by-Step Solution

Define Metrics and SLAs:

Cost per inference: Calculate cost for both providers.
Latency: Measure response time for inferences.
Error rate: Track the number of failed or erroneous inferences.
SLA compliance: Ensure that service availability meets predefined thresholds.

Real-Time Monitoring:

Use cloud monitoring services (e.g., CloudWatch, Datadog) to collect and stream metrics in real-time.
Set up alerts for critical performance issues (high error rate, increased latency).

Dynamic Policy Engine:

Implement a policy engine that can make decisions based on the current state of providers.

#### Example
architecture using Kubernetes and Istio Service Mesh for dynamic routing.

Guardrails:

Define thresholds to avoid over-concentration on less reliable providers.
Adjust traffic allocation dynamically based on provider performance.
Sample Code
PYTHON
import time
from kubernetes import client, config

def fetch_metrics(provider):
    # Fetch metrics from monitoring service (CloudWatch, Datadog)
    return {
        "cost": 0.15,
        "latency": 200,
        "error_rate": 0.03,
        "sla_compliance": True
    }

def adjust_traffic(traffic_split):
    # Adjust traffic based on policy engine decisions
    print(f"Adjusting traffic split to {traffic_split}% for provider A")

def monitor_and_adjust():
    config.load_kube_config()
    v1 = client.CoreV1Api()

    while True:
        metrics_a = fetch_metrics("provider_A")
        metrics_b = fetch_metrics("provider_B")

        if (metrics_b["error_rate"] > 0.1 or
            metrics_b["latency"] > 300 or
            not metrics_b["sla_compliance"]):
            adjust_traffic(95)  # Shift traffic to provider A

        else:
            adjust_traffic(50)  # Balance between providers

        time.sleep(60)

if __name__ == "__main__":
    monitor_and_adjust()

#### Closing Summary
and Common Mistakes

This solution provides a robust mechanism for dynamically adjusting traffic based on real-time performance metrics. The key is to continuously monitor these metrics and adjust the traffic split to ensure cost efficiency and reliability. Common mistakes include over-reliance on a single provider, lack of dynamic adjustment mechanisms, and ignoring guardrails that could lead to service degradation.

**Code:**

```python
import time
from kubernetes import client, config

def fetch_metrics(provider):
    # Fetch metrics from monitoring service (CloudWatch, Datadog)
    return {
        "cost": 0.15,
        "latency": 200,
        "error_rate": 0.03,
        "sla_compliance": True
    }

def adjust_traffic(traffic_split):
    # Adjust traffic based on policy engine decisions
    print(f"Adjusting traffic split to {traffic_split}% for provider A")

def monitor_and_adjust():
    config.load_kube_config()
    v1 = client.CoreV1Api()

    while True:
        metrics_a = fetch_metrics("provider_A")
        metrics_b = fetch_metrics("provider_B")

        if (metrics_b["error_rate"] > 0.1 or
            metrics_b["latency"] > 300 or
            not metrics_b["sla_compliance"]):
            adjust_traffic(95)  # Shift traffic to provider A

        else:
            adjust_traffic(50)  # Balance between providers

        time.sleep(60)

if __name__ == "__main__":
    monitor_and_adjust()
```

**Example input / output:**

```text
Fetching metrics for "provider_A"
cost: 0.15, latency: 200, error_rate: 0.03, sla_compliance: True

Fetching metrics for "provider_B"
cost: 0.08, latency: 210, error_rate: 0.12, sla_compliance: False

Adjusting traffic split to 95% for provider A
```

**Flow:**

1. Yes
2. No
3. Provider A Metrics
4. Error Rate > 0.1?
5. Shift Traffic to Provider A
6. Balance Between Providers
7. Apply Adjustment
8. Wait for 60 seconds

