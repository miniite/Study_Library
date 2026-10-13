---
type: "practice"
difficulty: "Expert"
---

# A canary looks healthy on error rate and latency but the new version leaks file descriptors and dies an hour after full rollout; how would you extend your canary or post-deploy checks to catch slow-burn resource leaks?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Canary and Progressive Delivery](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Canary_and_Progressive_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a situation where a new version of our application is performing well in terms of error rate and latency during a canary deployment, but after full rollout, it starts leaking file descriptors and eventually crashes. To prevent such issues from going unnoticed, we need to extend the monitoring and post-deployment checks to ensure that not only immediate performance metrics are monitored, but also resource usage over time. This involves setting up long-running tests and health checks.

First, let's break down the question: The interviewer is probing our ability to handle complex deployment scenarios where immediate metrics might mask underlying issues. We need to consider different types of monitoring and how we can integrate them into our CI/CD pipeline to catch slow-burn resource leaks early on.

#### Assumptions

The application is deployed using a CI/CD pipeline.
Canary deployments are in place for gradual rollout.
Current monitoring tools (e.g., Prometheus, Grafana) provide error rate and latency metrics.
We have access to logs and can integrate with external services like New Relic or Dynatrace.

Step-by-step solution:

Extend Monitoring Tools:

Use existing tools like Prometheus for metric collection but add custom checks for file descriptor usage over time.
Implement long-running tests using tools like JMeter or Locust to simulate real-world scenarios and monitor resource usage.

Set Up Alerting:

Define alert thresholds based on historical data to detect gradual increases in file descriptors.
Configure alerts to notify the DevOps team as soon as a certain threshold is exceeded, even if the application appears healthy in terms of error rate and latency.

Integrate with CI/CD Pipeline:

Ensure that post-deployment checks are part of the CI/CD pipeline. This can be done using Jenkins, GitLab CI, or other continuous integration tools.
Add steps to run long-duration tests during the deployment process and monitor the results immediately after full rollout.

Automate Health Checks:

Use scripts or plugins that continuously check file descriptor usage in the production environment.

#### Cons
ider using APM (Application Performance Management) tools like New Relic or Dynatrace which can provide deeper insights into application health beyond just metrics.

Logging and Analysis:

Enhance logging to capture detailed information about when, where, and how file descriptors are being used.
Implement automated log analysis to identify patterns leading up to the crash that might not be immediately obvious.

#### Closing Summary
By extending our monitoring practices to include long-term resource usage tracking and setting up robust alerting systems, we can catch slow-burn issues like resource leaks early on. This ensures that the application remains stable and reliable during deployments. Common mistakes include relying solely on immediate metrics or ignoring gradual changes in resource usage over time.

**Comparison:**

```
Metric	Current Value (canary)	Threshold
Error Rate	0.05	<0.1
Latency	300ms	<= 500ms
File Descriptors	<100	<200
```

**Code:**

```python
# Example Python script to monitor file descriptors using psutil library
import psutil
from datetime import datetime

def check_file_descriptors():
    process = psutil.Process()
    fd_count = len(process.open_files())
    timestamp = datetime.now().isoformat()

    if fd_count > 100:  # Threshold based on historical data
        print(f"Alert at {timestamp}: File descriptor count exceeded threshold - {fd_count}")

check_file_descriptors()
```

**Example input / output:**

```text
Input: Deploy new version of application with canary and full rollout.
Output:
1. Initial canary deployment shows error rate = 0.05, latency = 300ms.
2. Post-deployment checks show file descriptor count steadily increasing over 24 hours until it exceeds threshold (100).
3. Alert triggered at 17:00 with message "Alert at 2023-10-04T17:00:00Z: File descriptor count exceeded threshold - 150".
```

**Data:**

```json
{
  "thresholds": {
    "file_descriptor_limit": 100,
    "duration": "24 hours"
  },
  "metrics": [
    {"metric_name": "error_rate", "value": 0.05},
    {"metric_name": "latency", "value": 300}
  ]
}
```

**Flow:**

1. Canary OK
2. Canary Deployment
3. Deploy New Version
4. Full Rollout
5. Monitor Metrics
6. Check for Resource Leaks
7. Alert if Threshold Exceeded
8. Post-Deployment Checks
9. Notify DevOps Team
10. Long-running Tests

