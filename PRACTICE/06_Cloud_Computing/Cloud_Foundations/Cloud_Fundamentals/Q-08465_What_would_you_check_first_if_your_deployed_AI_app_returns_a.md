---
type: "practice"
difficulty: "Basic"
---

# What would you check first if your deployed AI app returns a 502 or 503 error right after a new release?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Troubleshooting 5xx Errors](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Troubleshooting_5xx_Errors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an AI application deployed in the cloud returns a 502 Bad Gateway or 503 Service Unavailable error immediately after a new release, it typically indicates issues with the infrastructure, configuration, or deployment process rather than the application itself. The first steps should focus on verifying network and service health to ensure that the issue is not due to misconfiguration or temporary outages.

Triaging the Issue

The question asks us to identify what checks would be most critical in such a scenario. An important aspect to consider is whether these HTTP status codes are indicative of an application error (502) or a server overload (503). A 502 error suggests that there's a problem with one of the servers handling the request, while a 503 could mean that the server is overloaded or temporarily unable to handle requests.

#### Assumptions
Application Code: We assume the application code has been tested and validated locally.
Environment Consistency: The deployed environment (e.g., cloud infrastructure) should match local testing environments as closely as possible.
Monitoring Tools: Basic monitoring tools like logs, metrics, and network health checks are available.

#### First-Principles Solution
Network Connectivity and DNS Resolution:
Check if the domain name resolution is correct.
Ensure there's no network partitioning or DNS changes affecting the application.
Cloud Load Balancer Health:
Verify that the load balancer is correctly configured and healthy.
Check for any unexpected redirects or rules that might be causing issues.
Service Dependencies:
If your app relies on other services, ensure those dependencies are also available and responding as expected.
Instance State and Configuration:
Validate the state of the cloud instances to see if they're up and running.
Ensure all environment variables and configuration settings have been correctly applied post-deployment.
Resource Limits and Scalability:
Check CPU, memory, disk I/O, and network limits on your instances.
If auto-scaling is enabled, ensure it's properly configured to handle the load.
Security Groups and Network ACLs:
Review security groups and network access control lists to confirm no unexpected blocking or restrictions.

#### Closing Summary
+ Common Mistakes

In summary, when encountering a 502 or 503 error post-release, start by checking for fundamental issues like network connectivity, load balancer health, service dependencies, instance states, resource constraints, and security configurations. These checks will help you quickly identify whether the issue lies in infrastructure misconfiguration rather than application bugs.

#### Common mistakes
include skipping basic validation steps (like DNS resolution or instance state) and assuming that application-level errors are the only possible cause for these HTTP status codes.

#### Triage
Checklist
DNS: Resolve domain name.
Load Balancer: Verify configuration and health.
Dependencies: Check connectivity to other services.
Instances: Ensure correct state and configurations.
Resources: Monitor CPU, memory, disk I/O, and network limits.
Security: Review security groups and ACLs.

**Comparison:**

```
Check	Status
Network Connectivity	Healthy
Load Balancer Health	Healthy with no alarms
Service Dependencies	Accessible and responding as expected
Instance State	Running
Resource Limits	Within limits
Security Groups & ACLs	No restrictions
```

**Code:**

```python
# Example Python code for checking instance state using Boto3 (AWS SDK)
import boto3

ec2 = boto3.resource('ec2')
instance_ids = ['i-0123456789abcdef0', 'i-0234567890abcdef1']

for instance_id in instance_ids:
    instance = ec2.Instance(instance_id)
    state = instance.state['Name']
    print(f"Instance {instance.id} is currently {state}.")
```

**Example input / output:**

```text
Input:
- Domain name: myapp.example.com
- Load Balancer State: Healthy with no alarms

Output:
- DNS resolution for domain_name is successful.
- All instances (i-0123456789abcdef0, i-0234567890abcdef1) are in a 'running' state.
- Service dependencies are accessible and responding as expected.
```

**Data:**

```json
{
  "domain_name": "myapp.example.com",
  "load_balancer_state": {
    "healthy": true,
    "alarms": []
  },
  "service_dependencies": [
    "database.example.com:3306",
    "cache.example.com:11211"
  ],
  "instance_ids": ["i-0123456789abcdef0", "i-0234567890abcdef1"]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Yes
5. Check DNS Resolution
6. DNS Resolves
7. Check Load Balancer Health
8. Load Balancer Healthy
9. Check Service Dependencies
10. Dependencies Accessible
11. Check Instance State
12. Instances Running
13. Monitor Resource Limits
14. Within Limits?
15. Review Security Groups &ACLs
16. No Restrictions?
17. Troubleshooting Complete
18. Scale Resources

