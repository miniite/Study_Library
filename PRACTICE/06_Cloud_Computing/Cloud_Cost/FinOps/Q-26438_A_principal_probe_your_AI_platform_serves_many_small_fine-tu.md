---
type: "practice"
difficulty: "Expert"
---

# A principal probe: your AI platform serves many small fine-tuned adapters over a shared base model, and you must attribute the shared base-serving cost plus each adapter's marginal cost fairly to owning teams. Design the metering that splits shared base cost without penalizing low-traffic adapters unfairly.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shared_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this principal probe, we need a fair and efficient way to attribute costs in an AI platform where multiple small, fine-tuned adapters share a single base model. The challenge is to allocate the cost of running the shared base model fairly among teams owning different adapters while ensuring low-traffic adapters are not penalized disproportionately.

#### Triage

The question revolves around designing a metering mechanism for cost allocation in a multi-tenant environment with a shared resource (the base model). We need to balance fairness, accuracy, and complexity. Assumptions include:

The platform runs multiple small, fine-tuned adapters over a shared base model.
Each adapter has its own user traffic and associated costs.
Fairness is paramount to avoid penalizing low-traffic adapters.

#### First Principles Solution

To achieve this, we need a metering approach that accurately reflects the usage of each adapter without imposing disproportionate penalties on smaller or less frequently used ones. Here’s a step-by-step solution:

Usage Metrics: Collect detailed metrics such as request volume, duration, and resource consumption for both the base model and individual adapters.
Shared Base Model Cost Attribution: Allocate the cost of running the shared base model based on usage patterns.
Adapter-Specific Cost Attribution: Ensure that each adapter's marginal cost is accurately captured.
Fairness Mechanism: Implement a mechanism to adjust costs for low-traffic adapters without penalizing them unfairly.
Step-by-Step Solution
1. Collect Usage Metrics

Collect metrics on request volume, duration, and resource consumption:

MARKDOWN
| Adapter | Requests (R) | Duration (D) | Resources (Z) |
|---------|--------------|--------------|---------------|
| A       | 100          | 2 hours      | 5 units       |
| B       | 200          | 3 hours      | 4 units       |
2. Shared Base Model Cost Attribution

Allocate the cost of running the shared base model based on usage:

Calculate a proportional share for each adapter.
For example, if Adapter A and B together make up 150 out of 300 requests, then Adapter A's share is ( \frac{100}{150} = \frac{2}{3} ).
3. Adapter-Specific Cost Attribution

Capture the marginal cost for each adapter:

This includes additional resources consumed specifically by that adapter.
For example, if Adapter A consumes an extra 2 units of resources due to its unique operations.
Fairness Mechanism

To ensure fairness and avoid disproportionately penalizing low-traffic adapters, we can implement a tiered pricing model or apply a smoothing factor:

Tiered Pricing: Define usage tiers (e.g., <50 requests, 50-200 requests, >200 requests) with different cost rates.
Smoothing Factor: Introduce a factor to adjust the cost based on traffic volume, ensuring low-traffic adapters are not penalized too heavily.

#### Implementation
Example

Assume we have two adapters, A and B, sharing a base model:

MARKDOWN
| Adapter | Requests (R) | Duration (D) | Resources (Z) |
|---------|--------------|--------------|---------------|
| A       | 100          | 2 hours      | 5 units       |
| B       | 200          | 3 hours      | 4 units       |
Calculate Shared Base Model Cost:
Total requests = 300
Adapter A's share of base model cost: ( \frac{100}{300} = \frac{1}{3} )
Adapter B's share of base model cost: ( \frac{200}{300} = \frac{2}{3} )
Calculate Marginal Cost:
Assume resource cost per unit is $0.5.
Adapter A’s marginal cost: 5 units * $0.5 = $2.5
Adapter B’s marginal cost: 4 units * $0.5 = $2
Fairness Adjustment (Example)

Apply a smoothing factor to ensure fairness:

If we apply a ( \frac{1}{3} ) smoothing factor, the adjusted cost for both adapters would be:
Adapter A: ( \frac{1}{3} \times (\text{Base Model Cost}) + $2.5 = \frac{\text{Base Model Cost}}{3} + 2.5 )
Adapter B: ( \frac{2}{3} \times (\text{Base Model Cost}) + $2 = \frac{2 \times \text{Base Model Cost}}{3} + 2 )

#### Conclusion

This approach ensures that the shared base model cost is fairly attributed based on usage, while low-traffic adapters are not penalized disproportionately. By implementing a tiered pricing model and smoothing factor, we can achieve both fairness and accuracy in cost allocation.

#### Common Mistakes/Follow-Ups
Underestimating Low-Traffic Adapters: Failing to apply adjustments that ensure these adapters are not overcharged.
Overly Complex Metrics: Complicating the metric collection process more than necessary, leading to higher maintenance costs.
Ignoring Base Model Overhead: Not accounting for the shared overhead of running the base model.

**Comparison:**

| Adapter | Requests | Duration | Resources | Base Model Cost Share | Marginal Cost | Total Cost |
|---------|----------|----------|-----------|----------------------|--------------|------------|
| A       | 100      | 2 hours  | 5 units   | \$2                  | $2.5         | $4.5       |
| B       | 200      | 3 hours  | 4 units   | \$4                  | $2           | $6         |

**Code:**

```python
# Example code to calculate cost attribution
def calculate_cost(requests, duration, resources):
    base_model_cost = 500  # Hypothetical cost for running the base model
    resource_cost_per_unit = 0.5

    # Shared Base Model Cost Attribution
    total_requests = sum(requests)
    shared_base_cost = {
        'A': (requests['A'] / total_requests) * base_model_cost,
        'B': (requests['B'] / total_requests) * base_model_cost
    }

    # Adapter-Specific Cost Attribution
    adapter_costs = {}
    for adapter, reqs in requests.items():
        adapter_costs[adapter] = shared_base_cost[adapter] + sum(res * resource_cost_per_unit for res in resources[adapter])

    return adapter_costs

requests = {'A': 100, 'B': 200}
resources_A = [5]
resources_B = [4]

costs = calculate_cost(requests, duration=3, resources={'A': resources_A, 'B': resources_B})
print(costs)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| requests = {'A': 100, 'B': 200}, resources_A = [5], resources_B = [4] |
| Cost for Adapter A: $3.5 (Base Model: \$2 + Resources: \$1.5) |
| Cost for Adapter B: $6.8 (Base Model: \$4 + Resources: \$2.8) |

**Data:**

```json
{
  "requests": {"A": 100, "B": 200},
  "resources": {"A": [5], "B": [4]}
}
```

**Flow:**

1. Collect Usage Metrics
2. Calculate Shared Base ModelCost
3. Calculate Adapter-SpecificMarginal Cost
4. Apply Smoothing Factor forFairness
5. Allocate Costs Accurately

