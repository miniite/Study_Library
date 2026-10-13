---
type: "practice"
difficulty: "Deep"
---

# How do you quantify a 'cognitive load' quality attribute for the platform so an ATAM can analyze it — e.g. 'a new SRE can safely execute a cluster upgrade within their first month using only the runbook'? What makes this measurable rather than hand-wavy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quantify a "cognitive load" quality attribute for a platform, we need to break down the process into actionable steps and measure them against real-world metrics that SREs will encounter. Cognitive load can be thought of as the mental effort required by an individual to perform a task correctly. In this context, we want to ensure that critical tasks like cluster upgrades are manageable within certain constraints.

The key idea is to define what "can safely execute" means in concrete terms and then measure how much cognitive effort it requires. For instance, if a new SRE can complete a cluster upgrade using only the runbook and should be able to do so without needing immediate assistance, we need to quantify this by breaking down each step of the process.

Let's consider an example scenario:

A cluster upgrade task involves several steps: environment setup, pre-upgrade checks, running the actual upgrade, post-upgrade verification.
Each step has associated tasks and decision points that require attention. For instance, checking for prerequisites, ensuring correct version compatibility, handling potential errors during the upgrade process.

Now, let's go through the first-principles solution:

Define the Task: Clearly define what "safe execution" means. In this case, we want to ensure the SRE can complete the task independently with minimal guidance.
Break Down the Steps: Decompose the cluster upgrade into smaller, manageable tasks. Each step should be as simple and self-explanatory as possible.
Cognitive Load Metrics:
Step Complexity: Measure how complex each step is in terms of the number of decisions required, the amount of documentation needed, and potential risks involved.
Time to Completion: Track how long it takes for an SRE with minimal experience (e.g., a new hire) to complete each step. This gives us an idea of the time burden.
Error Rates: Monitor error rates during the upgrade process. High error rates might indicate that the task is too complex or poorly documented.
Runbook Quality Metrics:
Clarity and Detail: Ensure runbooks are detailed enough to cover all possible scenarios, but not overly verbose.

#### Cons
istency: Runbooks should be consistent across different environments to avoid confusion.

To make this measurable rather than hand-wavy, we need a structured approach:

Metrics Collection: Collect data on the time taken for each step, error rates, and feedback from SREs during dry runs or actual upgrades.
Feedback Loops: Regularly review and update runbooks based on real-world performance metrics. This ensures that any gaps in documentation or understanding are addressed promptly.

#### Closing Summary
Quantifying cognitive load involves breaking down tasks into smaller steps, defining clear success criteria, and using measurable data to track progress. By focusing on key metrics like step complexity, time to completion, error rates, and runbook quality, we can ensure that critical operations remain within the manageable range for new SREs.

#### Common Mistakes/Follow-Ups

Overcomplicating Metrics: Focusing too much on overly complex metrics might obscure the real issues.
Lack of Consistency: Inconsistent runbooks or processes can lead to confusion and higher cognitive load.
Ignoring Feedback Loops: Not regularly reviewing and updating documentation based on actual usage data can lead to stagnation in process improvement.

**Comparison:**

```
Step	Time Taken (s)	Error Rate
Setup	15	0.0
Checks	30	0.1
Upgrade	60	0.2
Verify	45	0.0
```

**Code:**

```python
# Example Python code for tracking upgrade step completion time
import time

def track_upgrade_step(step_name, start_time):
    end_time = time.time()
    duration = end_time - start_time
    print(f"Step '{step_name}' completed in {duration:.2f} seconds")
```

**Example input / output:**

```text
Input:
- A new SRE starts the cluster upgrade process.

Output:
- Environment Setup completed in 15 seconds.
- Pre-Upgrade Checks completed in 30 seconds with one minor error.
- Running Upgrade completed in 60 seconds with two major errors.
- Post-Upgrade Verification completed in 45 seconds.

Error Analysis:
- Pre-Upgrade Checks: One minor error, suggesting the documentation could be clearer.
- Running Upgrade: Two major errors, indicating potential issues with the upgrade process itself or unclear instructions.
```

**Data:**

```json
{
  "upgrade_steps": [
    {
      "name": "Environment Setup",
      "time_taken": 15.0,
      "error_rate": 0.0
    },
    {
      "name": "Pre-Upgrade Checks",
      "time_taken": 30.0,
      "error_rate": 0.1
    },
    {
      "name": "Running Upgrade",
      "time_taken": 60.0,
      "error_rate": 0.2
    },
    {
      "name": "Post-Upgrade Verification",
      "time_taken": 45.0,
      "error_rate": 0.0
    }
  ]
}
```

**Flow:**

1. Runbook
2. Environment Setup
3. Pre-Upgrade Checks
4. Running Upgrade
5. Post-Upgrade Verification
6. Step Complexity
7. Time to Completion
8. Error Rates
9. Runbook Quality Metrics

