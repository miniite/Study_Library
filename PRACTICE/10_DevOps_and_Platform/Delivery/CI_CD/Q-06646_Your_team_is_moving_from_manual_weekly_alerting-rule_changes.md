---
type: "practice"
difficulty: "Deep"
---

# Your team is moving from manual weekly alerting-rule changes to continuous deployment of rules; what pipeline capabilities (linting, replay tests, canary, auto-rollback) would you build first and why?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deployment Observability](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When transitioning from manual weekly alerting rule changes to a continuous integration/continuous deployment (CI/CD) pipeline for alerting rules, we need to ensure that our deployments are both reliable and safe. The first capabilities I would build in the pipeline are linting and replay tests, as they form the foundation of ensuring code quality and functionality without disrupting production.

#### Triage
What the Question is Asking

The question is probing into the critical components needed for a robust CI/CD pipeline for alerting rules. Specifically, it asks which capabilities to prioritize first and why. This requires an understanding of how linting and replay tests work, their benefits, and potential trade-offs with other features like canary deployments and automatic rollbacks.

#### Assumptions
The team is moving from manual weekly changes to automated deployments.
The alerting rules are written in a specific language or framework (e.g., Python, YAML).
Production reliability is critical, and downtime should be minimized.

#### First-Principles Solution
1. Linting

Linting is the process of checking for coding standards, style guide violations, and potential errors in code before it is committed to version control. For alerting rules, linting ensures that:

The rules follow a consistent format.
There are no obvious syntax or semantic errors.
The rules adhere to best practices (e.g., avoiding redundant conditions).

Why Linting First: Linting acts as the first line of defense against basic coding mistakes. It helps catch errors early in the development process, reducing the likelihood of bugs making it into production. This is crucial for alerting rules because any incorrect rule could lead to false positives or false negatives, causing unnecessary alerts or missed issues.

Source Code
Linter
Compliance Check
2. Replay Tests

Replay tests involve running the alerting rules on historical data to ensure they behave as expected in production-like scenarios. This step helps verify that:

The rules generate alerts correctly based on past events.
There are no unexpected side effects or performance issues.

Why Replay Tests First: Replay testing is critical because it simulates real-world conditions and ensures the alerting logic performs as intended with actual data. It addresses concerns about false positives and negatives, ensuring that the changes do not break existing workflows or introduce new issues.

Historical Data
Rule Application
Alert Generation
Compliance Check
3. Canaries and Auto-Rollback

While canary deployments and auto-rollback are important, they should be built later as they add additional complexity. Canaries involve deploying the new rules to a subset of production environments first to ensure everything works as expected before rolling out globally. Auto-rollback automatically reverts changes if an issue is detected during canary or full deployment.

Why Build Later: Canaries and auto-rollback are valuable for minimizing risks, but they come with increased overhead and complexity. They should be implemented only after basic validation steps like linting and replay tests have been established to ensure the core functionality is robust.

#### Closing Summary

In summary, the first capabilities to build in the CI/CD pipeline for alerting rules are linting and replay tests. These foundational steps provide essential safeguards against basic coding mistakes and ensure that changes are thoroughly tested before production deployments. Canaries and auto-rollback can be added later as they offer additional protections but require more complex infrastructure.

#### Common Mistakes/Follow-Ups
Skipping Linting: Failing to perform initial lint checks can lead to a significant number of issues only being discovered in production.
Overlooking Replay Testing: Not running tests on historical data might result in unexpected behavior or missed issues, causing downtime.
Implementing Canaries and Auto-Rollback Too Early: These features add complexity without solidifying the basic validation steps.

**Comparison:**

```
Test	Description
Linting	Checks code against coding standards and best practices.
Replay Testing	Runs rules on historical data to simulate production behavior.
```

**Code:**

```python
# Example Linter Check Function (Pseudo-code)
def linter_check(rule):
    if not rule.is_valid_format():
        return False
    for condition in rule.conditions:
        if not condition.is_valid_syntax():
            return False
    return True

# Example Replay Test Function (Pseudo-code)
def replay_test(rule, historical_data):
    generated_alerts = rule.apply(historical_data)
    expected_alerts = calculate_expected_alerts(historical_data)
    assert generated_alerts == expected_alerts, "Replay test failed"
```

**Example input / output:**

```text
Input:
- Historical Data: JSON file containing past CPU and Memory usage metrics.

Output:
- Generated Alerts: List of alerts based on the replayed historical data.
- Expected Alerts: Pre-calculated list of expected alerts for comparison.
```

**Data:**

```json
{
  "rule": {
    "id": "R001",
    "conditions": [
      {"metric": "CPU Usage", "threshold": 80},
      {"metric": "Memory Usage", "threshold": 90}
    ]
  }
}
```

**Flow:**

1. Source Code
2. Linter Check
3. Pass
4. Compliance Check
5. Fails
6. Fix and Commit
7. Historical Data
8. Rule Application
9. Alert Generation
10. Expected Alerts Comparison
11. Pass
12. Deployment to Production
13. Fails
14. Replay Test Debugging

