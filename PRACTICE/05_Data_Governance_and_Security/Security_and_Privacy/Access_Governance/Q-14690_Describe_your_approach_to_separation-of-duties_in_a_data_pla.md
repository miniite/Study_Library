---
type: "practice"
difficulty: "Expert"
---

# Describe your approach to separation-of-duties in a data platform so that no single person can both grant themselves access and delete the audit record of doing so — down to the IAM and pipeline level.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary Overview

In this scenario, we need to implement separation-of-duties (SoD) in a data platform to ensure that no single individual can both grant themselves access and delete any audit records of such actions. This involves careful design at the Identity and Access Management (IAM) level and within pipeline workflows.

#### Triage
Understanding the Question

The interviewer is probing our understanding of SoD principles, particularly in the context of IAM policies and pipeline configurations. The key assumptions are that we need to prevent a user from having both administrative privileges and the ability to modify audit logs. We must also consider how this can be enforced across various components such as data pipelines and storage systems.

#### First-Principles Solution

IAM Policy Design:

Define clear roles with specific permissions.
Ensure that granting access (e.g., creating policies or users) is restricted to a different set of roles than those who can modify audit logs.

Pipeline Configuration:

Implement checks and balances in ETL (Extract, Transform, Load) pipelines to enforce SoD.
Use event-driven architectures where actions are logged before they occur, ensuring that changes cannot be backdated or undone after the fact.

Audit Log Management:

Ensure that audit logs are immutable once created, preventing any modifications.
Use external services or specialized log management tools to store and verify these logs.

Cross-Platform Security Controls:

Integrate with a centralized security platform for real-time monitoring and enforcement.
Employ multi-factor authentication (MFA) where appropriate to enhance security.

#### Implementation
Steps

Define IAM Roles and Permissions:

Access Grantors Role: Limited permissions to create, modify, or delete users, roles, and policies.
Audit Managers Role: Full access to view, create, update, and delete audit logs but no ability to grant or revoke other users' access.

Pipeline Design:

Use a workflow engine like Apache Airflow or AWS Step Functions to manage pipeline tasks.
For each data transformation task, generate an event that triggers logging before the action is executed.
User Action
Log Event
Pipeline Task
Audit Log Storage
Monitoring Service

Immutable Audit Logs:

Store audit logs in a separate, immutable storage layer such as AWS CloudWatch or Azure Log Analytics.
User Action
Log Event
Audit Log Storage (Immutable)

Real-Time Monitoring and Alerts:

Set up alerts to notify when any SoD violations are detected.
PYTHON
def monitor_audit_logs(logs):
    for log in logs:
        if is_sod_violation(log):
            send_alert(log)

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing these measures, we ensure that no single individual can both grant themselves access and delete audit records. This approach leverages IAM policies, pipeline design, immutable storage, and real-time monitoring to enforce separation of duties effectively.

Common Pitfalls:

Not having clear role definitions and permission boundaries.
Failing to integrate external monitoring tools for real-time alerts.
Ignoring the need for immutable log storage, which can be compromised if logs are mutable.

For follow-ups, we might discuss specific technologies like AWS IAM policies or Apache Airflow configurations. Additionally, we could explore how machine learning models can be used to detect potential SoD violations in large-scale environments.

**Comparison:**

| Log ID | User ID     | Action        | Target Role      | Timestamp           |
|--------|-------------|---------------|------------------|--------------------|
| 100    | admin       | grant_access  | data_analyst     | 2023-10-01T14:00Z   |
| 101    | auditor     | delete_audit_log |              | 2023-10-01T15:00Z   |

**Code:**

```python
def is_sod_violation(log):
    return log.action == 'grant_access' and log.target_role == 'audit_manager'
```

**Example input / output:**

```text
log_event = {"user_id": "admin", "action": "grant_access", "target_role": "data_analyst"}
audit_log_storage = {
    "logs": [
        {"user_id": "admin", "action": "grant_access", "target_role": "data_analyst"},
        {"user_id": "auditor", "action": "delete_audit_log", "timestamp": "2023-10-01T14:00Z"}
    ]
}
is_sod_violation(log_event)  # Returns False

audit_logs = monitor_audit_logs(audit_log_storage)
```

**Data:**

```json
{
  "logs": [
    {
      "user_id": "admin",
      "action": "grant_access",
      "target_role": "data_analyst"
    },
    {
      "user_id": "auditor",
      "action": "delete_audit_log",
      "timestamp": "2023-10-01T14:00Z"
    }
  ]
}
```

**Flow:**

1. User Action
2. Log Event
3. Pipeline Task
4. Audit Log Storage
5. Monitoring Service

