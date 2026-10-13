---
type: "practice"
difficulty: "Deep"
---

# How do you run an AIOps platform ATAM under acquisition-integration pressure where two companies' observability stacks (different agents, TSDBs, and incident tools) must interoperate in six months? Which scenarios expose whether integrate-at-the-edge (shared alert bus) or deep-merge is the lower-risk path?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ ATAM for Build vs Buy](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Build_vs_Buy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When integrating two AIOps platforms under tight time constraints, we need to conduct an Architectural Trade-off Analysis (ATAM) to evaluate and mitigate risks associated with different integration strategies. Specifically, this involves comparing "integrate-at-the-edge" (shared alert bus) and "deep-merge" approaches in the context of merging observability stacks. The goal is to ensure seamless interoperation while managing complexities such as data consistency, alerting policies, and operational efficiency.

#### Triaging the Question

The question asks us to analyze how to integrate two AIOps platforms where:

Different companies have distinct observability stacks (agents, TSDBs, incident tools).
The integration must be completed within six months.
We need to identify scenarios that favor "integrate-at-the-edge" or "deep-merge."

#### Assumptions

Both systems have robust data and alerting capabilities.
There are no existing shared services between the two companies.
The platforms use different technologies for their observability stacks.

#### First-Principles Solution
Step 1: Define High-Level Goals

Our primary goals are:

Ensure seamless integration with minimal downtime.
Maintain operational efficiency post-integration.
Handle alerting and incident management effectively across both systems.
Step 2: Identify Key Components to Integrate

We need to consider the following components for integration:

Observability agents (e.g., Jaeger, Zipkin)
Time Series Database (TSDB) systems (e.g., Prometheus, InfluxDB)
Incident Management tools (e.g., PagerDuty, OpsGenie)
Step 3: Evaluate "Integrate-at-the-Edge" Approach

#### Pros

Minimal Overhead: Agents can be unified using a shared alert bus.
Simpler Deployment: New agents can be deployed without altering existing TSDBs or incident tools.
Scalability: Centralized processing of alerts reduces the load on individual systems.

#### Cons

Complex Alert Correlation: Ensuring that alerts from different sources are correctly correlated and prioritized may introduce complexity.
Data Consistency Issues: Potential for data skew if not handled carefully.
Operational Risk: New dependencies can increase operational risk due to shared bus failures.
Step 4: Evaluate "Deep-Merge" Approach

#### Pros

High Data Integrity: Merging TSDBs ensures a unified view of all time-series data.
Unified Incident Management: Centralized incident management tools provide a consistent user experience.
Predictable Performance: No introduction of new dependencies reduces operational risk.

#### Cons

Complexity in Integration: Requires significant effort to merge and validate historical data across both TSDBs.
Potential for Data Inconsistencies: If not managed properly, merging could lead to duplicated or conflicting data points.
Longer Deployment Time: Due to the complex nature of merging systems, this approach may require more time than the "integrate-at-the-edge" method.
Step 5: Scenario Analysis

To determine which approach is lower-risk:

Alert Correlation and Prioritization: Test how well alerts from different sources can be correlated and prioritized using a shared alert bus.
TSDB Merging Feasibility: Assess the feasibility of merging TSDBs by analyzing historical data and identifying potential issues.
Operational Impact Analysis: Evaluate the operational impact of each approach, considering factors like downtime, training requirements, and resource allocation.

#### Closing Summary

Based on our analysis, "integrate-at-the-edge" might be a lower-risk path due to its simpler deployment and reduced complexity in data management. However, this depends heavily on the specific details of both observability stacks and the potential for alert correlation issues. The approach that best fits should be validated through detailed scenario testing and risk mitigation plans.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Not fully considering edge cases like overlapping alert conditions or data inconsistencies.
Underestimating Complexity: Overlooking the complexity of merging TSDBs, which can lead to significant downtime.
Failing to Plan Adequately: Insufficient planning for integration testing and operational support during transition.

**Comparison:**

```
Approach	Pros	Cons
Integrate-at-the-edge	Minimal overhead, simpler deployment	Complex alert correlation, potential for data consistency issues
Deep-merge	High data integrity, unified incident management	Complex integration, risk of operational impact
```

**Code:**

```python
# Example of a simple alert correlation function (hypothetical)
def correlate_alerts(alert1, alert2):
    # Correlate based on common attributes like service name, timestamp
    return True if are_similar(alert1, alert2) else False

# Example of an incident management merge strategy
def deep_merge_incidents(incident1, incident2):
    merged = {**incident1, **incident2}
    handle_conflicts(merged)
    return merged
```

**Example input / output:**

```text
Input:
- Existing Jaeger and Prometheus observability agents
- PagerDuty incident management tool

Output:
- Unified alert bus with shared alert correlation logic
- Merged Prometheus TSDB with historical data validation
- Centralized incident management using a single instance of PagerDuty
```

**Data:**

```json
{
  "agent": {
    "type": "Jaeger",
    "version": "0.54"
  },
  "tsdb": {
    "type": "Prometheus",
    "version": "2.18"
  },
  "incident_tool": {
    "type": "PagerDuty",
    "version": "3.0"
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Alert CorrelationIssues
4. Can Alerts Be Correlated?
5. Integrate-at-the-edge
6. Merge TSDBs
7. Unified Alert BusImplementation
8. TSDB Merging Strategy

