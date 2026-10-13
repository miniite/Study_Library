---
type: "practice"
difficulty: "Intermediate"
---

# How do you explain to a stakeholder the difference between what an SLA promises you (a credit) and what an SLO commits your team to (a target), when they think 'we get 99.99% from the cloud so we are covered'?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the difference between Service Level Agreement (SLA) and Service Level Objective (SLO) to a stakeholder, start by clarifying that while SLAs and SLOs both ensure high availability of services, they serve different purposes and are managed differently. Specifically:

An SLA is an agreement between a service provider and its customers outlining the level of performance expected from a service, often including financial penalties if the agreed-upon targets are not met.
An SLO, on the other hand, is a commitment made by your team or organization to maintain certain levels of service quality. It does not typically involve financial penalties but ensures that you have robust processes and monitoring in place.

When stakeholders think "we get 99.99% from the cloud so we are covered," they might be referring to the cloud provider's SLA, which guarantees high uptime for their services. However, this doesn't account for your team’s own responsibilities and commitments (SLOs).

Step-by-Step Explanation

SLA with Cloud Provider:

The cloud provider promises 99.99% uptime.
If they fail to meet this target, they will provide credits or discounts as compensation.

Your Team's SLO:

Your team must commit to maintaining a certain level of service quality, which could be higher than the SLA (e.g., 99.95%).
Ensuring that your application is resilient and can handle failures within those boundaries.
Implementing monitoring and alerting systems to quickly identify issues.

#### Example Scenario

Imagine you have a web application hosted on AWS, which has an SLA of 99.99%. However, your SLO requires that the application should be available at least 99.95% of the time. If there's a failure due to your misconfiguration or issues within your code, these are not covered by the cloud provider’s SLA, and you will need to ensure those are addressed.

#### Common Mistakes
Assuming that just because the cloud provider has a high SLA, your team does not need to worry about availability.
Overlooking the importance of monitoring and proactive maintenance practices.

#### Closing Summary

In summary, while the cloud provider’s SLA ensures basic service uptime, your team's SLO covers more specific and stringent requirements for maintaining reliable operations. Both are crucial but serve different purposes in ensuring that the service is available when needed.

**Comparison:**

```
SLA vs SLO
SLA
SLO
Example SLO Metrics
Uptime
Response Time
Error Rate
```

**Code:**

```python
# Example code snippet to illustrate monitoring checks (simplified)
import requests

def check_service_availability(url):
    try:
        response = requests.get(url, timeout=5)
        if response.status_code == 200:
            return True
        else:
            return False
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")
        return False

# Example usage
url = "https://example.com/api"
if check_service_availability(url):
    print("Service is available.")
else:
    print("Service is not available.")
```

**Example input / output:**

```text
Input:
- Service URL: https://example.com/api
- Expected Status Code: 200
- Timeout Seconds: 5

Output:
- Service is available.
```

**Data:**

```json
{
  "service_url": "https://example.com/api",
  "expected_status_code": 200,
  "timeout_seconds": 5
}
```

**Flow:**

1. Cloud Provider SLA
2. Check Availability
3. 99.99% Uptime
4. No Penalty, Service Continues
5. Your Team's SLO
6. Check Availability
7. 99.95% Uptime
8. Immediate Action Required forIssues

