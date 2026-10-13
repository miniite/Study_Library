---
type: "practice"
difficulty: "Expert"
---

# Retrospect an incident where a gateway config canary passed but full rollout failed because prod had traffic patterns staging didn't; redesign the org's config-validation to use production traffic replay/shadowing so environment-specific breakage is caught before rollout.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we faced a challenging issue where our API Gateway configuration passed canary testing in staging but failed during full rollout into production due to traffic patterns that were not adequately replicated in staging. The solution is to redesign the organization's configuration validation process to leverage production traffic replay or shadowing, which will help catch any environment-specific issues before a full release.

#### Triaging the Question

The key points of this question are:

Understanding the Issue: Identifying why the canary test passed but the full rollout failed.
Environment Differences: Recognizing that staging and production environments have different traffic patterns, leading to subtle differences in behavior.
Config Validation: Designing a more robust validation mechanism that accounts for these differences.
Production Traffic Replay/Shadowing: Implementing a system where production traffic is replayed or shadowed during the validation phase.

#### Assumptions
and First-Principles Reasoning

#### Assumptions
The current canary testing process works by simulating some aspects of the production environment but may not fully replicate its behavior.
Production environments have unique traffic patterns, such as high concurrency levels or specific user behaviors, that are not present in staging.
A more detailed validation step is needed to ensure that configurations work correctly across all environments.
Step-by-Step Solution

Traffic Pattern Analysis:

Collect and analyze production traffic data to identify unique patterns and behaviors.
This can be done using log analysis tools or custom scripts that capture request attributes, response times, error rates, etc.

Shadowing Mechanism:

Develop a shadowing mechanism where selected requests from the production environment are replayed into the staging environment.
This could involve setting up reverse proxies or custom routing rules in the API Gateway to route specific traffic to staging.

Automated Testing Frameworks:

Implement automated testing frameworks that can validate configurations against actual production data.
For example, use tools like K6 for load testing, and set up scripts that replay historical requests to test different scenarios.

Continuous Integration/Continuous Deployment (CI/CD) Enhancements:

Integrate the traffic validation step into the CI/CD pipeline so that every release candidate is tested with shadowed production traffic.
Ensure this validation step runs automatically and fails builds if any issues are detected.

Monitoring and Logging:

Implement comprehensive logging to capture detailed information about both production and staging environments.
Use monitoring tools like Prometheus or Grafana to visualize metrics and detect anomalies in real-time.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By implementing a shadowing mechanism and automated testing with production data, we can catch environment-specific issues earlier in the release cycle. This approach significantly reduces the risk of full rollouts failing due to subtle differences between staging and production environments.

#### Common Mistakes
Failing to thoroughly analyze production traffic patterns, leading to incomplete validation coverage.
Overlooking edge cases that are unique to production but not present in staging.
Not integrating these validations into the CI/CD pipeline, making it difficult to catch issues early.
Trialing the Solution

To validate this solution, we can:

Run a Pilot Test: Select a subset of production traffic and replay it through the shadowing mechanism.
Monitor Results: Analyze the results to ensure that configurations are validated accurately.
Iterate Based on Feedback: Make adjustments as needed based on feedback from the pilot test.

#### Follow-Ups
How would you handle situations where traffic patterns change frequently?
What additional metrics or logging can be added to improve visibility and traceability?

**Comparison:**

```
Time	Request Method	Path	Response Time	Status
2023-10-15T14:00Z	GET	/api/v1/user/profile	50ms	200 OK
2023-10-15T14:01Z	POST	/api/v1/user/update	75ms	201 Created
```

**Code:**

```python
# Example of a simple script for replaying production traffic in staging
import requests
from datetime import datetime, timedelta

def replay_production_traffic(start_time, end_time):
    url = "https://staging-gateway.example.com/replay"
    headers = {
        "X-Start-Time": str(start_time.timestamp()),
        "X-End-Time": str(end_time.timestamp())
    }
    response = requests.post(url, headers=headers)
    return response.json()

# Example usage
start_time = datetime.now() - timedelta(hours=12)
end_time = datetime.now()
replay_production_traffic(start_time, end_time)
```

**Example input / output:**

```text
Input:
- Start Time: 2023-10-15T14:00:00Z
- End Time: 2023-10-15T14:30:00Z

Output:
{
  "replayedRequests": [
    {
      "timestamp": "2023-10-15T14:00:00Z",
      "requestMethod": "GET",
      "requestPath": "/api/v1/user/profile",
      "responseTime": 50,
      "status": 200
    },
    {
      "timestamp": "2023-10-15T14:01:00Z",
      "requestMethod": "POST",
      "requestPath": "/api/v1/user/update",
      "responseTime": 75,
      "status": 201
    }
  ],
  "validationResults": {
    "passed": true,
    "reasons": [
      "All requests were successfully replayed.",
      "No unexpected errors or timeouts occurred."
    ]
  }
}
```

**Data:**

```json
{
  "productionTraffic": [
    {
      "timestamp": "2023-10-15T14:00:00Z",
      "requestMethod": "GET",
      "requestPath": "/api/v1/user/profile",
      "responseTime": 50,
      "status": 200
    },
    {
      "timestamp": "2023-10-15T14:01:00Z",
      "requestMethod": "POST",
      "requestPath": "/api/v1/user/update",
      "responseTime": 75,
      "status": 201
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is Production Traffic Available?
7. Replay Selected ProductionTraffic
8. Monitor and Log Results
9. Use Simulated Traffic
10. Perform Automated Testing
11. Analyze Validation Results
12. Is Everything Ok?
13. Proceed to Full Rollout
14. Fix Issues and RepeatValidation
15. End

