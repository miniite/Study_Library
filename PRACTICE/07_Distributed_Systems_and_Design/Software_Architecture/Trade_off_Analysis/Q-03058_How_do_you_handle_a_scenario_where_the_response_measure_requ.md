---
type: "practice"
difficulty: "Deep"
---

# How do you handle a scenario where the response measure requires data you would have to spend three weeks instrumenting to obtain? Does ATAM let you defer with a 'measurement gap' finding, and how do you keep that from being a way to dodge the analysis?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Handling Uncertain Response Measures](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Handling_Uncertain_Response_Measures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the scenario where obtaining critical response data requires an unfeasible amount of time (three weeks), we can leverage Architectural Trade-off Analysis Methodology (ATAM). ATAM helps us understand and evaluate trade-offs between different architectural decisions, even in the absence of complete data. Here’s a detailed breakdown of how to approach this:

Summary

We’ll first reframe the problem using ATAM principles to identify potential trade-offs and gaps in measurement. Then, we'll explore strategies for managing these gaps while ensuring that our analysis remains robust and meaningful.

#### Triaging the Question

The question is essentially asking whether ATAM allows us to defer making an informed decision due to missing data and how to avoid such deferrals becoming an excuse to sidestep critical analysis. ATAM involves systematic evaluation of architectural choices, but it requires careful handling when faced with incomplete data.

Key Assumptions
Data Criticality: The missing data is crucial for the final design decisions.
Time Constraints: Three weeks are required to obtain the necessary data.

#### Trade-offs
There might be other factors (e.g., cost, risk) that can inform our decision.
Step-by-Step Solution

Identify Critical Decisions:

Determine which architectural decisions will significantly impact the project's success if based on incomplete data.

Assess Uncertainty:

Use ATAM to quantify the uncertainty associated with missing data.

#### Cons
ider the potential impact of this uncertainty on key performance indicators (KPIs).

Define Measurement Gaps:

Clearly document what specific measurements are needed and why they are critical.
Highlight how these gaps affect the overall architecture.

Evaluate Alternatives:

Identify alternate architectural designs or solutions that can mitigate the need for this data.
Evaluate trade-offs between different approaches, such as cost, risk, and time-to-market.

Risk Mitigation Strategies:

Develop contingency plans to address potential gaps in data collection.

#### Cons
ider phased implementations where critical components are developed first, even if full data is not available initially.

Stakeholder Communication:

Communicate the rationale for making decisions based on partial data to stakeholders.
Ensure transparency about the risks and uncertainties involved.

#### Closing Summary

While ATAM provides a structured approach to handle complex architectural trade-offs, it’s crucial to manage measurement gaps carefully. By clearly documenting these gaps and developing strategies to mitigate their impact, we can make informed decisions without deferring analysis indefinitely.

#### Common Mistakes/Follow-Ups
Deferring Analysis: Avoid using the ‘measurement gap’ as an excuse to avoid making difficult architectural choices.
Inadequate Documentation: Poorly documented gaps and trade-offs can lead to confusion and lack of transparency in decision-making processes.
Ignoring Risks: Failing to adequately assess and communicate risks associated with missing data can result in significant project setbacks.

**Comparison:**

```
Decision	Impact	Critical Data Needed	Time Required
A	High	Latency Metrics	1 week
B	Medium	User Engagement Data	3 weeks
Measurement Gaps	Mitigation Strategy
No engagement data	Phase-wise implementation, focus on latency first
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Critical Decisions] --> B{Are Datas Needed Critical?};
    B -- Yes --> C[Assess Uncertainty];
    C --> D[Document Measurement Gaps];
    C --> E[Evaluate Alternatives];
    E --> F[Risk Mitigation Strategies];
    D --> F;
    F --> G[Stakeholder Communication];
    A --> H[Make Informed Decisions];
```

**Example input / output:**

```text
Input:
- Decision to choose between two architectural designs (A and B)
- Required data: Latency Metrics and User Engagement Data

Output:
- Architectural Design A selected due to higher cost-effectiveness despite missing engagement metrics
- Risk mitigation strategy implemented for design B pending additional data collection
```

**Data:**

```yaml
- decision: Architectural Design A vs. B
  impact: High
  data_needed: Latency Metrics, User Engagement Data
  time_required: 3 weeks
```

**Flow:**

1. Yes
2. Identify Critical Decisions
3. Are Datas Needed Critical?
4. Assess Uncertainty
5. Document Measurement Gaps
6. Evaluate Alternatives
7. Risk Mitigation Strategies
8. Stakeholder Communication
9. Make Informed Decisions

