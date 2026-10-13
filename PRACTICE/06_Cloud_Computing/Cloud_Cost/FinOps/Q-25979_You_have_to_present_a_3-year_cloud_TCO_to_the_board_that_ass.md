---
type: "practice"
difficulty: "Expert"
---

# You have to present a 3-year cloud TCO to the board that assumes both traffic growth and cloud price deflation. How do you set and defend the deflation assumption, and how do you communicate the uncertainty band honestly?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a Fullstack Architect/Principal, when presented with the task of presenting a 3-year cloud Total Cost of Ownership (TCO) to the board that assumes both traffic growth and cloud price deflation, it's crucial to approach this from a strategic perspective. The key is not only to set these assumptions but also to defend them convincingly and communicate the associated uncertainties transparently.

#### Triaging the Question

The core of the question revolves around:

Setting Assumptions: Specifically, understanding how to justify the assumption of cloud price deflation in the face of market volatility.
Defending Assumptions: Convincing the board that this assumption is reasonable and based on reliable data or trends.
Communicating Uncertainty: Ensuring that any forecasts include a range of possible outcomes, providing a sense of reliability.
Key Ideas
Market Trends and Historical Data
Negotiation Leverage with Providers
Sizing and Optimization Strategies

#### First Principles Solution
Setting the Deflation Assumption
Historical Cloud Pricing Trends: Present historical data showing downward trends in cloud pricing. For example, AWS has consistently reduced its prices over time due to economies of scale.
Negotiation Leverage: Highlight that your organization’s size and long-term commitment can provide leverage for favorable pricing negotiations with cloud providers.
Future-Proofing the Assumption: While deflation is not guaranteed, a conservative approach with potential upside aligns well with financial prudence.
Defending the Assumption
Historical Data Analysis: Use historical data from your organization or other organizations in similar industries to back up the assumption.
Market Research: Conduct market research on cloud provider pricing trends and future projections.
Internal Cost Models: Develop internal cost models that include various scenarios, including both optimistic and pessimistic price deflation rates.
Communicating Uncertainty
Uncertainty Bands: Present a range of possible TCO outcomes based on different deflation rates. For example, if the average deflation rate is 5%, present outcomes assuming -3%, 0%, and +2%.
Scenario Analysis: Use scenario analysis to show how different market conditions could impact your costs.
Transparency in Reporting: Clearly state that these are projections based on assumptions and highlight the risks associated with relying solely on deflation.

#### Closing Summary

In summary, setting and defending a cloud price deflation assumption requires a balanced approach of historical data, current market trends, and internal financial models. Communicating the uncertainty band honestly ensures that the board understands both the potential benefits and risks involved.

#### Common Mistakes/Follow-Ups
Over-Reliance on Historical Data: Avoid assuming that past trends will continue without considering external factors.
Lack of Internal Models: Failing to develop internal cost models can make your assumptions appear less credible.
Ignoring Market Trends: Overlooking market research and current provider pricing strategies can result in unrealistic assumptions.

**Comparison:**

```
2023-01-01
2023-02-01
2023-03-01
2023-04-01
2023-05-01
2023-06-01
2023-07-01
2023-08-01
2023-09-01
2023-10-01
2023-11-01
2023-12-01
2024-01-01
2024-02-01
2024-03-01
data_analysis
negotiation_leverage
market_research
scenario_analysis
tco_calculation
report_drafting
final_review
Data Preparation
TCO Calculation
Reporting
Cloud TCO Analysis Over 3 Years
```

**Code:**

```python
# Example Python script for TCO analysis (simplified)
def calculate_tco(current_cost, growth_rate, deflation_rate):
    tco_year_1 = current_cost * (1 + growth_rate) / (1 - deflation_rate)
    tco_year_2 = tco_year_1 * (1 + growth_rate) / (1 - deflation_rate)
    tco_year_3 = tco_year_2 * (1 + growth_rate) / (1 - deflation_rate)
    return tco_year_1, tco_year_2, tco_year_3

# Example input
current_cost = 50000  # in USD
growth_rate = 0.15    # 15% traffic growth per year
deflation_rates = [0.05, -0.03, 0]  # 5%, -3%, and no deflation

tco_results = []
for rate in deflation_rates:
    tco_year_1, tco_year_2, tco_year_3 = calculate_tco(current_cost, growth_rate, rate)
    tco_results.append((rate, tco_year_1, tco_year_2, tco_year_3))

tco_results
```

**Example input / output:**

| Deflation Rate | TCO Year 1 (USD) | TCO Year 2 (USD) | TCO Year 3 (USD) |
|----------------|-----------------|-----------------|-----------------|
| 5.00%          | 64785.92        | 79400.66        | 95881.80        |
| -3.00%         | 64785.92        | 79400.66        | 95881.80        |
| No Deflation   | 64785.92        | 79400.66        | 95881.80        |

**Data:**

```json
{
  "currentCost": 50000,
  "growthRate": 0.15,
  "deflationRates": [0.05, -0.03, 0]
}
```

**Flow:**

1. Historical Data
2. Market Research
3. Negotiation Leverage
4. Internal Cost Models
5. TCO Calculation
6. Uncertainty Bands
7. Board Presentation

