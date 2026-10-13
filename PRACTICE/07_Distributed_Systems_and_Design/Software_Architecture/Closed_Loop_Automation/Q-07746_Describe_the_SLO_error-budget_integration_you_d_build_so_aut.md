---
type: "practice"
difficulty: "Expert"
---

# Describe the SLO/error-budget integration you'd build so automation acts more aggressively when budget is healthy and conservatively when it's nearly spent.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Business Aware Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Business_Aware_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the SLO (Service Level Objective) or error-budget integration in a closed-loop automation architecture, we need to ensure that our automated systems react appropriately based on the health of the budget. When the error budget is healthy, automation should act more aggressively by making bolder changes and optimizations. Conversely, when the budget is nearly spent, it should become more conservative to avoid violating service-level agreements (SLAs). This requires a robust mechanism for monitoring, threshold definition, and dynamic decision-making in our automated processes.

#### Triaging the Question

The core of this question revolves around how automation can dynamically adjust its behavior based on the current health of the error budget. The interviewer is probing your understanding of SLOs, error budgets, closed-loop systems, and real-time decision-making frameworks. We'll need to make some key assumptions:

The system has a clear definition of what constitutes an error or violation.
There are defined thresholds for the error budget that signal when it’s healthy, cautionary, and nearing its limits.

#### First Principles Solution
Define Error Budget Metrics: Establish metrics for SLO violations (e.g., service downtime, response time, availability). These form the basis of our error budget.
Monitor Current State: Continuously monitor these metrics to determine the current health of the system and the remaining error budget.
Thresholds for Actions: Set thresholds that define when actions should be taken:
Healthy Budget (50%+ Error Budget Remaining): Automation can act more aggressively by making bold changes, rolling out new features, or optimizing existing processes.
Cautionary Budget (20-49% Error Budget Remaining): The system should start conservatively applying changes to avoid risk but still make incremental improvements where possible.
Nearing Limit (10-19% Error Budget Remaining): Automation should be highly conservative, perhaps only performing maintenance tasks or rolling back recent changes if issues arise.
Dynamic Adjustment: Implement mechanisms for dynamically adjusting these thresholds based on historical data and current conditions.

#### Implementation
Steps

Error Budget Monitoring:

PYTHON
def monitor_error_budget(current_violations, total_errors):
    return (total_errors - current_violations) / total_errors

Threshold-Based Decision Logic:

PYTHON
def decide_action(error_budget_remaining):
    if error_budget_remaining >= 0.5:
        return "AGGRESSIVE"
    elif error_budget_remaining >= 0.2 and error_budget_remaining < 0.5:
        return "CAUTIONARY"
    else:
        return "CONSERVATIVE"

Integrate with Automation Workflow:

PYTHON
def automation_workflow(current_violations, total_errors):
    action = decide_action(monitor_error_budget(current_violations, total_errors))
    if action == "AGGRESSIVE":
        perform_aggressive_actions()
    elif action == "CAUTIONARY":
        perform_cautionary_actions()
    else:
        perform_conservative_actions()

#### Closing Summary
and Common Mistakes

By integrating this SLO/error-budget mechanism into our closed-loop automation architecture, we ensure that the system acts more aggressively when it has a healthy buffer of error budget, allowing for bold moves. Conversely, during periods where the budget is nearly spent, we become much more conservative to mitigate risk. This approach not only improves reliability but also optimizes resource usage and performance.

#### Common mistakes
include failing to properly define SLOs and error budgets, ignoring historical data in decision-making, or over-automating without considering human oversight.

Field Tags

**Comparison:**

```
Error Budget Remaining	Action
> 0.5	AGGRESSIVE
0.2 <= x < 0.5	CAUTIONARY
<= 0.2	CONSERVATIVE
```

**Code:**

```python
def monitor_error_budget(current_violations, total_errors):
    return (total_errors - current_violations) / total_errors

def decide_action(error_budget_remaining):
    if error_budget_remaining >= 0.5:
        return "AGGRESSIVE"
    elif error_budget_remaining >= 0.2 and error_budget_remaining < 0.5:
        return "CAUTIONARY"
    else:
        return "CONSERVATIVE"

def automation_workflow(current_violations, total_errors):
    action = decide_action(monitor_error_budget(current_violations, total_errors))
    if action == "AGGRESSIVE":
        perform_aggressive_actions()
    elif action == "CAUTIONARY":
        perform_cautionary_actions()
    else:
        perform_conservative_actions()
```

**Example input / output:**

```text
# Example Input/Output
current_violations = 10
total_errors = 20

error_budget_remaining = monitor_error_budget(current_violations, total_errors)
print("Error Budget Remaining:", error_budget_remaining)

action = decide_action(error_budget_remaining)
print("Action Plan:", action)
```

**Data:**

```text
- `current_violations`: Number of service-level objective violations observed.
- `total_errors`: Total number of errors or issues recorded.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monitor Error Budget
6. Is Error Budget Healthy?
7. Perform Aggressive Actions
8. Is Error Budget Cautionary?
9. Perform Cautionary Actions
10. Perform Conservative Actions

