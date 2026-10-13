---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks: you claim self-hosting an open model is cheaper than the API at your volume, but the API price just dropped below your fully-loaded self-hosting cost. How did you account for the provider's ongoing price cuts in the original build-vs-buy decision, and would you now shut the self-hosted fleet?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this bar-raiser question, we need to revisit our initial cost analysis of both self-hosting an open model versus using a managed API service. Our original decision was based on a detailed cost breakdown that included not just direct costs but also operational overheads and long-term strategic considerations. Now, with the API price drop, we must reassess the cost comparison.

Firstly, let's break down our initial analysis:

Direct Costs:
Hosting self-hosted infrastructure (hardware, software licenses).
Operating expenses (OPEX) such as maintenance, monitoring, and security.
Opportunity Cost:
Time spent on operations versus leveraging a managed service to focus on core business objectives.
Long-Term Strategy:
Scalability requirements and potential future cost escalations in the cloud environment.

Now, let's dive into the details of the original analysis:

Initial Analysis

Self-hosted Open Model:

Infrastructure cost: $100K/year (for a mid-range setup).
OPEX: $50K/year for maintenance and support.
One-time implementation cost: $20K.
Total annual cost: $170K.

Managed API Service:

Annual subscription price: $300K/year before the price drop.

We concluded that self-hosting was cheaper at our volume, primarily due to the substantial one-time setup costs and lower OPEX compared to the managed service.

Current Analysis

The bar-raiser points out that the API price has dropped below our fully-loaded self-hosted cost. This changes the equation significantly. Let's re-run the numbers:

New Managed API Service Cost: $200K/year.
Self-hosted Open Model:
Infrastructure cost: $100K/year.
OPEX: $50K/year.
One-time implementation cost: $20K.
Total annual cost: $170K.

At first glance, the self-hosted model still looks cheaper. However, we need to consider several additional factors:

Revisiting Key Assumptions
Price Sensitivity: The API price could drop further in the future, but this is a valid concern.
Operational Flexibility:
Scalability: Managed services often offer more straightforward scaling options. If our volume grows significantly, the managed service might become even cheaper on a per-unit basis due to dynamic pricing and shared resources.
Risk of Lock-In: Self-hosting gives us more control over our infrastructure but also ties us into ongoing maintenance and operational costs.
Opportunity Cost:
Continuing to self-manage operations diverts resources that could be better spent on core business initiatives.
Reevaluation

Given the API price drop, we should now consider shutting down the self-hosted fleet if it no longer provides a cost advantage. The decision should weigh the following:

Risk Management: If we rely heavily on the open model for critical operations, there is risk associated with potential future price increases or service disruptions.
Operational Efficiency: Leveraging managed services can free up our team to focus on more strategic initiatives and reduce operational overhead.

#### Conclusion

We should indeed re-evaluate and potentially shut down the self-hosted fleet. While it initially seemed cheaper, the new API cost makes it a more attractive option. This decision aligns with the principle of continuously optimizing costs based on current market conditions and long-term strategic objectives.

#### Common mistakes
include:

Overlooking the one-time implementation cost in the original analysis.
Ignoring potential future price changes or scaling needs.
Underestimating the value of operational flexibility provided by managed services.

**Comparison:**

| Cost Component            | Self-Hosting (Annual)   | Managed API (Annual)    |
|----------------------------|------------------------|-------------------------|
| Infrastructure             | $100K                  | -                       |
| OPEX (Maintenance, Security)| $50K                   | -                       |
| Implementation             | $20K                   | -                       |
| Total                      | $170K                  | $200K                   |

**Code:**

```python
# Example code snippet to illustrate cost comparison

def compare_cost(self_hosted_cost, api_cost):
    if self_hosted_cost > api_cost:
        return "Self-hosted is cheaper"
    else:
        return "API is cheaper"

self_hosted_cost = 170_000  # Total annual cost for self-hosting
api_cost = 200_000          # New API subscription price

result = compare_cost(self_hosted_cost, api_cost)
print(result)  # Output: "API is cheaper"
```

**Example input / output:**

```text
| Input Parameters            | Value                    |
|-----------------------------|--------------------------|
| selfHostedCost              | $170,000                 |
| apiCurrentSubscription      | $200,000                 |

Output: API is cheaper.
```

**Data:**

```json
{
  "selfHostedCost": {
    "infrastructure": 100000,
    "OPEX": 50000,
    "implementation": 20000,
    "totalAnnual": 170000
  },
  "apiCost": {
    "subscription": 300000,  # Before price drop
    "currentSubscription": 200000  # After price drop
  }
}
```

**Flow:**

1. Initial Analysis
2. Self-hosted is cheaper
3. New API price drop to $200K
4. Re-evaluation needed
5. API now cheaper
6. Shut down self-hosted fleet

