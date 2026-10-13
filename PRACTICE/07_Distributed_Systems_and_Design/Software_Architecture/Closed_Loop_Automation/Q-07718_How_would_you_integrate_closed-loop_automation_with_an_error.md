---
type: "practice"
difficulty: "Expert"
---

# How would you integrate closed-loop automation with an error-budget policy so exhausting the budget automatically tightens what the loop is allowed to do?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Business Aware Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Business_Aware_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with integrating a closed-loop automation system with an error-budget policy. The goal is to ensure that when the error budget is exhausted, the automation system adapts by tightening its constraints or adjusting its actions accordingly. To achieve this, let's first define what each component does and then outline a step-by-step approach.

**Code:**

```python
# Example of an error budget policy class in Python

class ErrorBudgetPolicy:
    def __init__(self, initial_budget):
        self.budget = initial_budget

    def decrement(self, failure_count):
        self.budget -= failure_count
        if self.budget <= 0:
            return True  # Budget exhausted
        return False

# Example of a closed-loop automation system component

class AutomationSystem:
    def __init__(self, policy):
        self.policy = policy

    def perform_task(self, task):
        outcome = execute_task(task)  # Assume this function returns success or failure status
        if not outcome:
            if self.policy.decrement(1):  # Deduct from the error budget
                self.tighten_constraints()  # Adjust constraints when budget exhausted

    def tighten_constraints(self):
        # Implement logic to adjust automation behavior based on remaining budget
        pass
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Error Budget Exhausted?
5. Tighten Constraints
6. Continue Operations

