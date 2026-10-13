---
type: "practice"
difficulty: "Deep"
---

# An SLA breach post-mortem shows a DQ check silently returned zero rows and passed. How do you make checks fail-closed instead of fail-open?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validation Coverage and Quarantine](<../../../../02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a Data Quality (DQ) check failed to return any rows, thereby incorrectly passing an SLA, we need to ensure that such checks are "fail-closed." This means if there's even a single issue, the check should fail and trigger appropriate actions. Here’s how you can achieve this:

Firstly, let’s summarize: The current DQ system was silent on issues, leading to a false sense of security. To fix this, we need to introduce mechanisms that ensure any detected issues lead to immediate and actionable failure.

#### Triaging the Question
What is being probed: The interviewer is looking for your understanding of how DQ checks are implemented, and how they can be made more robust.

#### Assumptions
We’re dealing with a system where DQ checks are supposed to validate data integrity but failed to do so silently.

#### Solution
from First Principles
Identify the Issue: The core issue is that the check was not raising any errors or warnings, leading to an incorrect pass state.
Implement Error Handling: Ensure all potential issues trigger explicit failure states.
Logging and Alerting: Introduce comprehensive logging and alert mechanisms for any detected failures.
Regular Audits: Conduct regular audits of DQ checks to ensure they are effective.
Step-by-Step Solution

Modify the Check Logic:

Ensure every DQ check has a clear success or failure state.
If a single row fails validation, return an error and fail the entire check.

Introduce Error Logs:

For each failed check, log detailed information about the issue.
Use structured logging to capture relevant metadata such as timestamp, check type, failing data, etc.

Alerting Mechanisms:

Configure alerts for any DQ failures.
Ensure that these alerts are sent via multiple channels (email, Slack, SMS) to ensure timely awareness.

Regular Audits and Monitoring:

Schedule periodic reviews of all DQ checks.
Use monitoring tools to track the health and effectiveness of the checks over time.

#### Closing Summary

By implementing robust error handling, comprehensive logging, and effective alerting mechanisms, we can make sure that any DQ check failure is immediately detected and acted upon. This ensures data integrity and helps maintain SLA compliance by making the system "fail-closed" rather than "fail-open."

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to log detailed information about issues, leading to difficulty in diagnosing root causes.
Mistake 2: Not setting up alerts for DQ failures, resulting in delayed detection and resolution of issues.

#### Follow-Up
Questions:
How would you handle cases where a large dataset triggers an issue?
Can you discuss the trade-offs between fail-closed and other strategies like fail-safe?

**Comparison:**

```
Row	Name	Age
1	Alice	30
2	Bob	-5
Result
Failed
```

**Code:**

```python
def validate_data(row):
    # Example validation logic
    if row['age'] < 0 or row['age'] > 120:
        raise ValueError("Age is out of expected range")
    return True

# Example data
data = [
    {'name': 'Alice', 'age': 30},
    {'name': 'Bob', 'age': -5}
]

for row in data:
    try:
        validate_data(row)
    except ValueError as e:
        print(f"Validation failed: {e}")
```

**Example input / output:**

```text
Input:
[
  {'name': 'Alice', 'age': 30},
  {'name': 'Bob', 'age': -5}
]

Output:
Validation failed: Age is out of expected range
```

**Data:**

```json
{
  "data": [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": -5}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Data Valid?
5. Log Success and Pass
6. Log Error, Raise Alert, FailCheck
7. Audit and Review
8. Monitor and Audit Regularly

