---
type: "practice"
difficulty: "Intermediate"
---

# A pipeline runs the same test suite three times to 'reduce flakiness' and it's masking a real intermittent bug; how do you argue against retry-until-green and instead surface and fix flakiness?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a CI/CD pipeline running tests three times to mask an intermittent bug, let's first understand what’s really happening and why we should avoid relying on "retry-until-green" strategies.

#### Triaging the Question

The core issue here is that the pipeline is masking real bugs by retrying tests until they pass. This approach can be misleading because it gives a false sense of stability to the codebase, while the actual underlying issue persists. The goal should be to identify and resolve intermittent issues rather than silencing them.

#### First-Principles Solution

#### Assumptions
Intermittent Bug: There's an occasional bug in the test suite that passes sometimes but fails other times.
Impact on Productivity: Retrying tests can lead to wasted time, delayed feedback, and a false sense of security.
Cost of Investigation: The cost of investigating and fixing intermittent issues is lower than the cost of maintaining a system with unresolved bugs.
Step-by-Step Approach:

Identify the Root Cause:

Collect detailed logs from each test run to understand when and why tests fail.
Use monitoring tools to detect patterns in failures, such as timing or specific conditions that trigger intermittent issues.

Isolate the Environment:

Ensure consistent environments for testing by standardizing dependencies, libraries, and system configurations.
Use containerization (e.g., Docker) to create reproducible test environments.

Enhance Test Stability:

Implement retries within tests but not at a pipeline level. For example, retry individual failed test cases only once or twice before marking them as failed.
Use timeout mechanisms and graceful error handling in tests to make sure they don’t hang indefinitely.

Automate Flakiness Detection:

Introduce monitoring scripts that analyze test results over time to detect flaky tests automatically.
Set up alerts for developers when a certain threshold of failures is reached for a specific test case.

Investigate and Fix Flaky Tests:

Prioritize fixing intermittent issues based on their impact and frequency.
Encourage a culture where developers are responsible for ensuring the stability of their tests.
Use techniques like adding retries in test code, improving test setup/teardown logic, or refactoring complex tests.

#### Closing Summary

By focusing on identifying and addressing flakiness rather than masking it through multiple retries, we ensure that our codebase remains stable and reliable. This approach not only resolves issues more effectively but also improves overall productivity and developer satisfaction.

#### Common Mistakes
and Follow-Ups
Mistake 1: Relying too heavily on retry mechanisms without understanding the root cause.
Mistake 2: Ignoring intermittent failures, leading to compounded bugs over time.

#### Follow-Up
Discuss how to balance test stability with developer productivity. Consider using advanced CI/CD strategies like parallel testing and distributed load testing.

**Comparison:**

| Test Case | Status 1   | Status 2 | Status 3 | Notes                 |
|-----------|------------|----------|----------|-----------------------|
| test_a    | pass       | pass     | pass     | Stable                |
| test_b    | fail       | pass     | fail     | Intermittent          |
| test_c    | pass       | pass     | pass     | Stable                |

**Code:**

```python
# Example of a retry mechanism within tests (not recommended for the pipeline)
def test_something():
    attempts = 3
    while attempts > 0:
        try:
            # Test logic here
            return True
        except Exception as e:
            print(f"Test failed with {e}, trying again...")
            attempts -= 1
    raise Exception("Test failed after retries")
```

**Example input / output:**

```text
# Sample input for a monitoring script to detect flaky tests
Input:
{
  "tests": [
    {"name": "test_case_a", "status": "pass"},
    {"name": "test_case_b", "status": "fail"},
    {"name": "test_case_c", "status": "pass"}
  ]
}

# Output: Detected a potentially flaky test
Output:
{
  "flaky_tests": ["test_case_b"],
  "suggestion": "Investigate and fix 'test_case_b'"
}
```

**Data:**

```json
{
  "logs": [
    {
      "timestamp": "2023-10-01T14:00:00Z",
      "test": "unit_test_1",
      "outcome": "pass"
    },
    {
      "timestamp": "2023-10-01T14:05:00Z",
      "test": "unit_test_1",
      "outcome": "fail"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Flakiness
4. Is Test Consistent?
5. Monitor and Alert
6. Investigate and Fix
7. Prioritize Fixes
8. Implement Retries in Tests (ifnecessary)

