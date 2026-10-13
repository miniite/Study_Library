---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set up the incentive structure so that the team that owns a shared platform is motivated to make it cheaper for consumers, not to maximize its own chargeback revenue. How do you avoid the internal-monopoly rent-seeking failure mode?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Funding Models and Incentives](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Funding_Models_and_Incentives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of incentivizing a team that owns a shared platform to focus on making it cheaper for consumers, rather than maximizing its own chargeback revenue, we need to design an incentive structure that aligns their goals with overall cost optimization and value delivery. The key is to create a system where minimizing costs benefits both the owning team and the broader organization.

The primary challenge here is avoiding what economists call "internal monopoly rent-seeking." This occurs when a department or team hoards resources or capabilities, driving up costs for other teams while maximizing its own revenue. To prevent this, we must ensure that cost-saving efforts are recognized and rewarded, fostering collaboration rather than competition.

**Comparison:**

```
Platform	Current Cost (USD)	New Cost (USD)	Cost Savings Percentage
Shared Storage	5000	4500	10.0%
Database Management	7500	6800	9.33%
```

**Code:**

```text
# Example Code to Illustrate the Concept
class CostSavingsIncentive:
    def __init__(self, platform_cost_savings_percentage):
        self.savings_threshold = 0.1  # 10% cost savings threshold
        self.platform_cost_savings_percentage = platform_cost_savings_percentage

    def calculate_incentive(self, current_platform_cost, new_platform_cost):
        if (current_platform_cost - new_platform_cost) / current_platform_cost >= self.savings_threshold:
            return round((self.platform_cost_savings_percentage * ((current_platform_cost - new_platform_cost) / current_platform_cost)), 2)
        else:
            return 0
```

**Example input / output:**

```text
# Example Input and Output for Cost Savings Incentive Calculation
cost_savings_incentive = CostSavingsIncentive(platform_cost_savings_percentage=1.0)
print(cost_savings_incentive.calculate_incentive(5000, 4500))  # Expected output: 1.00

cost_savings_incentive = CostSavingsIncentive(platform_cost_savings_percentage=0.5)
print(cost_savings_incentive.calculate_incentive(7500, 6800))  # Expected output: 0.49
```

**Data:**

```yaml
# Example YAML for Cost Savings Incentives
platform_costs:
  - platform: shared_storage
    cost_before: 5000
    cost_after: 4500
  - platform: database_management
    cost_before: 7500
    cost_after: 6800
```

**Flow:**

1. Yes
2. No
3. Identify Cost SavingsOpportunities
4. Savings Threshold Met?
5. Incentive Payment
6. Monitor and Adjust Costs
7. Reward Team for Savings

