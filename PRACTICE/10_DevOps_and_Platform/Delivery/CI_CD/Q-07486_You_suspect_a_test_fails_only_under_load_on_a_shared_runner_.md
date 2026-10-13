---
type: "practice"
difficulty: "Deep"
---

# You suspect a test fails only under load on a shared runner; how would you determine whether it's flaky infra, a real concurrency bug, or a resource limit, and prove which it is?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why a test might be failing only under load in a shared CI/CD environment, we need to systematically rule out various causes. The key steps involve understanding the nature of the failure, analyzing logs and metrics, and potentially setting up dedicated runners for testing.

Firstly, we should consider whether it’s due to flaky infrastructure (e.g., network instability, resource contention), a concurrency bug in the code itself, or simply hitting some resource limits that aren't adequately accounted for. Let's break this down step-by-step:

#### Triage
What We’re Probing

The question is essentially asking us to determine whether the intermittent test failures are due to:

Flaky Infrastructure: Network issues, resource contention, or other environmental factors.
Concurrency Bugs: Issues arising from how the code handles concurrent execution.
Resource Limits: Whether the shared runner’s resources aren't sufficient for running tests under load.

#### Assumptions
We have access to logs and metrics from both the CI/CD environment and the application being tested.
The test suite is designed well enough that failures are not due to misconfigurations or test flakiness itself.
The shared runner can be dedicated for specific types of tests if necessary.

#### First Principles Solution

Isolate Environment Factors

Step 1: Analyze CI/CD Logs: Look for patterns in the logs around when and how failures occur. Are there consistent failure points or time intervals?
Step 2: Resource Metrics: Check CPU, memory, network latency metrics on the shared runner during test runs.

Reproduce Locally

Step 3: Reproduce the Test Locally: Try to replicate the same environment (network, resource usage) locally or in a controlled setup to see if the issue persists.
Step 4: Use Dedicated Runners: If possible, run tests on a dedicated runner to eliminate shared infrastructure issues.

Concurrency Analysis

Step 5: Concurrency Control: Introduce additional concurrency control mechanisms (e.g., increasing test timeouts, implementing retry logic) and observe the outcomes.
Step 6: Code Review: Conduct code reviews focusing on concurrency patterns and ensure there are no race conditions or deadlocks.

Resource Limits

Step 7: Stress Test: Perform stress tests under load to see if resource limits (CPU, memory, network) are being hit.
Step 8: Adjust Resources: If resource limits are reached, consider scaling the runner’s resources or optimizing test execution times.

#### Closing Summary
and Common Mistakes/Follow-Ups

By following these steps, we can systematically determine whether the issue is related to infrastructure, concurrency bugs, or resource limitations. The key takeaway is that each step should be methodical and based on evidence from logs, metrics, and local reproductions. Common mistakes include jumping to conclusions too quickly without thorough investigation and not considering all possible root causes.

#### Follow-ups
might involve further investigation into the shared runner’s configuration, deeper code analysis, or even setting up a more robust CI/CD pipeline that can handle load testing effectively.

**Comparison:**

| Time     | Runner Metrics         |
|----------|------------------------|
| 12:00 PM | CPU Usage: 90%         |
| 12:30 PM | Memory Usage: 85%      |
| 1:00 PM  | Network Latency: 20ms   |

**Code:**

```python
# Example Python script for logging test execution details
import logging

def execute_test(test_case):
    try:
        result = run_test(test_case)
        log_result(result)
    except Exception as e:
        log_exception(e)

def run_test(test_case):
    # Test logic here
    pass

def log_result(result):
    logging.info(f"Test {test_case} passed: {result}")

def log_exception(e):
    logging.error(f"Exception occurred: {e}")
```

**Example input / output:**

```
Input: Run a test suite on a shared CI/CD runner.
Output:
Logs showing network latency spikes during certain intervals.
Metrics indicating high CPU and memory usage.
Dry-run Example:
PLAINTEXT
Copy
10:32:45 [INFO] Test case A passed: True
10:33:07 [ERROR] Exception occurred: Network timeout exception
10:33:18 [INFO] Test case B passed: False (due to resource limits)
```

**Data:**

```json
{
  "runner_metrics": {
    "cpu_usage": "90%",
    "memory_usage": "85%",
    "network_latency": "12ms"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Isolate Environment Factors
7. Analyze CI/CD Logs
8. Check Resource Metrics
9. Reproduce Locally
10. Use Dedicated Runners?
11. Test Concurrency Control
12. Code Review?
13. Adjust Resources
14. Dedicated Runner SetupRequired
15. Stress Test Under Load

