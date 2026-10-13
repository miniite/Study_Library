---
type: "practice"
difficulty: "Deep"
---

# Your dbt CI cost exploded because every PR runs a full build against a cloned prod warehouse; what levers (slim CI, sampling, dedicated small warehouse, auto-suspend) would you pull and how do you measure impact?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CI Cost Optimization](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of exploding CI costs in dbt when every Pull Request (PR) runs a full build against a cloned production warehouse, we need to consider several levers that can be pulled to optimize the CI process. These include slimming down the CI pipeline, using sampling techniques, employing dedicated small warehouses, and auto-suspending resources during idle periods. Each of these strategies has its own set of trade-offs and impacts on performance and cost.

First, let’s define what we are trying to achieve: reducing the overhead associated with running full builds in CI while ensuring that the quality and integrity of our data models remain intact. We will analyze each lever one by one, starting from the basics and moving towards more advanced techniques.

**Comparison:**

```
CI Strategy	Average Time (sec)	Cost per Month
Full Build	300	$12,000
Slim CI	120	$4,800
Sampling	60	$2,400
```

**Code:**

```python
# Example Python script to simulate dbt CI runs for different strategies
def simulate_ci_run(strategy):
    if strategy == 'full_build':
        return 300  # Full build takes approximately 5 minutes (300 seconds)
    elif strategy == 'slim_ci':
        return 120  # Slim CI run takes 2 minutes (120 seconds)
    elif strategy == 'sampling':
        return 60   # Sampling reduces to 1 minute (60 seconds)
    else:
        raise ValueError("Invalid strategy")

# Simulate different scenarios
full_build_time = simulate_ci_run('full_build')
slim_ci_time = simulate_ci_run('slim_ci')
sampling_time = simulate_ci_run('sampling')

print(f"Full build: {full_build_time} seconds")
print(f"Slim CI run: {slim_ci_time} seconds")
print(f"Sampling: {sampling_time} seconds")
```

**Example input / output:**

```text
# Input for different CI strategies and their impact on cost
ci_strategies = ['full_build', 'slim_ci', 'sampling']
costs = []

for strategy in ci_strategies:
    cost = simulate_ci_run(strategy) / 60 * (8 * 24)  # Assume 8 hours of work per day, 24 days a month
    costs.append(cost)

# Output the cost comparison
print(f"Cost comparison: {costs}")
```

**Data:**

```json
{
  "warehouse_size": {
    "prod": "large",
    "dev": "small"
  },
  "cost_per_hour": {
    "large_warehouse": 100,
    "small_warehouse": 30
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. CI Strategy Selection
6. Full Build?
7. Run Full Build
8. Choose Between Slim CI andSampling
9. Slim CI?
10. Run Slim CI
11. Run Sampling
12. Calculate Cost
13. Monitor Performance & Cost

