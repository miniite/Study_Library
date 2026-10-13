---
type: "practice"
difficulty: "Basic"
---

# Why is it a good idea to test a new dashboard in a staging monitoring instance before pushing it to the one on-call uses?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Artifact Promotion Across Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Artifact_Promotion_Across_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Testing a new dashboard in a staging monitoring instance before deploying it to the production environment is crucial because it helps catch issues early, reduces downtime during on-call periods, and ensures that the changes do not disrupt existing operations.

#### Triaging the Question

The core of this question revolves around understanding why it's beneficial to have a staging environment for testing CI/CD pipelines. Specifically, the interviewer might be probing your knowledge of:

The importance of quality assurance in software development.
Understanding of the trade-offs between manual and automated testing.
Awareness of potential risks associated with pushing changes directly into production.

#### First Principles Solution
Risk Mitigation: Testing in a staging environment allows you to identify issues before they impact users. This can significantly reduce the risk of downtime or performance degradation during critical on-call periods.
Quality Assurance: By testing, you ensure that the new dashboard meets the necessary standards and works as expected. This includes validating data integrity, user experience, and integration with other systems.
User Experience: Users rely heavily on dashboards for monitoring and troubleshooting. Any issues can lead to delays in incident resolution or confusion during critical events.
Step-by-Step Explanation
Staging Environment Setup: The staging environment is a copy of the production environment but isolated from it. It mimics the real-world conditions where the new dashboard will be deployed.
Test Case Creation: Develop test cases that cover various scenarios, including edge cases and normal operations. For example:
Verify data accuracy
Ensure proper performance metrics display
Test interaction with other systems (e.g., alerting services)
Manual vs Automated Testing: While manual testing can provide insights into user experience, automated tests are crucial for ensuring consistent behavior across different scenarios.
Monitoring and Logging: Implement logging to capture any anomalies during the testing phase. This helps in quickly identifying and fixing issues before deployment.

#### Closing Summary

Testing a new dashboard in a staging monitoring instance is essential because it minimizes risks, improves quality assurance, and ensures a smooth user experience. By catching and resolving issues early, you can avoid costly downtime and maintain reliability during on-call periods.

#### Common Mistakes
and Follow-Ups
Overlooking Edge Cases: Failing to test edge cases can lead to unforeseen issues that only manifest in production.
Insufficient Logging: Poor logging practices can make it difficult to diagnose problems when they arise.
Skipping Manual Testing: Relying too heavily on automated tests without manual validation might overlook subtle user experience issues.

#### Triaging the Question
Revisited

The interviewer is likely looking for a nuanced understanding of both technical and operational aspects. Highlighting specific scenarios where staging environments have prevented critical issues can add depth to your response.

**Comparison:**

```
Scenario	Test Case	Expected Outcome
Data Accuracy	Verify all data points have values.	All metrics should pass validation without null values.
Integration	Ensure the dashboard integrates seamlessly with alerting services.	Alerting service calls should complete successfully.
```

**Code:**

```python
# Example test script in Python using pytest
def test_dashboard_data_accuracy():
    # Simulate dashboard data retrieval
    data = fetch_dashboard_data()
    assert len(data) > 0, "Dashboard should not be empty"
    for metric in data:
        assert metric['value'] is not None, "Metric value cannot be null"

def test_integration_with_alerting_service():
    # Mock alerting service call
    response = mock_alerting_service_call()
    assert response.status_code == 200, "Alerting service should respond successfully"
```

**Example input / output:**

```text
Input:
- Staged dashboard data with two metrics

Output:
- Pass tests for non-empty dashboard and valid metric values
- Fail tests if any metric value is null or if alerting service call fails
```

**Data:**

```json
{
  "dashboard": {
    "name": "Staging Dashboard",
    "metrics": [
      { "id": 1, "value": 95 },
      { "id": 2, "value": 88 }
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. Staging Environment Setup
4. Is Dashboard Fully Tested?
5. Identify and Fix Issues
6. Repeat Testing
7. Deploy to Production

