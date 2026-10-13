---
type: "practice"
difficulty: "Intermediate"
---

# You need to forecast next quarter's cloud spend for a service that's growing. What bottoms-up signals do you use, and how do you communicate the uncertainty to finance?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To forecast next quarter’s cloud spend for a growing service in FinOps and Cloud Cost Governance, we need to look at several key bottom-up signals that provide visibility into cost drivers. These include historical spending patterns, upcoming projects, service utilization metrics, and market trends. Communicating the uncertainty to finance is critical to ensure realistic expectations and effective budget planning.

#### Triaging the Question

The question asks for bottom-up signals to forecast cloud spend and how to communicate uncertainty. The core of the question revolves around identifying key indicators that influence cloud costs and understanding their variability. The interviewer is probing your ability to decompose a complex financial forecasting problem into manageable components and to articulate the confidence in these forecasts.

#### Assumptions
Historical Spend Data: We have access to previous quarter’s spending data.
Service Growth Rate: The service growth rate is stable or can be estimated from historical trends.
Market Dynamics: There are no significant market changes that would drastically alter the cost structure.
Operational Metrics: We have metrics on service utilization, such as CPU and memory usage.
Step-by-Step Solution
Step 1: Identify Key Bottom-Up Signals
Historical Spend Breakdown: Review the previous quarter’s spending to identify the major components contributing to cloud costs (e.g., compute, storage, database).
Service Growth Rate: Determine the expected growth rate of the service based on historical data or current trends.
Utilization Metrics: Analyze metrics such as CPU and memory usage to predict potential increases in cost due to increased load.
Upcoming Projects: Identify any new projects that will require additional resources, which could increase spending.
Market Trends: Consider external factors like price changes or service upgrades.
Step 2: Quantify the Signals
Historical Spend Analysis:
Break down previous quarter’s spend by category (e.g., compute costs: $X, storage costs: $Y).
Calculate percentage contributions to total spend.
Growth Projections:
Use historical growth rates to project next quarter's service demand.
Apply the growth rate to existing cost drivers.
Utilization Metrics:
Project CPU and memory usage based on current trends.
Estimate additional costs due to increased resource consumption.
Upcoming Projects:
Identify projects that require cloud resources.
Estimate their potential impact on spend.
Step 3: Calculate the Forecast
Compute Costs:
Current compute cost = ( \text{CPU usage} \times \text{price per unit CPU} + \text{memory usage} \times \text{price per unit memory} ).
Projected next quarter’s compute cost = projected CPU and memory usage * current prices.
Storage Costs:
Current storage cost = ( \text{storage used} \times \text{price per unit storage} ).
Projected next quarter’s storage cost = projected storage usage * current prices.
Database Costs:
If applicable, consider database usage and associated costs.
Step 4: Communicate Uncertainty
Sensitivity Analysis: Perform a sensitivity analysis on key variables (e.g., growth rate, resource usage) to understand the impact of changes in these parameters.
Scenario Planning: Develop different scenarios (best case, worst case, most likely scenario).
Confidence Intervals: Provide confidence intervals for your forecasts based on historical data variability.

#### Closing Summary

By decomposing the forecast into key bottom-up signals and performing a thorough analysis, we can provide finance with a realistic estimate of next quarter’s cloud spend along with an understanding of associated uncertainties. This approach ensures that both technical and financial stakeholders have a clear view of potential costs and can make informed decisions based on accurate projections.

#### Common Mistakes/Follow-Ups
Ignoring Historical Variability: Over-relying on historical data without considering variability.
Failing to Consider Market Dynamics: Not accounting for external factors like price changes or new services.
Overlooking Upcoming Projects: Failing to include planned initiatives that could significantly impact costs.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Forecasting Cloud Spend
    section Historical Data
        complete 2019-04-01 23:59, 75%
        Work on historical spend breakdown :done 2019-04-01 23:59
    section Growth Projections
        start 2019-04-02
        duration 6m
        Work on growth rate analysis
    section Utilization Metrics
        start 2019-04-08
        duration 2w
        Work on CPU and memory usage metrics
    section Upcoming Projects
        start 2019-05-01
        duration 3m
        Identify and estimate impact of new projects
```

**Code:**

```python
# Example Python code for calculating projected compute cost

def project_compute_cost(current_cpu_usage, current_memory_usage, price_per_unit_cpu, price_per_unit_memory):
    return (current_cpu_usage * price_per_unit_cpu) + (current_memory_usage * price_per_unit_memory)

# Historical data example
cpu_usage = 500  # in hours
memory_usage = 100  # in GB
price_per_unit_cpu = 0.05  # $ per hour
price_per_unit_memory = 0.02  # $ per GB

projected_cost = project_compute_cost(cpu_usage, memory_usage, price_per_unit_cpu, price_per_unit_memory)
print(f"Projected Compute Cost: ${projected_cost}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Current CPU Usage: 500 hours<br>Current Memory Usage: 100 GB<br>Growth Rate: 20%<br>Current Prices:<br>   - Compute: $0.05/hour<br>   - Storage: $0.02/GB | Projected Next Quarter Compute Cost: $36, Projected Storage Cost: $8 |

**Data:**

```json
{
    "historical_data": {
        "cpu_usage": 500,
        "memory_usage": 100,
        "compute_price": 0.05,
        "storage_price": 0.02,
        "database_price": 0.03
    },
    "growth_rate": 1.2, // Assuming a growth rate of 20%
}
```

**Flow:**

1. Historical Spend Breakdown
2. Growth Rate Analysis
3. Utilization Metrics
4. Upcoming Projects
5. Compute Cost Calculation
6. Storage Cost Calculation
7. Total Forecast
8. Total Forecast
9. Scenario Planning
10. Confidence Intervals

