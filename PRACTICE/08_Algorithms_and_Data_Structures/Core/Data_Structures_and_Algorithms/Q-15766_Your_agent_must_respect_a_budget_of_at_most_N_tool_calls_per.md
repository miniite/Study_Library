---
type: "practice"
difficulty: "Intermediate"
---

# Your agent must respect a budget of at most N tool calls per task; what counter structure and where in the call stack do you enforce it so a recursive sub-agent can't blow the global budget?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an AI agent respects a budget of at most (N) tool calls per task, we need to implement a counter mechanism within the call stack. The key idea is to track and manage the number of recursive calls made by the sub-agent without exceeding the global budget. Here’s how you can achieve this:

TL;DR: We will use a local counter in each function call to monitor the number of tool calls, ensuring that no single task exceeds the budget (N). This is done through a combination of recursion and context-aware counters.

#### Triage
The core issue is managing recursive function calls while respecting a global budget constraint. The interviewer might be probing on how you handle nested recursions, manage shared resources across multiple function calls, and ensure thread safety if the problem involves concurrent execution.

#### First-Principles Solution

Define a budgetManager class that maintains the current count of tool calls for each task.
Each recursive call will check if it can proceed based on the remaining budget in its context.
Use a local counter within each function to keep track of the number of calls made under that specific task.

Here’s a step-by-step breakdown:

Define the Budget Manager: Create a BudgetManager class with methods to initialize, check, and decrement the budget for a given task.
Integrate in Recursive Calls: Ensure every recursive call checks if it can proceed by checking against its local or inherited budget counter.
PYTHON
class BudgetManager:
    def __init__(self, max_budget):
        self.max_budget = max_budget
        self.current_budget = 0

    def has_budget(self):
        return self.current_budget < self.max_budget

    def use_budget(self):
        if not self.has_budget():
            raise Exception("Budget exceeded")
        self.current_budget += 1

    def reset_budget(self):
        self.current_budget = 0

def recursive_function(task, budget_manager):
    if budget_manager.has_budget():
        # Use the tool
        budget_manager.use_budget()

        # Recursive call (example operation)
        result = some_operation(task)
        sub_result = recursive_function(sub_task, budget_manager)

        return combine_results(result, sub_result)
    else:
        raise Exception("Budget exceeded for current task")

# Example usage
budget_manager = BudgetManager(max_budget=10)  # Set the max budget to 10 tool calls
try:
    final_result = recursive_function(main_task, budget_manager)
except Exception as e:
    print(e)

#### Closing Summary
By integrating a BudgetManager class within each function call, we ensure that no single task exceeds its allocated budget. This approach is context-aware and allows for dynamic adjustment of the budget based on the current state.

#### Common Mistakes/Follow-Ups

Not Accounting for Context Changes: Ensure the budget counter is updated correctly in sub-functions if they affect the overall budget.
Ignoring Edge Cases: Consider scenarios like task splitting, parallel execution, and ensuring thread safety in concurrent environments.
Performance Overhead: Be mindful of the overhead introduced by the additional checks and counters. In some cases, a simpler approach might be more efficient.

**Code:**

```python
class BudgetManager:
    def __init__(self, max_budget):
        self.max_budget = max_budget
        self.current_budget = 0

    def has_budget(self):
        return self.current_budget < self.max_budget

    def use_budget(self):
        if not self.has_budget():
            raise Exception("Budget exceeded")
        self.current_budget += 1

    def reset_budget(self):
        self.current_budget = 0
```

**Example input / output:**

```text
budget_manager = BudgetManager(max_budget=10)
try:
    result = recursive_function(main_task, budget_manager)
except Exception as e:
    print(e)  # Output: Budget exceeded for current task if the budget is exhausted.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a budget?
5. Use the tool
6. Budget -= 1
7. Proceed with recursive call
8. End
9. Exceeded Budget Error
10. Handle error or revert state
11. End

