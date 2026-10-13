---
type: "practice"
difficulty: "Expert"
---

# Justify to a cost-conscious CFO why the shared gateway line item tripled; design a FinOps model that attributes gateway cost per team and per API, exposes the chattiest consumers, and creates incentives to reduce egress and per-request overhead.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Cost Metering and Budgets](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/Gateway_Cost_Metering_and_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In our organization, we've seen a significant increase in our "shared gateway" line item from an unexpected source—our API Gateway service costs have tripled. This spike isn't just concerning; it also presents us with an opportunity to optimize and allocate these costs more effectively. As a Fullstack Architect/Principal, my responsibility is not only to justify why this cost has increased but also to design a FinOps model that will help us understand where the costs are being incurred, incentivize teams to reduce egress, and minimize per-request overhead.

#### Triage

The CFO wants to know why the shared gateway line item tripled. This implies a need to first verify if there was indeed an increase in usage or if it's due to a change in pricing terms. We'll also need to ensure that our API Gateway is being utilized efficiently across teams and APIs, and that we're not overpaying for services.

#### Assumptions
The cost has tripled within the last fiscal year.
There have been no significant changes in the gateway service contract.
Teams are using the API Gateway as intended without excessive overhead or redundant calls.

#### First-Principles Solution

To address this issue, we'll follow these steps:

Verify Usage Trends: Analyze historical usage data to identify if there's an actual increase in requests.
Cost Breakdown: Decompose the gateway costs to understand the components contributing to the increased expenses.
Per-Team and Per-API Allocation: Design a model that allocates gateway costs based on team and API usage.
Identify Chatty Consumers: Pinpoint which teams and APIs are consuming the most resources, leading to higher egress.
Incentivize Cost Reduction: Develop mechanisms to encourage efficient use of the API Gateway.
Step 1: Verify Usage Trends

First, we need to validate if there's a genuine increase in usage or if it’s due to other factors such as changes in the billing cycle. We'll gather data on request counts, response times, and any anomalies in the system logs over the past year.

SQL
SELECT
    DATE_TRUNC('month', timestamp) AS month,
    COUNT(*) AS total_requests
FROM api_gateway_logs
GROUP BY month
ORDER BY month ASC;
Step 2: Cost Breakdown

Next, we'll break down the costs to identify if it's due to higher rates or increased usage. Typically, API Gateway services have multiple components such as egress, API calls, and storage.

JSON
{
    "costComponents": [
        {"name": "Egress", "value": 0.015},
        {"name": "API Calls", "value": 0.02},
        {"name": "Storage", "value": 0.03}
    ]
}
Step 3: Per-Team and Per-API Allocation

We'll allocate costs based on the usage patterns of each team and API. This involves creating a detailed billing report that maps out who is using what.

API Team X
Calls to API A
Calls to API B
API Team Y
Calls to API A
Calls to API C
Step 4: Identify Chatty Consumers

Using the allocation model, we can identify which teams and APIs are contributing most to the increased costs.

SQL
SELECT
    team_id,
    api_name,
    SUM(egress_bytes) AS total_egress
FROM gateway_usage_metrics
GROUP BY team_id, api_name
ORDER BY total_egress DESC;
Step 5: Incentivize Cost Reduction

To drive efficiency, we can introduce financial incentives for teams that reduce their egress or minimize per-request overhead.

Cost Savings: Provide a rebate system where each dollar saved in gateway costs is rewarded to the respective team.
Efficiency Awards: Recognize and reward teams with the lowest egress usage or highest optimization efforts.

#### Closing Summary

By following these steps, we can not only justify why the shared gateway line item has tripled but also take proactive measures to optimize its use. This approach ensures that our API Gateway service is utilized more efficiently while maintaining cost control.

#### Common Mistakes
and Follow-Ups
Ignoring Historical Data: Overlooking past usage trends can lead to incorrect assumptions about current costs.
Lack of Incentives: Without mechanisms to encourage efficiency, the financial model may not yield desired results.
Insufficient Monitoring: Failing to monitor gateway usage regularly can result in unnoticed inefficiencies.

**Comparison:**

| team_id | api_name  | total_egress |
|---------|-----------|--------------|
| T1      | API-A     | 15000        |
| T2      | API-B     | 12000        |
| T3      | API-C     | 8000         |

**Code:**

```sql
-- SQL Query for verifying usage trends
SELECT
    DATE_TRUNC('month', timestamp) AS month,
    COUNT(*) AS total_requests
FROM api_gateway_logs
GROUP BY month
ORDER BY month ASC;

-- SQL Query for cost components breakdown
WITH cost_components AS (
    SELECT
        'Egress' AS component, 0.015 AS value
    UNION ALL
    SELECT
        'API Calls', 0.02
    UNION ALL
    SELECT
        'Storage', 0.03
)
SELECT * FROM cost_components;

-- SQL Query for identifying chatty consumers
SELECT
    team_id,
    api_name,
    SUM(egress_bytes) AS total_egress
FROM gateway_usage_metrics
GROUP BY team_id, api_name
ORDER BY total_egress DESC;
```

**Example input / output:**

```text
Input:
- SQL Query for verifying usage trends

Output:
+-------------------+----------------+
| month             | total_requests |
+-------------------+----------------+
| 2022-01-01        | 500            |
| 2022-02-01        | 600            |
| 2022-03-01        | 700            |
| 2022-04-01        | 800            |
+-------------------+----------------+

Input:
- SQL Query for cost components breakdown

Output:
+----------+-------+
| component| value |
+----------+-------+
| Egress   | 0.015 |
| API Calls| 0.02  |
| Storage  | 0.03  |
+----------+-------+

Input:
- SQL Query for identifying chatty consumers

Output:
+--------+-----------+--------------+
| team_id| api_name  | total_egress |
+--------+-----------+--------------+
| T1     | API-A     | 15000        |
| T2     | API-B     | 12000        |
| T3     | API-C     | 8000         |
+--------+-----------+--------------+
```

**Data:**

```json
{
    "costComponents": [
        {"name": "Egress", "value": 0.015},
        {"name": "API Calls", "value": 0.02},
        {"name": "Storage", "value": 0.03}
    ]
}
```

**Flow:**

1. Verify Usage Trends
2. Analyze Historical Data
3. Confirm Cost Increase Reason
4. Cost Breakdown
5. Per-Team and Per-APIAllocation
6. Identify Chatty Consumers
7. Incentivize Cost Reduction
8. Monitor and Optimize

