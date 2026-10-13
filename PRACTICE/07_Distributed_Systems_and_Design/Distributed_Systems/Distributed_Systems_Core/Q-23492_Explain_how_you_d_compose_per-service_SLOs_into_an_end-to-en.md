---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd compose per-service SLOs into an end-to-end user-journey SLO across a 12-hop path. Why is naive multiplication wrong and how does the error budget policy change deploy behavior?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SLO Burn Rate Alerting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/SLO_Burn_Rate_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how to compose per-service SLOs into an end-to-end user-journey SLO across a 12-hop path, we need to understand that each service in this journey is critical for delivering a seamless user experience. The challenge lies not just in multiplying the individual Service Level Objectives (SLOs), but in managing the error budget and understanding how deployment strategies impact reliability.

Firstly, let's define what we mean by SLOs: they are measurable targets that indicate the quality of service. For instance, a common SLO might be ensuring 99% of requests are processed within 100ms. However, when dealing with multiple services in a path, these individual SLOs must be aggregated into an overall user-journey SLO.

#### Triaging the Question

The question is asking how to effectively manage and combine SLOs across multiple services to ensure the entire user journey meets the desired reliability and performance metrics. The key aspects are:

Multiplication vs Aggregation: Why simple multiplication of per-service SLOs doesn't work.
Error Budget Policy: How it influences deployment behavior and how SLOs can be adjusted during deployments.

#### First-Principles Solution

To address the issue, we must recognize that naive multiplication is incorrect because:

Latent Errors: Not all errors are independent; some issues can propagate through the system.
System Resilience: A single service might have a high SLO under normal conditions but could fail during edge cases or unexpected scenarios.
Aggregating SLOs with Error Budget

The correct approach is to use an error budget policy, which allows for periods of higher failure rates while maintaining overall service quality. This policy involves:

Defining the Journey SLO: Establish a composite SLO that represents the desired reliability and performance of the entire user journey.
Error Budget Allocation: Allocate an error budget across services based on their relative importance to the end-user experience.
Monitoring and Feedback: Continuously monitor service health and adjust the error budgets as needed.

#### Example
Error Budget Policy

Assume we have a 100,000 requests per day for our user journey, and we want an overall SLO of at least 99.5% with no more than 24 hours of downtime per month (i.e., an error budget of 3 days worth of errors).

If Service A has a daily failure rate of 1%, it can tolerate up to 6 failures in the allowed period.
Service B might have a lower SLO, say 95% with 24 hours of downtime, giving it more leniency.

This allows for some services to experience higher failure rates while ensuring the overall user journey meets the desired SLA.

Impact on Deployment

The error budget policy significantly influences deployment strategies:

Staging and Canary Releases: Deploy changes gradually using canary releases to monitor impact on SLOs.
Rollback Mechanisms: Implement rollback plans to revert deployments if they exceed the error budget.
Automated Rollbacks: Automate the process of rolling back problematic deployments to maintain service levels.

#### Example
Deployment Scenario

#### Cons
ider a 12-hop path with an overall user-journey SLO of 99.5%. If one hop experiences issues, it can consume part of its allocated error budget but should not exceed the total allowed downtime. For instance:

Hop 3 (Service C) has a daily failure rate of 0.5%.
A deployment introduces an issue causing a failure rate to spike to 2%.

If this exceeds its allocated budget of 12 failures in a month, it triggers a rollback mechanism to restore the previous stable version.

#### Closing Summary

By using error budgets and monitoring mechanisms, we can effectively manage SLOs across multiple services. The naive multiplication approach is insufficient because it doesn't account for system resilience and dependency on other services. Error budget policies provide a flexible framework that allows for controlled risk during deployments while ensuring overall service quality.

#### Common Mistakes
and Follow-Ups
Ignoring Dependencies: Failing to consider how issues in one hop can impact others.
Static Budgets: Not allowing flexibility based on real-time monitoring data.
Manual Rollbacks: Relying solely on manual intervention for rollbacks, which can introduce delays.

#### Follow-up
questions could include:

How would you handle cascading failures between services?
What metrics would you use to continuously monitor service health and adjust error budgets?

**Code:**

```
MERMAID
Copy
graph TD;
    A[Service 1] --> B(95% SLO)
    B --> C[Service 3]
    C --> D[Service 4]
    D --> E[Service 5]
    E --> F[Service 6]
    F --> G[Service 7]
    G --> H[Service 8]
    H --> I[Service 9]
    I --> J[Service 10]
    J --> K[Service 11]
    K --> L[Service 12]
```

**Example input / output:**

| Service | Failure Rate (daily) | Error Budget Days |
|---------|----------------------|------------------|
| A       | 1.0                  | 6                |
| B       | 0.75                 | 9                |

**Data:**

```json
{
  "serviceA": {
    "failure_rate": 1.0,
    "error_budget_days": 6
  },
  "serviceB": {
    "failure_rate": 0.75,
    "error_budget_days": 9
  }
}
```

