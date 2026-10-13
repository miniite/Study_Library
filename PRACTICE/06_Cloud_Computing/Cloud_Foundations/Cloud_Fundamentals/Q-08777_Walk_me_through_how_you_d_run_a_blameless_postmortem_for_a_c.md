---
type: "practice"
difficulty: "Expert"
---

# Walk me through how you'd run a blameless postmortem for a cascading multi-service outage and, more importantly, how you turn its action items into durable architectural change rather than a list nobody completes.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Running a blameless postmortem is about learning from failures to improve our systems and culture, not pointing fingers. For a cascading multi-service outage, we need to methodically analyze what went wrong, understand root causes, and translate these insights into concrete architectural changes that enhance resilience and reliability. This process involves several key steps: data collection, analysis, action item identification, prioritization, and implementation.

To ensure the postmortem’s findings lead to lasting change, it's crucial to follow a structured approach:

Data Collection: Gather comprehensive logs, monitoring data, and any relevant communication records.
Root Cause Analysis (RCA): Identify the primary causes of the outage using techniques like the 5 Whys or Fishbone Diagrams.
Impact Assessment: Understand the impact on users and other services to prioritize recovery efforts.
Action Item Identification: Derive actionable steps from the RCA, ensuring they are specific, measurable, achievable, relevant, and time-bound (SMART).
Stakeholder Involvement: Engage with cross-functional teams including developers, operations, security, and management to ensure all perspectives are considered.

#### Implementation
Plan: Develop a detailed plan for implementing the action items, involving resources and timelines.
Communication Strategy: Ensure that the findings and actions taken are communicated transparently across the organization.

Let's dive into each step in more detail:

Data Collection

Collect logs from the affected services, infrastructure monitoring data (e.g., AWS CloudWatch), and any communication platforms used during the outage (Slack, Jira, etc.). This data is crucial for understanding how and why the failure occurred.

Log Collection
Infrastructure Monitoring
Communication Records
Root Cause Analysis (RCA)

Use RCA techniques to identify the root causes. The 5 Whys method involves repeatedly asking "why" until reaching a fundamental issue, while Fishbone Diagrams help visualize possible causes and their relationships.

Outage
Component Failure
System Dependency Issues
Configuration Errors
Impact Assessment

Assess the impact on users (e.g., service downtime, data loss) and other services. This helps prioritize recovery efforts and understand the broader implications.

Service Downtime
System Availability
User Satisfaction
Action Item Identification

Derive specific action items from the RCA. Ensure these are SMART:

Specific: Clearly define what needs to be done.
Measurable: Include metrics for success.
Achievable: Consider resource availability and timelines.
Relevant: Align with organizational goals.
Time-bound: Set deadlines.
Stakeholder Involvement

Engage cross-functional teams in the postmortem process. Developers, operations, security, and management should all contribute to ensure a well-rounded understanding of the issue.

Postmortem Team
Developers
Operations
Security
Management

#### Implementation
Plan

Develop a detailed plan for implementing the action items. This includes resource allocation, timelines, and stakeholder communication.

Action Items
Resource Allocation
Timelines
Communication Plan
Communication Strategy

Communicate the findings and actions taken transparently to stakeholders. This helps build trust and ensures everyone is aligned with the changes.

Postmortem Findings
Transparency
Stakeholder Engagement

By following these steps, we can turn a postmortem into more than just a list of action items; it becomes a process for continuous improvement that strengthens our architecture and culture.

**Comparison:**

| Stakeholder | Role | Contribution |
|--------------|------|-------------|
| Developers   | Code | Identify root causes, contribute to action items |
| Operations   | Monitoring | Ensure monitoring tools are updated and maintained |
| Security     | Risk Management | Review security implications of proposed changes |
| Management   | Oversight | Provide strategic direction and resource allocation |

**Code:**

```python
# Example function to collect logs
def collect_logs(service_name):
    logs = get_service_logs(service_name)
    return logs

# Example function to perform 5 Whys analysis
def five_whys(cause, level=1):
    if level > 5:
        return f"Root cause: {cause}"
    print(f"{level}. Why is the {cause} occurring?")
    new_cause = input()
    return five_whys(new_cause, level+1)

# Example action item template
class ActionItem:
    def __init__(self, description, responsible_party, deadline):
        self.description = description
        self.responsible_party = responsible_party
        self.deadline = deadline

    def __str__(self):
        return f"{self.responsible_party} will {self.description} by {self.deadline}"
```

**Example input / output:**

| Input (Logs) | Output (Action Items) |
|--------------|----------------------|
| Logs from multiple services showing database connection issues | Action Item: Database health checks every 5 minutes; Responsible Party: DBA Team; Deadline: Next week |
| Logs indicating unhandled exceptions in frontend service | Action Item: Implement exception handling and logging improvements; Responsible Party: DevOps Team; Deadline: Within 2 weeks |

**Data:**

```json
{
  "logs": [
    {
      "service": "frontend",
      "timestamp": "2023-10-05T14:00:00Z",
      "message": "Error: Unhandled exception in request handler"
    },
    {
      "service": "database",
      "timestamp": "2023-10-06T18:00:00Z",
      "message": "Connection refused by backend service"
    }
  ]
}
```

**Flow:**

1. Postmortem Initiation
2. Data Collection
3. RCA - 5 Whys/FishboneDiagrams
4. Impact Assessment
5. Action Item Identification(SMART)
6. Stakeholder Involvement
7. Implementation Plan
8. Communication Strategy

