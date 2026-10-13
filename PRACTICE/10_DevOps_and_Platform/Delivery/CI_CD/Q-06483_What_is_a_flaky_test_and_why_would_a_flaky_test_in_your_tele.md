---
type: "practice"
difficulty: "Basic"
---

# What is a 'flaky' test, and why would a flaky test in your telemetry-pipeline repo slow the whole team down?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps environment, "flaky tests" refer to automated tests that fail intermittently and unpredictably for no apparent reason, even though the underlying code remains unchanged. These tests can significantly slow down a Continuous Integration/Continuous Deployment (CI/CD) pipeline because they may repeatedly trigger unnecessary manual interventions, causing delays in the overall development process.

To understand why flaky tests are problematic, let's break down what CI/CD is and then look at how flaky tests impact it. CI/CD involves automating the software delivery process to integrate changes from multiple contributors, run automated tests, and deploy code to production. The goal is to reduce manual intervention and accelerate the release cycle.

#### Triage

What is being probed: The interviewer wants to understand your knowledge of CI/CD processes, specifically how flaky tests can disrupt automation.

#### Assumptions
We assume you are familiar with basic CI/CD concepts and can discuss common issues that arise in such environments.

#### First-principles solution

Definition of Flaky Tests:

A flaky test is one where the outcome (pass or fail) varies unpredictably, even though the input remains constant.
This variability makes it difficult to determine whether a failure is due to an actual issue in the code or just a transient problem.

Impact on CI/CD Pipeline:

When flaky tests are present in your telemetry-pipeline repo, they can cause several issues:
False Positives: A test failing intermittently can lead to false alarms, making it hard for developers to pinpoint real issues.
Manual Interventions: Continuous manual checks and re-runs of tests delay the build process and increase friction in development.
Slow Feedback Loops: Developers need to spend time verifying whether a failure is due to their changes or if it's just another flaky test, which slows down the feedback loop.

Reasoning Step-by-Step:

Identifying Flaky Tests: During the CI/CD pipeline run, if tests fail unpredictably, they need to be flagged as potential flaky candidates.
Re-running Tests: Implement a mechanism where these tests are re-run multiple times to ensure their outcomes are consistent.
Logging and Monitoring: Use logging and monitoring tools to track test failures and identify patterns or root causes.
Isolation and Prioritization: Isolate flaky tests from other tests to prevent them from blocking the entire pipeline. Prioritize resolving flaky issues over less critical ones.

#### Closing Summary
Flaky tests can significantly hinder a team's productivity by introducing unnecessary manual steps, slowing down feedback loops, and causing confusion about real issues. Effective strategies include identifying and re-running flaky tests, isolating them, and prioritizing their resolution to maintain the smooth operation of CI/CD pipelines.

#### Common Mistakes/Follow-Ups

Not Recognizing Flakiness: Teams might dismiss test failures as just part of the process without investigating.
Lack of Automation: Relying solely on manual interventions for flaky tests can lead to human error and delays.
Overlooking Data Collection: Failing to log and analyze data from failed tests can make it harder to identify root causes.

**Comparison:**

```
Test Attempt	Result
1	Pass
2	Fail
3	Pass
4	Pass
5	Pass
```

**Code:**

```python
# Example Python code snippet to re-run a test multiple times
import random

def run_test(test):
    for attempt in range(5):  # Run the test up to 5 times
        result = test()
        if not random.choice([True, False]):  # Simulate flakiness
            continue
        return result
    raise Exception("Test failed after multiple attempts")
```

**Example input / output:**

```text
Input:
- Test function `run_test` simulates a flaky test that fails intermittently.
Output:
- Test is re-run up to 5 times. Results after multiple attempts: ['pass', 'fail', 'pass', 'pass', 'pass']
```

**Data:**

```json
{
  "testResults": [
    {"attempt": 1, "result": "pass"},
    {"attempt": 2, "result": "fail"},
    {"attempt": 3, "result": "pass"},
    {"attempt": 4, "result": "pass"},
    {"attempt": 5, "result": "pass"}
  ]
}
```

**Flow:**

1. Yes
2. Pass
3. Fail
4. No
5. Start
6. Is Test Flaky?
7. Run Test Multiple Times
8. Check Results
9. Mark as Non-Flaky
10. Report and Log
11. Continue with Pipeline

