---
type: "practice"
difficulty: "Intermediate"
---

# You see 5xx errors from the app but the load balancer's target group reports every target healthy. Where do you look next and what does this combination usually tell you?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Health Checks and Load Balancers](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Health_Checks_and_Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering 5xx errors from an application while all targets in a load balancer’s target group are reported as healthy, it's crucial to systematically investigate both the network and application layers. This combination usually indicates that there might be issues at the application level or within the infrastructure itself, such as misconfigurations or resource constraints.

#### Triage
Understanding the Question: The question is probing whether you can identify where a discrepancy exists between the load balancer's health checks (indicating all targets are healthy) and the 5xx errors from the application.

#### Assumptions
The environment is running on cloud infrastructure, e.g., AWS or GCP.
Load balancers (e.g., Application Load Balancer in AWS) are used to route traffic.
There is a target group associated with the load balancer.
Step-by-Step Reasoning:
Check Health Checks: Ensure that the health checks configured on the load balancer match what the application expects.
Inspect Application Logs: Look for errors in application logs to pinpoint where the issue occurs.
Network Configuration: Verify network settings and firewall rules.
Resource Constraints: Check if the application is facing resource constraints like CPU or memory limits.
Load Balancer Settings: Ensure the load balancer is configured correctly, including sticky sessions (if used).
Database Health: If the application interacts with a database, check for any issues there.

#### Solution
Verify Health Check Configuration:
The health checks might not be accurately capturing the state of your application. For example, they might only be checking HTTP status codes and not the actual application logic.
Examine Application Logs:
Use a tool like aws cloudwatch logs or equivalent to gather and analyze logs from the application instances.
Network Configuration:
Check if any network policies or firewalls are blocking traffic to specific endpoints within the application.
Resource Constraints:
If you're using containerized applications, ensure that resource limits (CPU, memory) are not being hit.
Load Balancer Settings:
Review the settings of the load balancer to ensure it is distributing traffic correctly and not causing any bottlenecks.
Database Health:
If your application uses a database, check for issues like connection timeouts or performance degradation.

#### Closing Summary

By methodically investigating each layer (network, application, resources), you can identify the root cause of 5xx errors even when the health checks report all targets as healthy. Common mistakes include not checking application logs thoroughly and assuming that network or resource constraints are the only issues to look for.

#### Common Mistakes/Follow-Ups
Not investigating logs: Simply relying on health check reports without looking into detailed application logs.
Ignoring database performance: Assuming that if everything looks fine on the surface, there must be another problem elsewhere.
Overlooking misconfigurations: Failing to check configurations in the load balancer or network settings.

**Comparison:**

```
Step	Action
1	Verify health check configuration.
2	Examine application logs for errors.
3	Check network policies and firewalls.
4	Ensure resource limits are not being hit.
5	Review load balancer settings.
6	Check database performance if applicable.
```

**Code:**

```python
# Example Python script for fetching logs from AWS CloudWatch Logs
import boto3

client = boto3.client('logs')
response = client.get_log_events(logGroupName='your-log-group-name',
                                 logStreamName='your-log-stream-name')
for event in response['events']:
    print(event['message'])
```

**Example input / output:**

```text
Input:
aws logs get-log-events --log-group-name my-app-logs --log-stream-name prod-deploy-2023-10-01

Output:
{
  "events": [
    {
      "message": "500 Internal Server Error at /path/to/resource",
      "timestamp": 1698400000
    },
    ...
  ]
}
```

**Data:**

```json
{
  "logGroup": "my-app-logs",
  "streamNames": [
    "prod-deploy-2023-10-01"
  ]
}
```

**Flow:**

1. Check Health Checks
2. Verify Configuration
3. Examine Application Logs
4. Check NetworkPolicies/Firewalls
5. Ensure Resource Limits Not Hit
6. Review Load Balancer Settings
7. Check Database Performance(if applicable)

