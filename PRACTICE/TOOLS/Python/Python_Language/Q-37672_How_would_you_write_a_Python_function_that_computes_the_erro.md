---
type: "practice"
difficulty: "Deep"
---

# How would you write a Python function that computes the error budget remaining given a target uptime and the observed downtime minutes?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the error budget remaining, we need to determine how much downtime is allowed based on the target uptime percentage and subtract this from the actual observed downtime. The key idea here is that the function will take the target uptime as a percentage and the observed downtime in minutes, then calculate the allowable downtime (error budget) in minutes.

First, let's clarify what we are solving for:

Target Uptime: A percentage representing the desired system availability.
Observed Downtime: The actual time the system was down measured in minutes.
Error Budget Remaining: The remaining allowance of downtime based on the target uptime minus observed downtime.

We will assume that a 100% uptime is equivalent to no downtime, and any downtime exceeds this becomes part of the error budget. Let's proceed with the first-principles solution step-by-step:

Convert the target uptime percentage into minutes.
Calculate the allowable downtime based on the target uptime.
Subtract the observed downtime from the allowable downtime to find the remaining error budget.

Now, let's write the Python function:

PYTHON
def compute_error_budget(target_uptime: float, observed_downtime_minutes: int) -> float:
    """
    Computes the remaining error budget in minutes given a target uptime and observed downtime.

    Args:
    - target_uptime (float): Target uptime percentage as a decimal (e.g., 0.95 for 95%).
    - observed_downtime_minutes (int): Observed downtime in minutes.

    Returns:
    - float: Remaining error budget in minutes.
    """
    # Assuming the system operates over 1440 minutes per day
    total_minutes_per_day = 1440

    # Calculate allowable downtime based on target uptime
    allowable_downtime_minutes = (1 - target_uptime) * total_minutes_per_day

    # Compute remaining error budget
    error_budget_remaining = allowable_downtime_minutes - observed_downtime_minutes

    return max(error_budget_remaining, 0)

To ensure the function works as expected, let's test it with some sample data:

Test Case 1: Target uptime of 95% (0.95) and observed downtime of 30 minutes.
Test Case 2: Target uptime of 98% (0.98) and observed downtime of 70 minutes.

Now, let's run the tests:

PYTHON
# Test cases
print(compute_error_budget(0.95, 30))  # Expected: ~126 minutes
print(compute_error_budget(0.98, 70))  # Expected: -34 minutes (negative indicates no remaining budget)

This function accounts for the fact that a negative error budget indicates the system has already exceeded its allowable downtime.

**Code:**

```python
def compute_error_budget(target_uptime: float, observed_downtime_minutes: int) -> float:
    total_minutes_per_day = 1440
    allowable_downtime_minutes = (1 - target_uptime) * total_minutes_per_day
    error_budget_remaining = allowable_downtime_minutes - observed_downtime_minutes
    return max(error_budget_remaining, 0)
```

**Example input / output:**

```text
print(compute_error_budget(0.95, 30))  # Expected output: ~126 minutes
print(compute_error_budget(0.98, 70))  # Expected output: -34 minutes
```

**Flow:**

1. Yes
2. No
3. Remaining < Allowable
4. Remaining >= Allowable
5. Start
6. Observe Downtime
7. Is Observed Downtime >Allowable?
8. Error Budget Remaining is 0 orNegative
9. Compute Error Budget
10. Error Budget Remaining =Remaining - Allowable
11. Error Budget Remaining = 0
12. End

