---
type: "practice"
difficulty: "Deep"
---

# How would you deploy a new LLM router (model-selection logic) so that a routing bug can only send a small fraction of traffic to the wrong/expensive model before you catch it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Canary and Progressive Delivery](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Canary_and_Progressive_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a deployment bug in an LLM router sends only a small fraction of traffic to the wrong or expensive models, we need to implement a controlled rollout strategy with canary deployments and A/B testing. This approach allows us to carefully monitor the behavior of the new model before fully committing to it.

Firstly, let's clarify what the question is asking:

Deployment Strategy: We need to design a deployment process that minimizes the risk of a routing bug causing significant performance or financial issues.
Monitoring and Testing: We must have robust monitoring and testing mechanisms in place to detect any anomalies early.
Rollout Phases: The solution should include multiple phases, such as canary release and gradual increase.
Step-by-Step Solution

Define the Deployment Environment:

Use a staging environment that mirrors production but is isolated to prevent accidental service disruptions.
Ensure the LLM router is containerized using Docker or similar tools for consistent behavior across environments.

Implement Canary Releases:

Start by deploying a small subset of traffic (e.g., 1%) to the new model in the staging environment.
Monitor metrics such as response time, error rates, and throughput to ensure everything is functioning correctly.

A/B Testing:

Gradually increase the traffic percentage to the new model while keeping the old one active.
Use A/B testing tools or custom scripts to split traffic percentages dynamically based on predefined criteria (e.g., user ID, timestamp).

Real-time Monitoring and Alerts:

Set up comprehensive logging and monitoring for both models.
Implement alerting mechanisms that trigger notifications if any key metrics cross threshold values.

Rollback Plan:

Ensure a reliable rollback strategy is in place to switch back to the previous model quickly if issues are detected.
Automate this process using CI/CD pipelines with conditional branches for rapid deployment and rollbacks.

User Experience Considerations:

Prioritize user experience by ensuring that any degradation in performance or service availability is minimal during the transition.
Use load testing tools to simulate high traffic scenarios before going live.

#### Closing Summary

By employing a canary release strategy with A/B testing, we can systematically introduce new models into production environments while minimizing the risk of catastrophic failures. This approach ensures that any issues are confined to a small fraction of users and can be addressed quickly without disrupting the entire user base.

#### Common Mistakes
and Follow-Ups
Not Enough Testing: Failing to perform thorough testing in staging can lead to undetected bugs.
Ignoring Metrics: Relying on subjective feedback rather than objective metrics can result in delayed detection of issues.
Over-Rollout Too Quickly: Releasing too much traffic too quickly increases the risk of user dissatisfaction and financial loss.

