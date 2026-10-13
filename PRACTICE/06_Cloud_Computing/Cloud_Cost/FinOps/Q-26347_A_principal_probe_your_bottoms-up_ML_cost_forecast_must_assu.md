---
type: "practice"
difficulty: "Expert"
---

# A principal probe: your bottoms-up ML cost forecast must assume both training-demand growth (more models, bigger runs) and steep GPU price deflation plus new accelerator generations. How do you set and defend the deflation assumption in a 3-year ML-TCO model, and how do you communicate the uncertainty band honestly to the board?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the principal probe on setting and defending the deflation assumption in an ML TCO model over a three-year period, we need to start by understanding that cost governance involves both strategic planning and transparent communication. The key is to build a robust model that not only accounts for growth but also acknowledges the dynamic nature of hardware costs.

TL;DR: We must set a conservative deflation assumption based on historical data trends, industry forecasts, and vendor contracts. To defend this, we need to validate it with multiple scenarios and communicate the uncertainty bands clearly to stakeholders, ensuring transparency and trust.

#### Triaging the Question

The question is asking how to establish a deflation assumption for GPU costs in an ML TCO model over three years. The interviewer wants to understand our approach to handling assumptions, validating them, and communicating uncertainties effectively. We need to consider historical trends, vendor agreements, and market projections while also acknowledging the inherent uncertainty.

#### First-Principles Solution
Historical Data Analysis: Start by analyzing GPU price history over the past 5-10 years. This will help identify any patterns of deflation or inflation.
Industry Forecasts: Utilize industry reports and analyst projections to gauge future trends.
Vendor Contracts: Review current and upcoming contract terms with hardware vendors for guaranteed pricing or discounts.
Scenario Analysis: Develop multiple scenarios (optimistic, pessimistic, base-case) to model various deflation rates.
Communication Plan: Present the assumption along with its basis and potential impact on TCO.
Setting the Deflation Assumption

Let's assume we've gathered data showing an average annual GPU price reduction of 10% over the past five years, supported by vendor contracts indicating a 9% deflation rate for the next three years. Based on this, we could set our base-case assumption at a 10% yearly deflation.

Defending the Assumption

To defend this assumption:

Historical Trends: Present a chart showing GPU prices over time to illustrate the trend.
Vendor Agreements: Highlight specific contract terms that guarantee certain pricing levels.
Market Forecasts: Cite industry reports like those from Gartner or IDG, which predict similar deflation rates.
Communicating Uncertainty

We need to communicate the uncertainty band honestly:

Scenario Analysis Results: Present a table summarizing different scenarios and their impacts on TCO.
Sensitivity Analysis: Perform sensitivity analysis to show how changes in key assumptions (e.g., price deflation rate) affect overall costs.
Qualitative Factors: Discuss qualitative factors that could impact GPU prices, such as supply chain disruptions or technological advancements.

#### Closing Summary

In summary, setting and defending the deflation assumption requires a combination of historical data analysis, industry forecasts, vendor agreements, and scenario modeling. Communicating this with stakeholders involves presenting multiple scenarios and their implications, ensuring transparency and trust.

#### Common Mistakes/Follow-Ups
Over-relying on Historical Data: Failing to consider new market dynamics.
Ignoring Vendor Agreements: Not leveraging contract terms for more accurate predictions.
Lack of Scenario Analysis: Missing out on the full range of possible outcomes.

For follow-ups, the interviewer might probe deeper into specific assumptions or ask about handling other cost components (e.g., cloud services).

**Comparison:**

| Scenario | Deflation Rate (%) | TCO Impact (USD) |
|----------|--------------------|-----------------|
| Base Case | 10                 | $X              |
| Optimistic | 12                 | $Y              |
| Pessimistic | 8                  | $Z              |

**Code:**

```python
# Example Code to Analyze GPU Price Trends
import pandas as pd

gpu_prices = {
    'Year': [2018, 2019, 2020, 2021, 2022],
    'Price': [3500, 2800, 2400, 2000, 1700]
}

df_prices = pd.DataFrame(gpu_prices)

# Calculate Annual Deflation Rate
def calculate_deflation_rate(prices_df):
    prices_df['Deflation_Rate'] = (prices_df['Price'].diff() / prices_df['Price'].shift(1)) * -100
    return prices_df

df_prices = calculate_deflation_rate(df_prices)
print(df_prices[['Year', 'Deflation_Rate']].to_string(index=False))
```

**Example input / output:**

| Year | Price | Deflation_Rate |
|------|-------|---------------|
| 2018 | 3500  | -20%           |
| 2019 | 2800  | -20.0%         |
| 2020 | 2400  | -17.65%        |
| 2021 | 2000  | -16.67%        |
| 2022 | 1700  | -15%           |

**Data:**

```json
{
  "gpu_prices": {
    "2018": 3500,
    "2019": 2800,
    "2020": 2400,
    "2021": 2000,
    "2022": 1700
  }
}
```

**Flow:**

1. Historical Data Analysis
2. Identify Deflation Trends
3. Vendor Agreements Review
4. Industry Forecasts Integration
5. Scenario Development
6. Communication Plan Design
7. Stakeholder Briefing

