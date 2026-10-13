---
type: "practice"
difficulty: "Expert"
---

# A principal-level scenario: leadership wants a single 'reliability-and-cost health' dashboard where an executive sees SLO burn and $/hour burn side by side. Design the composite view so it doesn't let teams trade reliability for cost invisibly, and explain what a naive combined score would hide.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Healthy vs Wasteful Cost Growth](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Healthy_vs_Wasteful_Cost_Growth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a single 'reliability-and-cost health' dashboard that ensures teams don’t trade off reliability for cost invisibly, we need to first understand the core concepts of Service Level Objectives (SLOs) and financial metrics in the context of FinOps. Our goal is to create a composite view that provides both a clear picture of service reliability and cost efficiency at a glance.

#### Triaging the Question

The question asks us to design a dashboard that shows SLO burn alongside $/hour burn, ensuring transparency and preventing any trade-offs between reliability and costs. A naive combined score might obscure critical insights, leading to poor decision-making. We need to break down the problem into its fundamental components:

SLOs: Service Level Objectives are metrics used to measure the performance of a service against agreed-upon targets.
Cost Metrics: These include burn rates and other financial indicators that reflect cloud spending.

#### Trade-offs
Ensuring these two dimensions are not conflated but presented in a way that highlights any potential compromises.

#### First-Principles Solution

To achieve this, we need to:

Define the SLOs and Cost Metrics precisely for each service or application.
Create Separate Visualizations for SLO burn and $/hour burn.
Implement a Composite View that overlays these two metrics in a way that prevents any misleading combined scores.
Step-by-Step Solution
Define the SLOs: Establish clear, quantifiable targets for service reliability (e.g., 99.9% uptime). Monitor key performance indicators (KPIs) like error rates and availability.
Cost Metrics: Track cloud spending at a granular level, including hourly costs, usage patterns, and cost-per-unit of SLO achieved or violated.
Overlay Visualizations:
Use a bar chart for SLO burn, with each bar representing the difference between actual performance and target (SLO).
Overlay a line graph for $/hour burn, showing total costs over time.

#### Implementation
Dashboard Design
Bar Chart: Display the SLO burn percentage. A red color could indicate violations.
Line Graph: Show cumulative cost per hour in a contrasting color. Use different scales if necessary to ensure readability.
Service Reliability (SLO Burn)
Red: Violations, Green:Achieved
Cost Burn ($/hour)
Green: Within Budget, Red:Over Budget
Composite View
Combine Visuals: Use a dual-axis plot to overlay SLO burn on the cost graph. This ensures that any compromise in reliability is immediately visible.
Service Reliability (SLO Burn)
Red: Violations, Green:Achieved
Cost Burn ($/hour)
Green: Within Budget, Red:Over Budget
Dual-Axis Plot
Overlay SLO Burn on CostGraph

#### Closing Summary

By designing a dashboard that clearly shows both service reliability and cost efficiency side by side, we ensure transparency and prevent teams from inadvertently trading one for the other. The dual-axis plot allows executives to make informed decisions based on a holistic view of their cloud resources.

#### Common Mistakes/Follow-Ups
Naive Combined Scores: A single metric that combines SLO burn with $/hour burn might mask critical issues.
Insufficient Data Granularity: Not tracking costs at the appropriate level (e.g., by service or component) can lead to misleading insights.
Failure to Monitor Trade-offs: Failing to highlight instances where reducing costs results in lower reliability.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Service Reliability and Cost Burn Dashboard
    dateFormat YYYY-MM-DD
    section SLOs
    SLO_A: from "2023-05-01"  duration 60
    SLO_B: after SLO_A, 40
    section Costs
    Cost_A: from "2023-05-01"  duration 60
    Cost_B: after Cost_A, 40
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Service Reliability (SLO Burn)] --> B[Red: Violations, Green: Achieved];
    C[Cost Burn ($/hour)] --> D[Green: Within Budget, Red: Over Budget];
    E[Dual-Axis Plot] --> F[Overlay SLO Burn on Cost Graph];
```

**Example input / output:**

| Service | Target Uptime (%) | Current Uptime (%) | SLO Violation Count | Cost Burn Rate ($/hour) |
|---------|-------------------|--------------------|---------------------|------------------------|
| A       | 99.9              | 98.5               | 20                  | $20                    |
| B       | 99.9              | 99.7               | 10                  | $30                    |

**Data:**

```json
{
  "SLOs": {
    "ServiceA": {
      "targetUptime": 99.9,
      "currentUptime": 98.5,
      "violationCount": 20
    },
    "ServiceB": {
      "targetUptime": 99.9,
      "currentUptime": 99.7,
      "violationCount": 10
    }
  },
  "Costs": {
    "totalBurnRate": "$50/hour",
    "serviceABurnRate": "$20/hour",
    "serviceBBurnRate": "$30/hour"
  }
}
```

**Flow:**

1. Service Reliability (SLO Burn)
2. Red: Violations, Green:Achieved
3. Cost Burn ($/hour)
4. Green: Within Budget, Red:Over Budget
5. Dual-Axis Plot
6. Overlay SLO Burn on CostGraph

