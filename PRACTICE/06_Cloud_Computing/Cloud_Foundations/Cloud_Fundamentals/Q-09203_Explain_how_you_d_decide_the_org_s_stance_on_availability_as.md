---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd decide the org's stance on 'availability as a shared responsibility with the provider', so teams stop treating a provider SLA credit as though it were their own uptime commitment to customers.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding the organization’s stance on "availability as a shared responsibility with the provider," we need to ensure that the DevOps team understands and accepts its role in maintaining system availability while leveraging the provider's SLA (Service Level Agreement) commitment. This involves several key steps, from understanding the SLAs to implementing best practices, monitoring, and continuous improvement.

#### Triaging the Question

The core of this question revolves around clarifying the mutual responsibility between the organization and the cloud service provider in ensuring system availability. The interviewer is probing your ability to articulate a balanced approach that leverages the strengths of both parties while avoiding the pitfalls of over-reliance on SLAs as if they were an uptime commitment.

#### Assumptions
Provider’s SLA: The provider has committed to maintaining a certain level of service availability.
Team’s Uptime Commitment: The DevOps team is responsible for managing application dependencies, configurations, and user experiences that impact overall system availability.
Cost Considerations: Minimizing costs while ensuring robust services.

#### First-Principles Solution
Step 1: Understand the SLA

First, we need to review and understand the provider's SLA in detail. This includes:

Defining the service levels (uptime guarantees).
Identifying the scope of the SLA.
Understanding what is excluded from the SLA.

For example, a typical AWS SLA might state 99.95% uptime for EC2 instances but exclude downtime due to maintenance or network issues.

Step 2: Internal Service Level Objectives (SLOs)

Next, we define internal SLOs that align with the provider’s SLA but also take into account the specific needs of our organization and services. For instance:

Setting a target of 99.5% uptime for mission-critical applications.
Identifying the acceptable downtime per week or month.
Step 3: Design Resilient Infrastructure

The DevOps team must design resilient infrastructure that can handle potential failures gracefully. This involves:

Implementing multi-region deployments to ensure redundancy.
Setting up health checks and auto-scaling policies.
Using monitoring tools to detect and respond to issues proactively.
Step 4: Monitoring and Alerting

Continuous monitoring is crucial. We need to set up robust alerting systems that notify the DevOps team about any issues before they affect customers. Tools like Prometheus, Grafana, or CloudWatch can be used for real-time monitoring.

Step 5: Incident Management

Develop a comprehensive incident management plan that includes:

Clear communication protocols.
Roles and responsibilities.
Post-mortem analysis processes to continuously improve.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the organization should adopt an approach where the DevOps team focuses on internal controls and resilience while leveraging the provider’s SLAs for outages beyond their control. This ensures that the team remains proactive in maintaining service availability without over-relying solely on external guarantees.

#### Common mistakes
include:

Treating the provider's SLA as a direct commitment to end-users, which can lead to complacency.
Failing to implement necessary internal controls and monitoring.
Not regularly reviewing and updating SLOs based on changing business needs.

#### Follow-ups
might involve discussing specific tools or technologies used for monitoring and incident response, or exploring real-world scenarios where this approach has been successfully implemented.

**Comparison:**

```
SLA	Internal SLO
99.95% Uptime EC2 instances	99.5% Uptime Mission-Critical Applications
```

**Code:**

```python
# Example code snippet for setting up basic alerts using Prometheus and Grafana

def setup_prometheus_alerts():
    # Define a rule to alert on high CPU usage
    rules = [
        {
            "alert": "HighCPUUsage",
            "expr": 'avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) < 0.2',
            "for": "1m",
            "labels": {"severity": "critical"},
            "annotations": {"summary": "High CPU usage detected on instance {{ $labels.instance }}"}
        }
    ]
    # Implement this rule in the Prometheus configuration
```

**Example input / output:**

```text
Input: Monitoring data indicating a high CPU usage on node instance-123.
Output: Alert sent to DevOps team via Grafana dashboard and email notification.

Request: User reports application downtime.
Response: Incident logged in Jira, assigned to on-call engineer. Engineer begins troubleshooting.
```

**Data:**

```yaml
# Example YAML for an SLO dashboard

slos:
  - name: MissionCriticalAppUptime
    target: 99.5%
    scope: application
    metric: application_uptime_percentage
    thresholds:
      warning: 98.0%
      critical: 96.0%

monitoring-tools:
  - tool: Prometheus
    metrics:
      - node_cpu_seconds_total
      - application_uptime_percentage

alert-rules:
  - rule: HighCPUUsage
    expr: 'avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) < 0.2'
    for: "1m"
```

**Flow:**

1. Review Provider's SLA
2. Identify Exclusions
3. Define Internal SLOs
4. Design Resilient Infrastructure
5. Implement Monitoring andAlerting
6. Develop Incident ManagementPlan
7. Continuous Improvement

