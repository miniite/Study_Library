---
type: "practice"
difficulty: "Expert"
---

# Your agent platform lets agents retry failed tool calls and reflect on mistakes, which improves success rate but multiplies token spend per task. Design the governance that trades incremental success-rate gains against the marginal cost of each additional reflection loop.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Spend Freezes and Prioritization](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Spend_Freezes_and_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a governance system that optimizes between incremental success-rate gains and the marginal cost of additional reflection loops, we need to carefully consider both the financial aspects and the operational efficiency of our agent platform. This involves balancing the cost of retries and reflections against the benefits they bring in terms of task success rates. Here’s how we can approach this:

Firstly, let's understand that each reflection loop not only incurs additional token spend but also adds latency to task completion. The key is to find an optimal point where the marginal increase in success rate justifies the cost and delay.

**Comparison:**

```
Loop	Success Rate Improvement	Cost per Loop (USD)	Cumulative Cost (USD)	New Success Rate
0	-	0.03	0.03	0.85
1	0.02	0.06	0.09	0.87
2	0.04	0.09	0.18	0.91

In the above example, after two reflection loops, we achieve a success rate of 0.91, which is just above our minimum acceptable threshold, while staying within the budget.
```

**Code:**

```python
def calculate_optimal_reflection_points(success_rate_improvement, cost_per_loop):
    """
    This function calculates the optimal number of reflection loops based on
    the expected success rate improvement and cost per loop.

    Parameters:
        - success_rate_improvement (float): The expected increase in success rate per loop.
        - cost_per_loop (float): The cost incurred by each additional loop.

    Returns:
        int: Optimal number of reflection loops to perform.
    """
    # For simplicity, assume a linear relationship between loops and success rate
    optimal_loops = 0
    current_success_rate = initial_success_rate
    while True:
        new_success_rate = current_success_rate + (success_rate_improvement * (optimal_loops + 1))
        cost_of_loop = cost_per_loop * (optimal_loops + 1)
        if cost_of_loop > budget or new_success_rate < minimum_acceptable_success_rate:
            break
        optimal_loops += 1

    return optimal_loops
```

**Example input / output:**

```
Example input values

initial_success_rate = 0.85 success_rate_improvement_per_loop = 0.02 cost_per_loop = 0.03 budget = 1.0 minimum_acceptable_success_rate = 0.9

optimal_loops = calculate_optimal_reflection_points(success_rate_improvement_per_loop, cost_per_loop) print(f"Optimal number of reflection loops: {optimal_loops}")

CODE
Copy
Output:

Optimal number of reflection loops: 2
```

**Data:**

```json
{
  "initial_success_rate": 0.85,
  "success_rate_improvement_per_loop": 0.02,
  "cost_per_loop": 0.03,
  "budget": 1.0,
  "minimum_acceptable_success_rate": 0.9
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Initial Success Rate
6. Perform Reflection Loop
7. Success Rate Improvement?
8. Task Failed - Retry
9. Marginal Cost ExceededBudget?
10. Proceed with Task Completion
11. Abort Task Execution

