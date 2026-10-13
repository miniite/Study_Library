---
type: "practice"
difficulty: "Expert"
---

# How do you use ATAM outputs to negotiate the boundary of an SLA with a downstream internal platform team? Show how a sensitivity point on their latency becomes a formal dependency contract with a measured response goal.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ ATAM for Build vs Buy](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Build_vs_Buy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When negotiating Service Level Agreements (SLAs) with a downstream internal platform team, using Architectural Trade-off Analysis Methodology (ATAM) helps us understand and communicate the trade-offs involved in different design decisions. Specifically, when focusing on latency sensitivity points of their system, we can translate these into formal dependency contracts that both teams can agree upon.

First, let's summarize what ATAM does: It systematically identifies and evaluates the trade-offs between different architectural designs to ensure they meet organizational goals while considering various constraints such as performance, cost, and risk. In this context, our goal is to make sure that any latency concerns are clearly defined and incorporated into a concrete SLA.

#### Triage

The question asks how we use ATAM outputs to negotiate the boundary of an SLA with a downstream platform team, particularly focusing on their latency sensitivity points. The key here is to ensure that these sensitivity points become formal dependency contracts with measurable response goals. Essentially, this involves:

Identifying Latency Sensitive Points: Determine which parts of the system are most sensitive to delays.
Understanding Their Impact: Evaluate how these latencies impact our overall system performance and business objectives.
Formulating SLA Requirements: Create clear, measurable goals for these latency points.

#### First-Principles Solution

Let's walk through each step:

Identify Latency Sensitive Points:

Conduct a thorough analysis of the downstream platform’s critical paths and bottlenecks.
Use performance testing tools to identify where delays occur under various load conditions.
Prioritize these points based on their impact on user experience and business outcomes.

Understand Their Impact:

Quantify the impact of each latency point. For example, how does a 100ms delay in a critical transaction affect conversion rates?
Use historical data or simulations to predict potential failures under high load conditions.

Formulate SLA Requirements:

Define clear response goals for these latency points. For instance, "95% of requests should complete within 200ms."
Ensure the goals are realistic and achievable by the downstream platform team.
Agree on monitoring tools and metrics to track performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using ATAM outputs, we can effectively negotiate an SLA that addresses critical latency points. The key is to translate these into measurable goals and formal dependency contracts. Common mistakes include:

Failing to identify all sensitive points: It’s crucial to thoroughly analyze the system.
Unrealistic or unachievable goals: Goals must be realistic but still challenging enough to drive improvement.
Lack of monitoring and feedback: Regularly tracking performance ensures that both teams stay aligned.

#### Follow-Ups
What if the downstream team cannot meet these SLA requirements?
Explore alternative solutions such as caching, async processing, or even service degradation strategies.
How do you handle unexpected changes in latency sensitivity points over time?
Revisit and update the ATAM every quarter to reflect new insights and changing business needs.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Identify Latency Sensitive Points] --> B[Quantify Impact]
    B --> C[Formulate SLA Requirements]
```

**Flow:**

1. Identify Latency SensitivePoints
2. Quantify Impact
3. Formulate SLA Requirements

