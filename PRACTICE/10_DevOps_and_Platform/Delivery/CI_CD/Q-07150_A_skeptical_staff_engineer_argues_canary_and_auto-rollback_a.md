---
type: "practice"
difficulty: "Expert"
---

# A skeptical staff engineer argues canary and auto-rollback are over-engineering for your traffic and rolling deploys are enough; as the architect you disagree for tier-1 services. Steelman their case, then make the decision and the criteria that determine per-service which model applies.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Canary and Progressive Delivery](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Canary_and_Progressive_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When deciding between rolling deploys, canary deployments, and auto-rollback strategies in CI/CD pipelines for tier-1 services, it's crucial to consider factors such as risk tolerance, traffic distribution, service criticality, and rollback complexity. While rolling deploys are sufficient for low-risk services, more resilient strategies like canaries and auto-rollback are essential for critical services.

#### Triaging the Question

The skeptical staff engineer argues that simple rolling deploys are adequate for all but the most critical services due to their simplicity and speed of deployment. As an architect tasked with making the decision based on service tier, we need to evaluate risk, complexity, and operational reliability. Canaries and auto-rollback strategies provide a safety net in case a new version introduces issues.

#### Assumptions
Service Tiers: Tier-1 services are mission-critical, high-traffic services that cannot afford downtime or degraded service.
Risk Levels: Lower-tier services can tolerate higher risk due to less criticality and lower traffic.
Traffic Distribution: Canaries require a way to route small percentages of traffic without affecting the majority.

#### First Principles Solution
Step 1: Evaluate Risk Tolerance

For tier-1 services, even minor bugs can have significant consequences. Rolling deploys are fast but risky; a bug could impact thousands or millions of users immediately. Canary deployments and auto-rollback provide a gradual rollout mechanism that minimizes the risk.

Step 2: Implement Canaries for Critical Services

Canaries work by deploying new versions to a small subset of traffic (e.g., 1% of users) while closely monitoring their performance and user experience. If issues are detected, the rollback can be triggered immediately, reverting traffic back to the previous stable version.

Step 3: Consider Auto-Rollback for Enhanced Safety

Auto-rollback is an automated feature that reverses a deployment if certain metrics (e.g., error rate, latency) exceed predefined thresholds within a specified time window. This ensures that even undetected issues can be resolved quickly without manual intervention.

Decision Criteria:

Service Criticality:

Tier-1: Use canaries and auto-rollback.
Lower Tiers: Rolling deploys are sufficient if the service is less critical and has lower traffic.

Traffic Distribution:

Canaries require a way to route traffic. For high-traffic services, this routing must be seamless and low-impact on existing user experience.

Operational Impact:

Lower tiers may opt for rolling deploys due to the lower risk of impact.
Higher tiers need more resilient strategies like canaries to ensure service stability.

#### Closing Summary

In conclusion, while rolling deploys are a valid approach for less critical services, tier-1 services necessitate a higher level of resilience through canary deployments and auto-rollback capabilities. This ensures that even small issues do not cascade into full service disruptions.

#### Common Mistakes
and Follow-Ups:
Overlooking Low-Risk Services: Assuming all services should use the same deployment strategy.
Neglecting Monitoring: Failing to set up proper monitoring and alerting for canaries and auto-rollback.
Ignoring User Experience: Not accounting for the impact on user experience during deployments.
Trialing Different Scenarios:
Scenario 1: A new payment processing service in Tier-1. Implement a canary deployment with detailed monitoring and auto-rollback capabilities.
Scenario 2: A low-traffic analytics dashboard, where rolling deploys are sufficient due to lower risk and fewer users affected by potential issues.

**Comparison:**

| Service Tier | Deployment Strategy             |
|--------------|--------------------------------|
| Tier-1       | Canaries and Auto-Rollback      |
| Lower Tiers  | Rolling Deploys                 |

**Code:**

```yaml
# Example YAML for Canary Deployment
spec:
  service: payment-service
  canary:
    version: v2.1
    trafficSplit: 1%
  rollback:
    thresholdErrorRate: 5%
    thresholdTimeWindow: 30m
```

**Example input / output:**

```text
graph TD;
    A[Deploy v2.1] --> B[Monitor Metrics];
    B --> C{Metric Exceeds Threshold?};
    C -- Yes --> D[Rollback to v1];
    C -- No --> E[Continue Deployment]
```

**Data:**

```json
{
  "service": "payment-service",
  "canary": {
    "version": "v2.1",
    "trafficSplit": 1,
    "metrics": [
      { "name": "errorRate", "threshold": 5 },
      { "name": "latency", "threshold": 200 }
    ]
  },
  "rollback": {
    "triggeredByMetric": "errorRate",
    "thresholdErrorRate": 10,
    "timeWindow": 30
  }
}
```

**Flow:**

1. Yes
2. No
3. Deploy v2.1
4. Monitor Metrics
5. Metric Exceeds Threshold?
6. Rollback to v1
7. Continue Deployment

