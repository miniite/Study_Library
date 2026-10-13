---
type: "practice"
difficulty: "Deep"
---

# How do you use ATAM findings to negotiate the boundary of an internal SLA between the GenAI platform team and a product team consuming the assistant API? Show how a sensitivity point on model-provider latency becomes a formal dependency contract with a measured response goal.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Regulatory Requirements in Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To use Architectural Trade-off Analysis (ATAM) findings to negotiate an internal Service Level Agreement (SLA) between the GenAI platform team and the product team consuming the assistant API, we need a systematic approach that involves analyzing trade-offs at multiple levels of abstraction. ATAM helps identify key sensitivities in the system, such as model-provider latency, and translates these into concrete performance goals and dependencies.

In this scenario, we're focusing on the boundary between two teams: the GenAI platform team, which is responsible for the underlying AI models and infrastructure, and the product team, which develops applications that consume the assistant API. The primary sensitivity point is the model-provider latency—the time taken by the GenAI platform to generate responses.

#### Triage

The core question here is how to formalize the relationship between these two teams into a clear SLA with measurable goals. ATAM involves examining various trade-offs, such as response time, reliability, cost, and resource utilization, to ensure that both parties understand the dependencies and constraints.

#### First-Principles Solution

Define Sensitivity Points:

The model-provider latency is a critical sensitivity point because it directly affects user experience.

Identify Dependencies:

The GenAI platform team's performance in terms of latency will impact the product team’s ability to deliver a smooth and responsive user interface.

Formalize SLA with Measured Goals:

Define a goal that measures the acceptable range for model-provider latency, e.g., 500 milliseconds (ms) or less.
Establish a response time requirement from the product team’s perspective, which is typically tied to user satisfaction and usability.

Sensitivity Analysis:

Conduct sensitivity analysis to understand how changes in GenAI platform performance affect the overall system's behavior.
For instance, if the model-provider latency increases by 200 ms due to a new update, it might impact the product team’s ability to meet their response time requirements.

Negotiate Dependencies:

Use ATAM findings to negotiate a formal dependency contract that includes performance targets and service level objectives (SLOs).
For example, if the GenAI platform consistently meets its latency goal, the product team should be able to meet theirs. Conversely, if the GenAI platform fails to meet its goals, it might need to provide compensatory services or adjustments.

Monitor and Adjust:

Set up monitoring systems to continuously track both performance metrics.
Regularly review and adjust the SLA as needed based on actual performance data.

#### Closing Summary

By using ATAM findings, we can systematically identify key dependencies and trade-offs between the GenAI platform team and the product team. This process ensures that both teams understand their roles and responsibilities in maintaining a smooth user experience. The formalized SLA becomes a living document that evolves with the system’s needs.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to include enough sensitivity points or trade-offs, leading to an incomplete understanding of the dependencies.
Mistake 2: Setting unrealistic performance goals without considering practical constraints and resource limitations.

#### Follow-Up
Ensure that all stakeholders are involved in the negotiation process, including both technical and business representatives.

**Code:**

```
MERMAID
Copy
graph TD;
    A[GenAI Platform Team] --> B[Model-Provider Latency];
    B --> C[Product Team];
    C --> D[Response Time Goal];
    E[SLA Negotiation] --> F[Dependency Contract];
```

**Example input / output:**

```text
- **Input:** GenAI platform latency of 600 ms.
- **Output:** Product team response time is affected, requiring adjustments or compensatory measures.
```

**Data:**

```yaml
model_provider_latency: 500ms
response_time_goal_product_team: 1s
```

