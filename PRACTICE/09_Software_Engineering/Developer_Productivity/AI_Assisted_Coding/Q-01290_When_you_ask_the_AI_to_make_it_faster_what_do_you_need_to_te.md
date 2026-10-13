---
type: "practice"
difficulty: "Intermediate"
---

# When you ask the AI to 'make it faster', what do you need to tell it so the optimization is measurable?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Onboarding to Codebases with AI](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Onboarding_to_Codebases_with_AI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you ask an AI like Copilot or Cursor to "make it faster," you need to provide clear and specific instructions that allow the tool to understand the performance goals and context of your code. The key is to make sure the optimization is measurable, meaning both the starting point (current performance) and the target (desired improvement) should be well-defined.

Firstly, let's clarify what "faster" means in this context. We need a concrete metric such as execution time or resource usage. For instance, you might want to reduce the runtime of your code from 10 seconds to 2 seconds on average for a specific task.

To achieve this, follow these steps:

Identify the Performance Bottleneck: Start by profiling your code to find out where the slowest parts are. Tools like gprof, perf, or even simple logging can help identify hotspots.

Specify the Goal Clearly: When you ask Copilot/Cursor to optimize, provide a specific goal. For example, "Make this function run 30% faster" or "Reduce the number of database queries in this block by half."

Provide Contextual Information: The AI needs context to make informed decisions. For instance, if you're working with a large dataset and using sorting algorithms, let Copilot/Cursor know that you’re targeting an O(n log n) algorithm but want a faster implementation.

Measure Incrementally: Optimize in small, incremental steps and measure the impact after each change to ensure progress towards your goal.

Refine Your Requests: As the AI makes changes, refine your requests based on the outcomes. If you find that certain optimizations aren’t working as expected, provide feedback so the AI can adjust its approach.

By following these steps, you can make sure that the optimization is not just a superficial change but a measurable improvement in performance.

#### Closing Summary
When asking an AI to optimize code, it's crucial to be specific about what "faster" means and provide enough context for the tool to understand your goals. Measurable metrics and incremental changes are key to achieving meaningful optimizations.

**Comparison:**

```
Step	Action	Expected Outcome
1	Profile code	Identify bottlenecks
2	Set optimization goals	Specific runtime or resource reduction targets
3	Implement changes incrementally	Test and measure improvements after each step
4	Provide feedback to AI	Adjust further optimizations based on outcomes
```

**Code:**

```python
# Example of a profiling script using Python's cProfile module

import cProfile
import pstats

def example_function(n):
    # Some operations that might be slow
    for i in range(n):
        pass

cProfile.run('example_function(10000)')

# After making optimizations, rerun and compare stats
```

**Example input / output:**

```text
- **Initial Scenario**: Function runs in 10 seconds.
- **Optimized Scenario**: After making changes, the function now runs in approximately 7 seconds, achieving a 30% reduction.

- **Feedback Loop**:
  - Initial Request: "Make this function run 30% faster."
  - Optimized Function (with new metrics): Runs in 6.5 seconds.
  - Revised Request: "Target at least a 40% improvement on the current runtime."

- **Final Target**: A more aggressive optimization request, resulting in a run time of around 3 to 4 seconds.

This iterative process ensures that each change is measured and contributes towards the overall goal of making the function faster.
```

**Data:**

```json
{
  "metrics": {
    "current_runtime": 10,
    "desired_reduction": 30
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Identify Bottleneck
5. Profile Code
6. Set Optimization Goals
7. Implement ChangesIncrementally
8. Test and MeasureImprovements
9. Provide Feedback to AI
10. Check if Goal Met

