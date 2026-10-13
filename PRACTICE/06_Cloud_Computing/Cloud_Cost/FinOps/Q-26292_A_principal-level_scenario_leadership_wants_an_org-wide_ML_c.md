---
type: "practice"
difficulty: "Expert"
---

# A principal-level scenario: leadership wants an org-wide 'ML cost efficiency ratio' for the board that can't be gamed by shifting cost between training (capex-like) and inference (opex-like) buckets. Design the composite metric and explain what it deliberately hides.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Healthy vs Wasteful Cost Growth](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Healthy_vs_Wasteful_Cost_Growth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design an 'ML cost efficiency ratio' that measures the overall cost effectiveness of our machine learning operations across both training (capex-like) and inference (opex-like) stages. The key challenge is to ensure the metric cannot be manipulated by shifting costs between these two buckets. To achieve this, we'll focus on creating a composite metric that balances these aspects while clearly defining what it deliberately hides.

To start, let's break down the problem into its core components:

#### Triaging the Question

The primary goal is to create an ML cost efficiency ratio that reflects overall cost effectiveness without allowing manipulation through shifting costs. This means we need to:

Identify and separate training and inference costs.
Define a metric that captures both stages' efficiencies.
Ensure the metric cannot be gamed by artificial shifts in cost allocation.

#### Assumptions
We have historical data on ML operations, including training and inference costs.
The board is interested in a high-level overview rather than detailed breakdowns.

#### First Principles Solution

Define Costs:

Training costs (capex-like) will include hardware, software, cloud resources, and license fees.
Inference costs (opex-like) will include cloud service usage, licenses, and maintenance.

Choose a Composite Metric: To create an ML cost efficiency ratio, we can use a combination of metrics such as:

Cost per Model: This measures the total cost incurred to train each model.
Inference Efficiency: This measures how much inference is achieved per unit cost.

Formula for ML Cost Efficiency Ratio: [ \text{ML Cost Efficiency Ratio} = \frac{\text{Total Inference Efficiency}}{\text{Training Costs}} ]

Deliberately Hidden Information:

This ratio deliberately hides the actual distribution of costs between training and inference.
It also abstracts away from specific cost items, focusing on overall efficiency.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using a composite metric like the ML Cost Efficiency Ratio, we ensure that the metric cannot be easily gamed by shifting costs. The ratio provides a high-level overview of the organization's ML operations' overall effectiveness without revealing detailed cost breakdowns. However, it does not provide insights into specific cost drivers within training or inference stages.

#### Common Mistakes

Overlooking the need to balance between capex and opex aspects.
Failing to choose appropriate metrics that can be easily gamed.
Not considering the impact on board-level decisions when providing a high-level overview.

#### Follow-Ups

How do we handle scenarios where training costs are significantly higher than inference costs?
Are there better ways to visualize cost distributions while maintaining the desired level of abstraction?

**Comparison:**

```
Metric	Description
Training Costs	Capex-like costs associated with model training
Inference Efficiency	Opex-like costs and performance of inference operations
ML Cost Efficiency Ratio	Overall cost effectiveness balancing both stages
```

**Code:**

```python
# Example Python code to calculate ML Cost Efficiency Ratio

def calculate_cost_efficiency_ratio(training_costs, total_inference_efficiency):
    return total_inference_efficiency / training_costs

# Sample Data
training_costs = 100000  # in USD
total_inference_efficiency = 50000  # inference units per dollar spent on training

ml_cost_efficiency_ratio = calculate_cost_efficiency_ratio(training_costs, total_inference_efficiency)
print(f"ML Cost Efficiency Ratio: {ml_cost_efficiency_ratio}")
```

**Example input / output:**

```text
Input:
- Training Costs = $100,000
- Total Inference Efficiency = 50,000 inference units per dollar

Output:
- ML Cost Efficiency Ratio = 0.5 (50% efficiency)
```

**Data:**

```json
{
  "training_costs": 100000,
  "total_inference_efficiency": 50000
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Is there a need for a high-leveloverview?
6. Define ML Cost Efficiency Ratio
7. Calculate Total InferenceEfficiency
8. Divide by Training Costs to getML Cost Efficiency Ratio
9. Present the ratio, hidingdetailed cost breakdowns
10. End
11. Are there specific insightsneeded?
12. Adjust metric definition

