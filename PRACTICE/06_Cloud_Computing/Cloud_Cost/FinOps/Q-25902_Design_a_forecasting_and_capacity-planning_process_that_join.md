---
type: "practice"
difficulty: "Expert"
---

# Design a forecasting and capacity-planning process that jointly optimizes commitment purchases and hardware/quota reservations 18 months out, under uncertainty about product roadmap and pricing changes. How do you handle the compounding uncertainty?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a forecasting and capacity-planning process that optimizes commitment purchases and hardware/quota reservations 18 months out, we need to address several key challenges, including handling the compounded uncertainties from product roadmap changes and pricing fluctuations. This solution will involve a multi-step approach combining statistical models with machine learning techniques for demand forecasting, coupled with dynamic optimization methods to determine optimal purchase quantities.

#### Triaging the Question

The core of this question is about developing an effective process that can handle significant levels of uncertainty over a long horizon (18 months). The interviewer is probing our ability to manage complex, interdependent variables and make decisions based on probabilistic outcomes. We need to assume that product demand may change due to new features or market conditions, and pricing could shift unpredictably.

#### First-Principles Solution
Step 1: Demand Forecasting

We start with robust demand forecasting models. These models should be able to handle the uncertainty in both seasonality (e.g., quarterly cycles) and trend changes (e.g., new product launches). We can use time series analysis, such as ARIMA or exponential smoothing, but for more complex patterns, we might opt for machine learning techniques like Random Forests or Long Short-Term Memory (LSTM) networks.

Step 2: Cost Modeling

Next, we model the cost structure associated with different levels of commitment purchases. This includes fixed and variable costs, discounts for large commitments, and potential savings from bulk procurement. We also factor in the costs associated with over-provisioning or under-utilization during periods of uncertainty.

Step 3: Optimization Framework

We use an optimization framework to determine the optimal level of commitment purchase based on our demand forecasts and cost models. This could be a mixed-integer linear programming (MILP) model, where we define decision variables for different levels of commitment and constraints that reflect business rules and operational limits.

Step 4: Scenario Analysis

Given the significant uncertainty, we incorporate scenario analysis to evaluate multiple possible futures. For instance, we might consider three scenarios:

Best Case: All planned products are launched successfully.
Most Likely Case: Market conditions remain stable but with some new product launches.
Worst Case: Market competition intensifies, and demand falls short of expectations.

For each scenario, we run our optimization model to determine the optimal commitment levels.

Step 5: Regular Reevaluation

The process should be iterative and flexible. We need mechanisms for regular re-evaluation at least quarterly to adjust forecasts based on actual performance data and any changes in the product roadmap or market conditions.

#### Closing Summary
& Common Mistakes

This approach ensures that we maintain a balance between flexibility (to adapt to changing conditions) and stability (to avoid unnecessary costs from frequent changes). Key mistakes often involve ignoring the dynamic nature of demand, over-relying on historical data without considering future trends, and failing to adequately account for potential cost savings or penalties.

Common Follow-Ups
How do you handle seasonal variations in demand?
What are your thoughts on using real-time data streams (e.g., from IoT devices) to improve forecasts?
How would you integrate feedback loops into this process?

**Comparison:**

```
Setup Data Collection
Define Models
Fit SARIMA Model
Validate Model
Deploy Model
Define Cost Structure
Validate Costs
Deploy Cost Model
Set Up Optimization Framework
Validate Optimizations
Implement Optimizations
Initial Setup
Demand Forecasting
Cost Modeling
Optimization
Forecast and Capacity Planning Timeline
```

**Code:**

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX
import pandas as pd

# Example of a time series model
def fit_sarima_model(data, seasonal_order=(1, 1, 1, 12)):
    model = SARIMAX(data, order=(1, 1, 1), seasonal_order=seasonal_order)
    results = model.fit(disp=False)
    return results

# Example data
data = pd.Series([10, 15, 20, 30, 40, 50, 60, 70, 80, 90, 100], index=pd.date_range(start='2023-01-01', periods=11))
model = fit_sarima_model(data)
forecast = model.forecast(steps=10)
print(forecast)
```

**Example input / output:**

| Month       | Demand (Forecast) |
|-------------|-------------------|
| 2023-01     | 10                |
| 2023-02     | 15                |
| 2023-03     | 20                |
| 2023-04     | 30                |
| 2023-05     | 40                |
| 2023-06     | 50                |
| 2023-07     | 60                |
| 2023-08     | 70                |
| 2023-09     | 80                |
| 2023-10     | 90                |
| 2023-11     | 100               |

**Data:**

```json
{
  "data": [
    {"month": "2023-01", "demand": 10},
    {"month": "2023-02", "demand": 15},
    {"month": "2023-03", "demand": 20},
    {"month": "2023-04", "demand": 30},
    {"month": "2023-05", "demand": 40},
    {"month": "2023-06", "demand": 50},
    {"month": "2023-07", "demand": 60},
    {"month": "2023-08", "demand": 70},
    {"month": "2023-09", "demand": 80},
    {"month": "2023-10", "demand": 90},
    {"month": "2023-11", "demand": 100}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a new product launch?
5. Update Demand Forecast
6. Run Optimization Model
7. Evaluate Cost Savings
8. Implement Optimizations
9. Monitor Performance

