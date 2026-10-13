---
type: "practice"
difficulty: "Deep"
---

# How would you decide when a shared internal ML service (a feature store, an embedding service, a central inference gateway) has become a dangerous single point of failure for the whole company, and re-architect it without a big-bang?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether a shared internal ML service has become a dangerous single point of failure (SPOF), we need to consider multiple factors including the criticality of the service, its impact on business operations, and the potential consequences of a failure. In this scenario, as an Architect/Principal ML Engineer with deep understanding, our goal is to re-architect the system incrementally rather than through a big-bang approach. Here’s how we would systematically tackle this problem:

Assess Criticality and Impact: First, determine which services rely on this shared service and assess their importance. For instance, if the feature store or embedding service supports critical real-time decision-making systems like fraud detection or recommendation engines, a failure could lead to significant financial loss, reputational damage, or regulatory issues.

Identify Risk Factors: Next, identify potential risk factors such as dependency concentration, lack of redundancy, and operational fragility. For example, if the service relies on a single database with no backup, or if it lacks proper failover mechanisms, these are clear indicators of high risk.

Evaluate Service Dependencies: Map out all dependencies and understand how services interact. If multiple critical systems depend heavily on this single service, it becomes more likely to be an SPOF.

Historical Failure Analysis: Look into the history of failures or near-misses related to this service. Analyze root causes, downtime duration, and impact on downstream services. This can provide valuable insights into potential vulnerabilities.

Performance Metrics and Monitoring: Assess current monitoring and alerting mechanisms in place. If there are no robust metrics or alerts for critical performance indicators like latency, availability, and throughput, the system is more prone to unnoticed failures.

Risk Assessment Framework: Use a risk assessment framework to quantify risks. For example, consider factors such as probability of failure, impact on business operations, time-to-recovery, and cost of downtime.

Incremental Re-architecture Plan: Develop an incremental re-architecture plan that minimizes disruption while improving resilience. This might involve:

Introducing redundant instances or clusters.
Implementing load balancing and failover mechanisms.
Setting up proper caching strategies to reduce dependency on single points of failure.
Implementing a service mesh for better observability and resiliency.

Pilot and Iterative Deployment: Begin with a small-scale pilot in non-critical systems before rolling out changes company-wide. This allows you to test new configurations, identify issues early, and gather feedback.

Continuous Monitoring and Feedback Loops: Establish continuous monitoring and feedback loops to ensure that the re-architected system performs as expected. Regularly review performance metrics and make adjustments as needed based on real-world usage patterns.

Communication and Training: Communicate changes clearly to stakeholders and provide necessary training for operations teams to handle new architectures effectively.

By following these steps, we can proactively address the risk of a shared internal ML service becoming an SPOF, ensuring the reliability and resilience of critical systems.

**Comparison:**

```
Dependency	Criticality	Impact
Feature Store	High	Severe
Recommendation Engine	Medium	Moderate
Risk Factors	Description
Single Point of Failure	Database not backed up, no failover mechanisms.
Lack of Monitoring	No alerting or performance metrics in place.
```

**Code:**

```python
# Example of monitoring code snippet using Prometheus
from prometheus_client import start_http_server, Counter

# Initialize counters for request latency and failures
latency_counter = Counter('service_latency', 'Latency distribution in the service')
failure_counter = Counter('service_failures', 'Number of service failures')

def monitor_service():
    # Simulate monitoring logic
    start_http_server(8000)
    while True:
        try:
            latency = get_request_latency()
            latency_counter.observe(latency)
            if is_request_successful():
                print("Request successful")
            else:
                failure_counter.inc()
        except Exception as e:
            print(f"Error: {e}")

def get_request_latency():
    # Simulate request latency
    import random
    return random.uniform(0.1, 1)

def is_request_successful():
    # Simulate request success condition
    import random
    return random.choice([True, False])
```

**Example input / output:**

```text
Input: Service Dependency List
Output: Risk Factors Identified
[{
  "service": "Feature Store",
  "criticality": "High",
  "impact": "Severe"
},
{
  "service": "Recommendation Engine",
  "criticality": "Medium",
  "impact": "Moderate"
}]

Input: Historical Failure Analysis
Output:
- Failure on March 5th, downtime: 2 hours, impact: severe
- Near-miss on April 10th, root cause: database failure

Input: Performance Metrics and Monitoring
Output:
- Latency counter: 0.3s, 0.4s, 0.6s, ...
- Failure counter: 5 failures in the last week
```

**Data:**

```json
{
  "dependencies": [
    {
      "service": "Feature Store",
      "criticality": "High",
      "impact": "Severe"
    },
    {
      "service": "Recommendation Engine",
      "criticality": "Medium",
      "impact": "Moderate"
    }
  ],
  "risk_factors": [
    {
      "factor": "Single Point of Failure",
      "description": "Database not backed up, no failover mechanisms."
    },
    {
      "factor": "Lack of Monitoring",
      "description": "No alerting or performance metrics in place."
    }
  ]
}
```

**Flow:**

1. Rendering diagram…

