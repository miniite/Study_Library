---
type: "practice"
difficulty: "Deep"
---

# What is an SLA an SLO and an SLI and how are they related?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In cloud engineering, Service Level Agreements (SLAs), Service Level Indicators (SLIs), and Service Level Objectives (SLOs) form a critical trio that ensures service quality and reliability in modern cloud-based systems. TL;DR: SLAs are formal agreements with customers defining the expected performance of a service, SLOs are quantifiable targets for key metrics, and SLIs measure how well we're meeting these targets.

#### Triaging the Question

This question is about understanding the foundational concepts in cloud operations—specifically, the roles of SLA, SLO, and SLI. The interviewer is probing your knowledge of service management practices, your ability to differentiate between related but distinct terms, and your understanding of how they interrelate.

#### First-Principles Solution

Service Level Agreement (SLA): This is a legal contract or a mutual agreement that formalizes the expected performance of a service. It defines what users can expect from the provider in terms of uptime, availability, response times, and other quality metrics. For example, an SLA might promise 99.95% uptime with penalties for non-compliance.

Service Level Indicator (SLI): An SLI is a metric or measurement used to track progress towards an SLO. It quantifies the service performance in a way that can be objectively measured and reported. Examples include response time, error rate, or request latency.

Service Level Objective (SLO): An SLO sets specific numerical goals for SLIs based on business priorities. For instance, if your company values fast response times highly, an SLO might state that 95% of requests should be responded to within 100 milliseconds. This is a target we aim to hit as part of our SLA.

How They Are Related
SLAs Drive SLOs: The terms in the SLA set the stage for what needs to be achieved. SLOs are derived from these agreements.
SLOs Inform SLIs: Once the targets are set, we use SLIs to measure how well we are meeting those objectives.
SLIs Provide Visibility: By tracking SLIs, teams can monitor and adjust their operations to ensure they meet the SLOs specified in the SLA.

#### Closing Summary

Understanding these concepts is crucial for managing cloud services effectively. An SLA provides a framework, while SLOs set specific goals, and SLIs offer real-time visibility into performance. Misalignment between any of these can lead to poor service quality and customer dissatisfaction.

#### Common Mistakes/Follow-Ups
Common Mistake: Confusing the terms or not understanding their hierarchical relationship.

#### Follow-Up
Discussing practical examples where each term is used in a cloud environment, such as AWS Service Level Agreements for RDS, SLOs for monitoring and alerting, and SLIs like response time.

**Code:**

```
MERMAID
Copy
graph TD;
    A[SLA] --> B[Derives Goals];
    B --> C[SLO];
    C --> D[Metric to Measure];
    D --> E[SLI];
```

**Flow:**

1. Service Level Agreement
2. Service Level Objective
3. Service Level Indicator
4. Metric to MeasurePerformance

