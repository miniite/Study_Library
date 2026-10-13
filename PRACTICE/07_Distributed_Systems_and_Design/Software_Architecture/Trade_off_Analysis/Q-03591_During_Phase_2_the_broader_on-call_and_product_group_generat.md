---
type: "practice"
difficulty: "Deep"
---

# During Phase 2 the broader on-call and product group generates scenarios that contradict the priorities the AIOps architects set in Phase 1 (on-call wants fewer pages, product wants more coverage). How do you reconcile the two prioritizations and what does the divergence tell you about org alignment?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scenario Prioritization](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Scenario_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we are dealing with a classic case of organizational conflict where different groups within an organization (on-call and product) have conflicting priorities in AIOps architecture design. The goal is to reconcile these differing viewpoints while ensuring the overall health and effectiveness of the system. This reconciliation involves understanding the underlying drivers of each group's priorities and finding a balanced approach that aligns with both short-term operational needs and long-term strategic goals.

#### Triage

The core issue here is organizational alignment—specifically, how to integrate the varying perspectives from the on-call team (focusing on minimizing downtime and reducing alert noise) and the product team (aiming for comprehensive coverage and detailed insights). This requires a deep dive into the trade-offs between operational efficiency and feature completeness. We need to evaluate the potential impact of each group's priorities on system reliability, maintainability, and user satisfaction.

#### Assumptions
On-call Team: Prioritizes reducing alert fatigue and ensuring minimal downtime.
Product Team: Focused on comprehensive monitoring and detailed insights for product development.
AIOps Architects: Responsible for designing the architecture that best meets operational needs while supporting product requirements.

#### First-Principles Solution

To reconcile these priorities, we can follow a structured ATAM (Architectural Trade-off Analysis Method) framework:

Identify Key Stakeholders and Their Goals

On-call: Minimize alert fatigue.
Product: Maximize monitoring coverage.
AIOps Architects: Balance operational efficiency with feature completeness.

Model the Architecture

Define the current architecture and its performance metrics (e.g., number of alerts, system uptime).
Use a simple model to represent different architectural choices:
Current State
Reduce Alert Fatigue
Comprehensive Monitoring
Enhance FeatureCompleteness
Architectural Trade-offs
Optimized State

Analyze the Impact of Each Priority

Evaluate how reducing alerts impacts system performance and maintainability.
Assess the benefits and costs of increasing monitoring coverage.

Quantify the Trade-offs

Use metrics such as Mean Time to Recovery (MTTR), alert accuracy, and feature completeness score.
For example:
table
|Metric| On-call Focused | Product Focused |
|---|---|---|
|Alerts per Hour| 500| 200|
|Uptime| 98.5%| 97.8%|
|Feature Completeness| N/A| 95%|

Find a Balanced Solution

Determine the optimal balance between reducing alerts and enhancing monitoring coverage.

#### Cons
ider implementing a phased approach, starting with small, incremental changes.

Implement and Monitor

Deploy the chosen architecture in a controlled manner.
Continuously monitor the system’s performance and gather feedback from both on-call and product teams.

#### Closing Summary

The divergence between on-call and product priorities highlights the need for an integrated approach to AIOps architecture design that considers both operational efficiency and feature completeness. By using ATAM, we can systematically analyze and balance these conflicting objectives, ensuring a more robust and user-friendly system.

#### Common Mistakes/Follow-Ups
Ignoring Feedback: Not actively seeking feedback from end-users (on-call engineers and product managers) can lead to suboptimal solutions.
Overcomplicating the Solution: Sometimes, simpler architectural choices that address both groups' needs are more effective than complex ones.
Lack of Continuous Monitoring: Failing to continuously monitor system performance post-deployment can result in unidentified issues.

To ensure a successful reconciliation, it’s crucial to maintain open communication channels and involve all stakeholders throughout the process.

**Code:**

```

graph TD; A[Current State] --> B[Reduce Alert Fatigue]; C[Comprehensive Monitoring] --> D[Enhance Feature Completeness]; E[Architectural Trade-offs] --> F[Optimized State];

CODE
Copy

```markdown
|Metric| On-call Focused | Product Focused |
|---|---|---|
|Alerts per Hour| 500| 200|
|Uptime| 98.5%| 97.8%|
|Feature Completeness| N/A| 95%|
```

**Flow:**

1. Current State
2. Reduce Alert Fatigue
3. Comprehensive Monitoring
4. Enhance FeatureCompleteness
5. Architectural Trade-offs
6. Optimized State

