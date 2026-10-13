---
type: "practice"
difficulty: "Expert"
---

# How do you build the negotiation leverage before a hyperscaler EDP renewal — usage growth projections, competitive quotes, workload-portability signals — without bluffing a migration you can't actually execute?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To navigate the complex landscape of negotiating with a hyperscaler for an Enterprise Discount Program (EDP) renewal while building robust negotiation leverage, we must approach the process methodically. This involves understanding your current and projected usage growth, securing competitive pricing quotes from multiple sources, and preparing workload-portability signals that demonstrate your capability to migrate if necessary. The key is to avoid bluffing or making unfeasible threats, which can backfire and harm long-term relationships.

The core of this solution lies in leveraging data-driven insights and a strategic approach. Let's break it down step-by-step:

Usage Growth Projections: Accurate and transparent usage growth projections are crucial. You need to have a solid understanding of your current spend, historical trends, and forecasted growth based on business needs and market conditions.
Competitive Quotes: Obtain competitive quotes from other hyperscalers or managed service providers (MSPs). This will give you leverage in negotiations by showing how much you can save elsewhere.
Workload Portability Signals: Prepare to demonstrate the ease with which you can migrate workloads if necessary. This might involve documenting your current infrastructure, APIs, and integration points.

The key assumptions are:

You have a clear understanding of your current usage patterns.
Your business is financially healthy enough to withstand short-term costs during potential migrations.
You have well-documented plans for workload portability.

Now let's go through the first-principles solution step-by-step:

Usage Growth Projections:

Data Collection: Gather detailed metrics on current usage, including CPU, memory, storage, and network utilization.
Trend Analysis: Use historical data to project future growth based on business projections. This could be done using simple linear regression or more advanced techniques like time series analysis with machine learning models.
Scenario Planning: Develop multiple scenarios (best-case, worst-case) to cover different market conditions.

Competitive Quotes:

Vendor Selection: Choose at least two reputable competitors to request quotes from. This ensures you have a balanced comparison.
Request for Proposal (RFP): Clearly define your requirements and ask vendors to provide comprehensive pricing details, including any discounts or incentives they might offer.
Comprehensive Comparison: Evaluate the quotes based on total cost of ownership (TCO), service level agreements (SLAs), support options, and other relevant factors.

Workload Portability Signals:

Documentation: Document all your current infrastructure components, including virtual machines, databases, storage systems, and network configurations.
APIs and Integrations: Identify any third-party integrations or APIs used by your applications and document their interfaces.
Migration Strategy: Develop a detailed migration plan that includes timelines, resources required, and potential risks.

By following these steps, you can build a strong negotiation position without bluffing. This approach not only helps in securing better terms but also strengthens the relationship with your current provider by demonstrating transparency and professionalism.

**Comparison:**

```
Scenario	Best-Case	Worst-Case
Usage Growth	+50%	-20%
TCO Savings	$10,000/Mo	$3,000/Mo

Explanation: The table outlines two scenarios for usage growth and potential cost savings. Best-case assumes maximum growth with significant cost reductions; worst-case considers minimum growth or even slight increases in costs.
```

**Code:**

```python
# Example usage growth projection code using simple linear regression
from sklearn.linear_model import LinearRegression
import pandas as pd

def project_usage_growth(current_usage_data, future_months=12):
    df = pd.DataFrame(current_usage_data)
    X = df.index.values.reshape(-1, 1)  # reshape to a single feature column
    y = df['usage']

    model = LinearRegression()
    model.fit(X, y)

    x_future = np.arange(len(df), len(df) + future_months).reshape(-1, 1)
    projected_usage = model.predict(x_future)

    return pd.Series(projected_usage, index=df.index + future_months)

# Example workload portability signals
def document_workloads(current_infrastructure):
    # Document virtual machines, databases, storage systems, and network configurations
    workload_documentation = {
        'virtual_machines': {
            # VM names and configurations
        },
        'databases': {
            # Database types and configurations
        },
        'storage': {
            # Storage system details and access methods
        },
        'network': {
            # Network configurations, APIs used, etc.
        }
    }
    return workload_documentation

# Example competitive quote comparison
def compare_quotes(quotes):
    # Compare quotes based on TCO, SLAs, support options, etc.
    best_quote = min(quotes, key=lambda x: x['total_cost'])
    return best_quote
```

**Example input / output:**

```text
| Input (current_usage_data) | Output (projected_usage) |
|----------------------------|--------------------------|
| [{"month": 1, "usage": 50}, {"month": 2, "usage": 60}, {"month": 3, "usage": 70}] | [80, 90]                  |

Explanation: The input data shows usage growth over three months. Using linear regression, the model predicts a 40% increase in usage by month 6.
```

**Data:**

```json
{
  "current_usage_data": [
    {"month": 1, "usage": 50},
    {"month": 2, "usage": 60},
    {"month": 3, "usage": 70},
    {"month": 4, "usage": 80},
    {"month": 5, "usage": 90}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is Current Usage DataAccurate?
7. Perform Trend Analysis
8. Collect Detailed Metrics
9. Develop Growth Projections
10. Document Workloads
11. Have Competitive Quotes?
12. Compare Quotes
13. Request Quotes from Vendors
14. Prepare Migration Strategy
15. Document Workloads
16. Develop Migration Plan
17. Finalize Projections andStrategies
18. Finalize Documentation
19. Negotiate with Provider
20. End

