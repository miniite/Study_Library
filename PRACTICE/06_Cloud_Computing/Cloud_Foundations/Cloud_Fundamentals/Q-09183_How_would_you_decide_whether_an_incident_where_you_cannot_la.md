---
type: "practice"
difficulty: "Deep"
---

# How would you decide whether an incident where you cannot launch new instances (provider control-plane degraded) but running instances are healthy changes your response versus a normal outage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Control Plane Failures](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Control_Plane_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with the scenario of being unable to launch new instances due to a provider control-plane degradation, while existing instances remain healthy and functional, the decision on how to respond hinges on several key factors—primarily the nature and scope of the outage, potential risks, and service-level objectives (SLOs). This scenario requires a nuanced understanding of cloud fundamentals and operational principles.

Firstly, let's triage the situation: The core issue here is that our ability to scale or replace instances has been compromised. However, since existing instances are still functioning as expected, this does not immediately impact ongoing service delivery. But it significantly limits our ability to handle increasing loads or recover from failures proactively.

Reasoning Step-by-Step

Assess the Outage Scope:

Identify if the control-plane degradation is isolated to a specific region or across all regions.
Determine whether this outage affects only certain instance types or all.

Evaluate Risk and Impact:

#### Cons
ider how long the outage has persisted and its likelihood of resolving soon.
Assess the impact on service availability, user experience, and business operations.
Evaluate if the current operational workloads can be sustained until a resolution is found.

Risk Management and SLIs/SLOs:

Review the relevant Service Level Indicators (SLIs) and Service Level Objectives (SLOs).
Determine whether the outage will cause any service degradation that exceeds our agreed-upon SLOs.
Analyze if there are any regulatory or compliance implications.

Mitigation Strategies:

Develop short-term mitigation plans such as resizing existing instances, leveraging auto-scaling groups with appropriate buffer policies.
Explore alternative compute resources within the same cloud provider (e.g., different regions) or external providers to maintain availability.
Plan for long-term solutions like implementing more robust control-plane failover mechanisms.

Communication and Escalation:

Communicate with stakeholders, including internal teams and customers, about the current situation and steps being taken.
Escalate the issue to the provider if necessary, ensuring we are part of the resolution process.
Full Solution

Given that the existing instances remain healthy but new launches are impossible, our primary focus should be on maintaining operational continuity rather than initiating new deployments. This involves:

Proactive Monitoring: Continuously monitor the health and performance metrics of the running instances to catch any early signs of failure.
Resource Optimization: Utilize existing resources efficiently by adjusting instance sizes or distribution based on load patterns.
Plan for Future Incidents: Develop and test disaster recovery plans that factor in control-plane degradation, ensuring you have a fallback strategy.

#### Closing Summary

In summary, while the current outage does not immediately disrupt service delivery, it severely limits our ability to scale or respond to new demands. Our response should focus on maintaining operational continuity, optimizing resource usage, and preparing for potential future incidents. Proactive monitoring, strategic planning, and clear communication are key components of this approach.

#### Common mistakes
include over-reliance on the current healthy state leading to neglect of proactive measures, or failing to communicate effectively with stakeholders about the limitations imposed by the outage.

**Comparison:**

```
Instance ID	Region	Health Status
i-0123456789abcdef0	us-west-2	OK
i-0f0e9d8c7b6a5948	us-west-2	OK
```

**Code:**

```python
# Example Python script for monitoring instance health
import boto3

def check_instance_health(region):
    ec2 = boto3.client('ec2', region_name=region)
    response = ec2.describe_instances()
    # Logic to parse and monitor instance health metrics
```

**Example input / output:**

```text
Input:
- Region: us-west-2

Output (Dry Run):
- Checked instance i-0123456789abcdef0 in region us-west-2, health status OK.
- Checked instance i-0f0e9d8c7b6a5948 in region us-west-2, health status OK.
```

**Data:**

```json
{
  "instance_ids": ["i-0123456789abcdef0", "i-0f0e9d8c7b6a5948"],
  "region": "us-west-2",
  "health_check_interval": 5,
  "health_threshold": 3
}
```

**Flow:**

1. Yes
2. No
3. Check Existing Instances
4. Is Health Check Passing?
5. Monitor and Optimize
6. Report Issue, Adjust Resources
7. Monitor for Long-TermSolutions
8. Evaluate Mitigation Strategies

