---
type: "practice"
difficulty: "Deep"
---

# Your ALB health checks pass but real users get 5xx. How can health-check config and target-group settings mask a real routing/networking failure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DNS Failover](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where ALB (Application Load Balancer) health checks are passing but real users encounter 5xx errors, we need to dive into the configuration details of the target group and its associated load balancer settings. The core idea is that a misconfigured health check can mask underlying network or application-level issues.

#### Triage

The problem at hand involves understanding how ALB’s health checks work and ensuring they accurately reflect the state of your backend services. Health checks are crucial for monitoring service availability, but if improperly configured, they can lead to users experiencing errors while the load balancer believes everything is fine.

#### Assumptions
ALB Configuration: The ALB is already set up with health checks.
Target Group: The target group associated with the ALB has been defined and contains the backend instances.
5xx Errors: These errors are originating from the application layer, possibly due to issues like server crashes, resource exhaustion, or misconfigured application code.

#### First-Principles Solution

Review Health Check Settings:

Interval: The health check interval should be balanced between frequent checks (to catch issues early) and not overwhelming the backend.
Timeout: Ensure that the timeout is appropriate for your application’s response time, but also consider the trade-off with increasing false positives.
Unhealthy Threshold: This determines how many consecutive failures are required to mark an instance as unhealthy. A lower threshold can help quickly identify issues but may lead to frequent false positives.
Healthy Threshold: The number of consecutive successes needed to mark an instance as healthy.

Target Group Settings:

Port and Protocol: Ensure that the health check port and protocol match the backend services’ configuration.
Path Pattern: For HTTP/HTTPS listeners, configure a path pattern that represents the minimal viable application endpoint (e.g., /healthcheck).

Monitor Backend Instances:

Logs: Analyze logs from your backend instances to understand if there are intermittent issues causing 5xx errors.
Resource Utilization: Check CPU, memory, and other resource metrics to ensure they don’t exceed capacity.

Load Balancer Behavior:

Routing Policies: Ensure that routing policies do not unintentionally direct traffic away from unhealthy instances without marking them as such.
Listener Configuration: Verify that the listener rules are correctly configured to forward requests based on health status and other criteria.

#### Closing Summary

By thoroughly reviewing and adjusting the ALB’s health check settings, we can ensure that it accurately reflects the state of our backend services. This involves configuring intervals, timeouts, thresholds, and ensuring the target group settings align with the application layer requirements. Additionally, monitoring backend instances and load balancer behavior is crucial to identify and address root causes of 5xx errors.

#### Common Mistakes/Follow-Ups
Misconfigured Health Checks: Overly aggressive or insufficient health check intervals can lead to both false positives and negatives.
Inadequate Target Group Settings: Incorrect ports, protocols, and path patterns can cause confusion in determining real availability.
Lack of Monitoring: Failing to monitor logs and resource utilization can result in undetected issues that only manifest as 5xx errors.

**Comparison:**

| Property             | Value                |
|----------------------|---------------------|
| Health Check Protocol | HTTP                 |
| Health Check Path    | /healthcheck         |
| Interval (s)         | 30                   |
| Timeout (s)          | 5                    |
| Healthy Threshold    | 2                    |
| Unhealthy Threshold  | 3                    |

**Code:**

```python
# Example code for ALB health checks configuration (pseudo-code)
def configure_health_checks(alb_client, target_group_arn):
    response = alb_client.modify_target_group_attributes(
        TargetGroupArn=target_group_arn,
        HealthCheckProtocol='HTTP',
        HealthCheckPath='/healthcheck',
        HealthCheckIntervalSeconds=30,
        HealthCheckTimeoutSeconds=5,
        HealthyThresholdCount=2,
        UnhealthyThresholdCount=3
    )
    return response
```

**Example input / output:**

```text
Input:
- Target Group ARN: arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/abcdef123456

Output:
{
  "HealthCheckProtocol": "HTTP",
  "HealthCheckPath": "/healthcheck",
  "HealthCheckIntervalSeconds": 30,
  "HealthCheckTimeoutSeconds": 5,
  "HealthyThresholdCount": 2,
  "UnhealthyThresholdCount": 3
}
```

**Data:**

```json
{
  "HealthCheckProtocol": "HTTP",
  "HealthCheckPath": "/healthcheck",
  "HealthCheckIntervalSeconds": 30,
  "HealthCheckTimeoutSeconds": 5,
  "HealthyThresholdCount": 2,
  "UnhealthyThresholdCount": 3
}
```

**Flow:**

1. Yes
2. No
3. Review ALB Health CheckConfiguration
4. Are health checks passing?
5. Monitor backend instancesand load balancer behavior
6. Adjust health check settings;reconfigure target group
7. Verify logs and resourceutilization
8. Implement changes, test, andmonitor

