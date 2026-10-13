---
type: "practice"
difficulty: "Deep"
---

# A principal probe: you're asked why the AIOps team runs its own cost-forecasting model when a commercial FinOps SaaS already forecasts spend. Defend keeping the model in-house because it fuses telemetry and change-events the SaaS can't see, and set the switching criteria under which you'd retire your model for the vendor's.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost and Reliability Correlation](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_and_Reliability_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this scenario, we need to justify keeping an in-house cost-forecasting model despite a commercially available FinOps SaaS that can also forecast spending. The key advantage lies in our ability to integrate telemetry and change events from AIOps systems, which the commercial solution cannot access. We will outline the criteria for switching to the vendor's solution if certain conditions are met.

#### Triage
The question is probing our understanding of the trade-offs between custom-built solutions and off-the-shelf tools, particularly in terms of data integration, flexibility, and cost governance. Our assumptions include having robust telemetry systems and a clear process for assessing the performance of both models. We will reason from first principles to evaluate these factors and establish switching criteria.

#### First-Principles Solution

Data Integration Capabilities:

Telemetry Data: The in-house model uses real-time AIOps telemetry data, providing immediate insights into infrastructure changes and usage patterns.
Change Events: It integrates change events from automated system changes, such as deployments or updates, which can significantly impact costs but might not be captured by a commercial SaaS.

Customization and Flexibility:

The in-house model allows us to tailor the forecasting logic to our specific infrastructure needs.
We can dynamically adjust the model based on evolving business requirements without waiting for vendor updates or customizations.

Performance Monitoring:

The accuracy of cost forecasts is crucial, especially during critical periods like budgeting and financial planning.
Frequent validation ensures that both models are accurate but gives us an edge in real-time adjustments due to our direct access to AIOps data.

Cost Considerations:

While the commercial SaaS might initially appear more cost-effective, we must consider long-term costs associated with integrating and maintaining it.
Custom solutions often have lower total ownership costs but require significant upfront investment in development and maintenance.

Switching Criteria:

Performance Metrics: Define key performance indicators (KPIs) like forecast accuracy, response time, and ease of integration with existing systems.
Budget Impact Analysis: Regularly assess the cost savings versus development/maintenance costs for both models.
Change Event Coverage: Evaluate how well the commercial solution can capture change events compared to our telemetry data.

#### Closing Summary
By maintaining an in-house model, we ensure that critical AIOps data is integrated seamlessly into our cost governance processes. This approach provides us with flexibility and real-time insights, which are vital for accurate and responsive financial management. However, the criteria outlined above will guide us on when to retire the custom model if a commercial solution proves more effective in terms of performance and cost.

#### Common Mistakes/Follow-Ups

Failing to establish clear KPIs for evaluating both models.
Overlooking long-term maintenance costs associated with custom solutions.
Not regularly validating forecast accuracy, leading to potential financial mismanagement.
Ignoring the importance of change-event data in accurate cost forecasting.

**Comparison:**

| Performance Metric | Custom Model | Vendor's SaaS |
|--------------------|--------------|---------------|
| Forecast Accuracy   | 98%          | 92%           |
| Response Time       | <1 second    | 5 seconds     |
| Integration Effort  | Low          | High          |
| Cost Decision Criteria | Value |
|------------------------|-------|
| Custom Solution Cost   | $50k  |
| Vendor's Savings      | $20k  |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Telemetry Data Integration] --> B[Custom Forecast Model];
    C[Vendor's SaaS Solution] --> D[Performance Validation];
    E[Change Event Handling] --> F[Custom Forecast Model];
    G[Budget Impact Analysis] --> H[Cost Decision Criteria];
    I[KPIs Established] --> J[Switching Criteria Defined];
```

**Example input / output:**

```text
Input: Telemetry data and change events from AIOps systems.
Output: Accurate cost forecasts integrated with financial planning tools.
Request: Perform a quarterly review of both models based on defined KPIs.
Response: In-house model outperforms SaaS in terms of forecast accuracy and response time, but vendor solution shows potential for long-term cost savings if integration is seamless.
```

**Data:**

```json
{
  "performance_metrics": ["forecast_accuracy", "response_time", "integration_effort"],
  "budget_impact_analysis": {
    "custom_solution_cost": "$50,000 per year",
    "vendor_solution_savings": "$20,000 per year"
  },
  "change_event_coverage": [
    {"event_type": "deployment", "coverage_custom": true, "coverage_vendor": false},
    {"event_type": "system_update", "coverage_custom": true, "coverage_vendor": false}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Model Comparison
4. Quarterly Review
5. KPIs Met?
6. Keep Custom Model
7. Evaluate Vendor's SaaS
8. Budget Impact Analysis
9. Cost Decision Criteria
10. Switch to Vendor if Better

