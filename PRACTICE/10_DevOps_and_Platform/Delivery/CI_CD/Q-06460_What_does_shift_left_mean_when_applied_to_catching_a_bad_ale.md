---
type: "practice"
difficulty: "Basic"
---

# What does 'shift left' mean when applied to catching a bad alert rule early in a pipeline rather than in production?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Shift left is a DevOps principle that emphasizes moving quality and security checks closer to the start of the development process, where potential issues can be detected earlier, reducing costs and effort compared to finding them later in the pipeline or even in production.

In the context of catching bad alert rules early, 'shift left' means identifying and fixing problematic alert rules before they are deployed into a live environment. This involves moving alert rule validation steps as close as possible to the initial stages of development, such as during code review, static analysis, or unit testing phases.

First, we need to understand what an "alert rule" is: it’s a predefined condition in monitoring systems that triggers notifications when certain conditions are met. For example, if a server's CPU usage exceeds 80%, this could trigger an alert.

To implement 'shift left' for catching bad alert rules early:

Code Review: During the code review process, developers should ensure that new or modified alert rules make sense and align with the application’s operational needs.
Static Analysis Tools: Use static analysis tools to check alert rules during development. These tools can flag potential issues before deployment.
Unit Testing: Incorporate tests specifically for alert rules within unit test suites, ensuring they behave as expected in controlled environments.

For instance, if a developer writes an alert rule that checks the number of failed requests per minute and sets this threshold too low, it could lead to false positives in production. By catching such issues early via static analysis or during code review, the team can refine these rules before deployment.

**Comparison:**

```
Code Review	Static Analysis	Unit Testing
Check logic correctness and alignment with requirements.	Flag potential issues early in development.	Ensure rules behave as expected in controlled environments before deployment.
```

**Code:**

```python
# Example of a unit test for an alert rule
import unittest

class TestAlertRule(unittest.TestCase):
    def test_failing_threshold(self):
        # Mocked data
        failed_requests = [10, 20, 30]

        # Alert threshold set too low (5)
        self.assertTrue(check_alert_rule(failed_requests, 5), "Threshold should have triggered an alert")

    def test_passing_threshold(self):
        # Mocked data with a higher limit
        failed_requests = [1, 2, 3]

        # Alert threshold set appropriately (40)
        self.assertFalse(check_alert_rule(failed_requests, 40), "No alert should have been triggered")
```

**Example input / output:**

```text
Input:
- Alert rule threshold set to 5 for the number of failed requests per minute.
- Mocked data with values exceeding this threshold.

Output:
- Unit test passes, indicating the alert should trigger.
```

**Data:**

```json
{
  "alertRule": {
    "id": 123,
    "name": "High Request Failures",
    "threshold": 5,
    "status": "active"
  },
  "mockedData": [
    { "time": "2023-04-01", "value": 10 },
    { "time": "2023-04-02", "value": 20 },
    { "time": "2023-04-03", "value": 30 }
  ]
}
```

**Flow:**

1. Code Review
2. Unit Test
3. Static Analysis
4. Unit Test
5. Implementation of Alert Rule
6. Unit Test

