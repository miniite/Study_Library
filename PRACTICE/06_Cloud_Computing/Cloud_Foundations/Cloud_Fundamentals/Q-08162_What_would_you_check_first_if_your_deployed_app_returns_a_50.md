---
type: "practice"
difficulty: "Deep"
---

# What would you check first if your deployed app returns a 502 or 503 error?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Troubleshooting 5xx Errors](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Troubleshooting_5xx_Errors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an application deployed on cloud infrastructure returns a 502 Bad Gateway or 503 Service Unavailable error, it indicates that there might be issues with either the upstream service (for example, another microservice) or the gateway itself. A deep understanding of cloud fundamentals will help you quickly diagnose and resolve these issues by following a structured approach.

#### Triage
What’s Probing

The question is probing your ability to diagnose common cloud-based deployment issues, specifically focusing on how to systematically troubleshoot errors at the gateway level or with upstream services. Key areas to consider include network connectivity, service availability, resource utilization, and configuration settings.

#### Assumptions
The application is deployed in a cloud environment (e.g., AWS, GCP, Azure).
The error is consistent and not transient.
The issue is not related to client-side requests or user behavior.

#### First-Principles Solution

Check the Gateway Health:

Verify if the gateway service itself is up and running by checking its health status in the cloud console or monitoring tools.
Review logs for any error messages or warnings that could indicate misconfigurations or underlying issues.

Network Connectivity:

Ensure there are no network issues between the gateway and the upstream services. This can be checked using tools like ping, traceroute, or cloud-provided network diagnostics.
Confirm that security groups or firewalls do not block necessary traffic paths.

Upstream Service Health:

Monitor the health of the upstream services. Use cloud-provided monitoring tools to check their availability and response times.
Check if the upstream service has adequate resources (CPU, memory) allocated and is not overloading due to high traffic or performance issues.

Resource Utilization:

Assess resource utilization metrics for both the gateway and upstream services. High CPU/Memory usage can lead to slower response times and eventually to 502/503 errors.
Check if there are any known incidents or maintenance activities on the cloud provider that might affect service availability.

Configuration and Routing:

Review configuration files for the gateway and upstream services, ensuring they align with expected behavior.
Verify routing rules to ensure traffic is correctly directed between different layers of your application stack.

Log Analysis:

Analyze logs from both the gateway and upstream services for any clues about what might be going wrong. Look for patterns or anomalies in log data that could indicate specific issues.

Backup and Failover Mechanisms:

Ensure backup and failover mechanisms are correctly configured, as these can mitigate downtime and improve overall system resilience.
Check if automatic failovers are occurring and whether they are restoring services to a healthy state.

Historical Trends:

Review historical trends in service metrics over time to identify any patterns or anomalies that could indicate an impending issue.
Use A/B testing or canary releases to introduce minor changes gradually, monitoring their impact on the system without causing disruption.

#### Closing Summary

By systematically checking these areas, you can isolate and address the root causes of 502 Bad Gateway or 503 Service Unavailable errors. The key is to leverage cloud-provided tools and services for monitoring and diagnostics while also having a deep understanding of underlying configurations and resource management principles.

#### Common Mistakes/Follow-Ups
Failing to Verify Gateway Health: Assuming the gateway itself is fine can lead to overlooking critical issues.
Ignoring Network Diagnostics: Often, network issues are overlooked but can be the root cause of 502 or 503 errors.
Not Reviewing Logs Thoroughly: Logs provide crucial insights into what might be going wrong, yet many overlook this step.
Overlooking Resource Utilization: High resource utilization is a common issue that often goes unnoticed until it causes errors.

**Comparison:**

```
Service	Status	Health Check URL
Gateway	Unhealthy	https://example-gateway.com/health
Upstream Service 1	Healthy	https://upstream-service1.com/health
Upstream Service 2	Down	http://upstream-service2.com/health
```

**Code:**

```python
# Example Python code to check service health and network status (for illustration)
import requests

def check_gateway_health(gateway_url):
    response = requests.get(f"{gateway_url}/health")
    if response.status_code != 200:
        print("Gateway is not healthy.")
```

**Example input / output:**

```

Input: check_gateway_health('https://example-gateway.com/health')

Output:

CODE
Copy
Gateway is not healthy.
```

**Data:**

```json
{
  "serviceHealthCheck": {
    "status": "down",
    "details": "Service not responding to health checks."
  }
}
```

**Flow:**

1. Check Gateway Health
2. Verify Network Connectivity
3. Monitor Upstream Services
4. Assess Resource Utilization
5. Review Configuration andRouting
6. Analyze Logs
7. Check Backup and FailoverMechanisms
8. Review Historical Trends

