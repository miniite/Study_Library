---
type: "practice"
difficulty: "Deep"
---

# How do you fold error budgets and SLO policy into the ATAM utility tree so that an availability scenario is analyzed against the negotiated budget rather than an aspirational 'always up'? Give the concrete response measure you attach.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Turning ATAM Findings into Principles](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Turning_ATAM_Findings_into_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When analyzing an availability scenario in the context of Architectural Trade-off Analysis (ATAM), it's crucial to integrate error budgets and Service Level Objective (SLO) policies to ensure that the system is evaluated against realistic, negotiated constraints rather than abstract "always up" targets. This approach helps in making informed decisions about trade-offs between performance, cost, and reliability.

To begin, let’s clarify what ATAM is: it's a structured method for analyzing architectural trade-offs by breaking down a complex system into smaller components and evaluating the impact of design choices on various stakeholders (e.g., users, developers, operations). Error budgets and SLO policies are key metrics that help us understand how much availability or performance degradation can be tolerated.

#### Triaging the Question

The core of this question is about integrating error budgets and SLOs into ATAM. Specifically, we need to ensure that when analyzing a system's availability scenario, we do so within the context of these negotiated constraints rather than an idealistic "always up" state. We must also determine what concrete response measures are appropriate given the budget.

#### First-Principles Solution

Understand Error Budgets:

An error budget is a quantifiable measure of how much downtime or degradation in service quality your system can sustain within an acceptable timeframe.
It acts as a safety margin that allows for planned outages, maintenance, and unexpected issues.

Define Service Level Objectives (SLO):

SLOs are the desired levels of service that you aim to achieve over time for specific services or SLIs (Service Level Indicators).
For example, an SLO might be that 99% of requests should complete in under 2 seconds.

Integrate Error Budgets and SLOs into ATAM:

During the ATAM utility tree analysis, each architectural component is evaluated based on its impact on the overall system.
For availability scenarios, we need to assess how changes affect the error budget and SLO compliance.
If a design choice would push the system beyond its negotiated error budget, it must be reconsidered.

Attach Concrete Response Measures:

Once you have identified a scenario that impacts the error budget or SLOs, determine appropriate response measures.
These could include:
Allocating more resources to improve performance and reduce errors.
Implementing redundancy and failover mechanisms.
Adjusting capacity planning to ensure adequate service levels.

#### Example Scenario

#### Cons
ider a system with an initial availability of 99.5% (an SLO) and a negotiated error budget of 0.5% downtime per month. A proposed architectural change aims to improve the response time for critical operations. If this change could potentially push the availability below 99%, we would need to analyze:

Impact on Error Budget: Calculate how much additional downtime is acceptable.
Response Measure: Determine if additional resources or a different architecture can be implemented without exceeding the error budget.

#### Closing Summary
and Common Mistakes

In summary, integrating error budgets and SLOs into ATAM ensures that architectural decisions are made with a realistic understanding of system limits. The key is to identify when proposed changes exceed these constraints and take appropriate measures to stay within them.

#### Common Mistakes

Ignoring the negotiated error budget in favor of idealistic performance goals.
Failing to quantify SLOs and error budgets accurately.
Overlooking the implications of architectural trade-offs on availability and service quality.

**Comparison:**

```
99.5%
0.7%
40%
Further Analysis
Error Budget
Initial Downtime
Current Availability
D
New Downtime
F
Impact Percentage
H
Action Required
J
```

**Code:**

```python
# Example Python code for calculating error budget impact
def calculate_error_budget_impact(current_downtime, new_downtime):
    return abs((current_downtime - new_downtime) / current_downtime * 100)

# Error budget and SLO example
current_downtime = 0.5  # 0.5% downtime per month (error budget)
new_downtime = 0.7     # Potential increase due to architectural change

impact_percentage = calculate_error_budget_impact(current_downtime, new_downtime)
if impact_percentage > 20:  # Threshold for significant impact
    print("Proposed change may exceed error budget; further analysis required.")
else:
    print("Change is within acceptable limits based on current error budget.")
```

**Example input / output:**

| Input                  | Output                                            |
|------------------------|--------------------------------------------------|
| Current Downtime (0.5%) | New Downtime (0.7%), Impact Percentage (40%)      |
|                       | Change exceeds error budget; further analysis     |
|                       | required.                                         |

**Data:**

```json
{
  "error_budget": {
    "current_downtime_percentage": 0.5,
    "threshold_percentage": 20
  },
  "slo": {
    "target_percentage": 99.5,
    "service_level_indicator": [
      {"operation": "critical", "response_time": "<=2ms"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Propose Change
4. Calculate Downtime Impact
5. Exceeds Error Budget?
6. Adjust Architecture/ResourceAllocation
7. Continue with Change
8. Monitor New Availability

