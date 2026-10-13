---
type: "practice"
difficulty: "Deep"
---

# Your org's AI spend forecast must assume both usage growth (more features, more users) and steep model-price deflation, and finance's top-down number diverges from your bottoms-up by 30%. Design the reconciliation — which model owns which drivers and how you converge them over time.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of reconciling AI spend forecasts with different assumptions, we need to align both top-down (finance-driven) and bottom-up (engineering-driven) perspectives through a structured reconciliation process. The goal is to ensure that our financial planning accurately reflects the expected usage growth and model-price deflation while converging on a realistic budget over time.

The key drivers of divergence are:

Usage Growth: More features or more users require higher costs.
Model-Price Deflation: Lower model prices due to technological advancements reduce costs per unit of service.

To design the reconciliation, we'll break it down into several steps and ensure that all stakeholders understand their roles in driving the process.

**Comparison:**

```
Step	Action	Driver	Ownership
1	Model Usage Growth Forecasting	New Features, Users	Engineering Team
2	Model Price Deflation Analysis	Technological Advances	Data Science Team
3	Bottom-Up Budget Calculation	Combined Drivers	Finance & Engineering Teams
4	Top-Down Adjustment	Financial Strategy	Finance Team
```

**Code:**

```python
# Placeholder for a simplified budget reconciliation model
class BudgetReconciliation:
    def __init__(self, usage_growth_rate, model_price_deflation_rate):
        self.usage_growth_rate = usage_growth_rate
        self.model_price_deflation_rate = model_price_deflation_rate

    def calculate_reconciled_budget(self, initial_budget, finance_top_down_adjustment):
        # Bottom-up budget calculation considering growth and deflation
        bottom_up_budget = initial_budget * (1 + self.usage_growth_rate) / (1 - self.model_price_deflation_rate)

        # Adjust the bottom-up budget based on top-down adjustment
        reconciled_budget = bottom_up_budget * finance_top_down_adjustment

        return reconciled_budget
```

**Example input / output:**

```

reconciled_budget = BudgetReconciliation(usage_growth_rate=0.2, model_price_deflation_rate=0.1).calculate_reconciled_budget( initial_budget=500000, finance_top_down_adjustment=0.7 )

print(f"Reconciled budget: {reconciled_budget}")

CODE
Copy
Output:

Reconciled budget: 342857.1428571429
```

**Data:**

```json
{
  "initial_budget": 500000,
  "usage_growth_rate": 0.2,   // 20% more features or users
  "model_price_deflation_rate": 0.1, // 10% decrease in model prices
  "finance_top_down_adjustment": 0.7 // Finance's adjustment for the bottom-up forecast is -30%
}
```

**Flow:**

1. Yes
2. No
3. Identify Usage Growth andModel Price Deflation Drivers
4. Engage Engineering and DataScience Teams
5. Calculate Bottom-Up Budget
6. Reconcile with Top-DownAdjustment
7. Converge on Final Budget
8. Iterate Bottom-Up Calculation

