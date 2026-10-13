---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design an incident-response and on-call architecture for a system-of-systems where no single team understands the whole graph. Cover runbooks vs automation, blast-radius-aware alerting, and how you avoid alert fatigue while ensuring the correlated multi-service incident actually pages the right owner.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant vs Dedicated Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Multi-tenant_vs_Dedicated_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an incident-response architecture for a system-of-systems with distributed ownership, the key is to ensure that each service is self-aware and can respond effectively within its domain while also triggering broader coordination where necessary. This involves balancing runbooks with automation, implementing blast-radius-aware alerting, and managing alert fatigue while ensuring comprehensive coverage.

To break this down step-by-step:

Runbooks vs Automation: Runbooks provide detailed, manual steps for complex incidents that can't be fully automated. They are essential for understanding the entire system's behavior. However, automation is crucial for routine issues, reducing human intervention and speeding up resolution.
Blast-Radius-Aware Alerting: This ensures that alerts are contextually relevant to their scope of impact, preventing unnecessary paging and alert fatigue.
Avoiding Alert Fatigue: By optimizing the alerting system to minimize false positives and prioritize critical alerts, we can ensure that the correct team is paged for each incident.

Let's delve into these concepts in more detail:

Runbooks vs Automation

Runbooks are comprehensive, documented steps used by engineers when they need to address complex issues. They provide a detailed roadmap of actions required to resolve an issue, which is crucial for situations that require human judgment and expertise. For example, a database outage might necessitate a runbook that includes specific steps like checking logs, restarting services, or reaching out to hardware vendors.

Automation, on the other hand, is about using tools and scripts to handle routine tasks quickly and efficiently. This can include automated health checks, log monitoring, and simple error recovery mechanisms. For instance, if an application experiences a timeout, a script might automatically restart it without requiring human intervention.

Blast-Radius-Aware Alerting

Blast-radius refers to the area of impact when something goes wrong in one part of the system. By understanding this, we can design alerts that are relevant only for services or components within the affected area. For example, if a database outage occurs, an alert might be paged only to teams responsible for that specific database service.

#### Example Scenario
Database Outage: An alert is triggered when the primary database node fails. The blast-radius analysis shows it affects two microservices: PaymentService and OrderProcessingService. A runbook would then kick in, but a well-defined alerting system ensures only these teams are paged.
Network Disruption: If there's an issue with the network connection to external services, alerts might be sent to teams responsible for network management and integration.
Avoiding Alert Fatigue

Alert fatigue occurs when too many irrelevant or false-positive alerts lead to desensitization. To avoid this:

Prioritize Critical Alerts: Implement a severity-based alert system where critical issues are flagged with high priority.
Use Metrics and Thresholds Wisely: Set thresholds that accurately capture real issues, avoiding unnecessary alerts for minor fluctuations.
Contextual Alerts: Ensure that each alert provides enough context to determine its importance without requiring additional investigation.
Full Solution

To implement this architecture effectively:

Define Service Boundaries: Clearly define the responsibilities and boundaries of each service within the system-of-systems.
Automated Health Checks: Implement automated health checks for common issues, such as timeouts or resource limits.
Runbook Mechanism: Have a centralized repository of runbooks for complex incidents that require manual intervention.
Contextual Alerting System: Develop an alerting framework that uses blast-radius analysis to determine which teams should be paged based on the incident's impact.
Alert Fatigue Mitigation: Use severity levels and thresholds to ensure only critical alerts are sent, reducing false positives.

#### Common Mistakes
and Follow-Ups
Mistake 1: Over-reliance on Automation: Not having a runbook for complex incidents can lead to missing critical steps.
Mistake 2: Underestimating Blast Radius: Failing to account for the impact of an incident can result in paging teams unnecessarily.

#### Follow-Up
Questions:
How would you handle a cascading failure scenario?
What tools and technologies do you recommend for implementing this architecture?
Can you walk through how you'd prioritize alerts during a high-impact event?

**Comparison:**

```
Service	Dependencies	Alerts Triggered On Failure
PaymentService	OrderProcessingService	Yes
OrderProcessingService	PaymentService	Yes
OtherServices	None	No
```

**Code:**

```python
# Example of a basic alerting function in Python
def send_alert(service, severity):
    if severity >= 5:  # High priority threshold
        teams = get_teams_for_service(service)
        for team in teams:
            page_team(team)

def get_teams_for_service(service):
    return {
        "PaymentService": ["team1", "team2"],
        "OrderProcessingService": ["team3"]
    }.get(service, [])
```

**Example input / output:**

```text
Input: Database node failure (primary node)
Output:
- Alert sent to teams responsible for `PaymentService` and `OrderProcessingService`
- No alert sent to other unrelated services

Input: Network disruption affecting external service
Output:
- Alert sent to network management team and integration team
- No alert sent to application development or database teams
```

**Data:**

```yaml
# Example configuration for blast-radius analysis
services:
  - name: PaymentService
    dependencies:
      - OrderProcessingService
  - name: OrderProcessingService
    dependencies:
      - PaymentService

thresholds:
  timeout_threshold: 30s
  memory_limit: 512MB
```

**Flow:**

1. Database Node Failure
2. PaymentService Team
3. OrderProcessingService Team
4. No other teams paged
5. Network Disruption
6. Network Management Team
7. Integration Team
8. No other teams paged

