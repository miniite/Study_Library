---
type: "practice"
difficulty: "Deep"
---

# A change advisory board is slowing your teams to a crawl. How do you replace human CAB gating for schema changes with automated policy without increasing incident rate? Show the metrics you'd commit to.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of slow schema change approvals due to human Change Advisory Board (CAB) gating, we need to automate the process while ensuring robust policy enforcement and minimizing the risk of incidents. The key is to implement a well-defined automated governance framework that integrates with existing database management systems and leverages modern DevOps practices.

#### Triage

The core issue here is reducing manual review times for schema changes without increasing operational risks or incident rates. We need to define clear policies, automate enforcement, monitor compliance, and continuously improve the system based on feedback loops.

#### Assumptions
The database management system supports schema versioning and tracking.
There are defined governance policies for different types of schema changes (e.g., read-only, write, critical).
The team has access to a logging and monitoring infrastructure.

#### First-Principles Solution
Step 1: Define Policies

First, we need to define the policies that govern how different types of schema changes are managed:

Read-Only Changes: Can be reviewed by an automated system.
Non-Critical Write Changes: Need minimal human review and quick approval.
Critical Changes: Require thorough review and approval from key stakeholders.
Step 2: Automation Framework

Next, we'll implement the automation framework using a combination of database tools and DevOps practices:

Schema Versioning and Tracking:

Use a version control system (e.g., Git) to track schema changes.
Implement a continuous integration/continuous deployment (CI/CD) pipeline for automated testing and validation.

Automated Approval Mechanisms:

For read-only and non-critical write changes, use an approval bot that automatically approves based on predefined rules.
For critical changes, integrate the CAB into the CI/CD process with a manual approval step.

Policy Enforcement:

Use database triggers or stored procedures to enforce compliance rules during schema changes.
Implement a rule engine that checks against the defined policies before applying any changes.

Monitoring and Alerting:

Set up monitoring for critical operations and implement alerting mechanisms to notify stakeholders of potential issues.

Feedback Loops:

Regularly review incident data and policy enforcement logs to identify areas for improvement.
Use A/B testing to optimize the automation process and reduce false positives/negatives.
Step 3: Metrics

To commit to metrics, we'll track several key performance indicators (KPIs) that will help us measure success:

Approval Time: Average time from schema change request to approval.
Incident Rate: Number of incidents per million transactions.
Policy Compliance: Percentage of schema changes that comply with defined policies.
Human Interaction: Number of human interactions required for each type of schema change.

#### Closing Summary

By automating the CAB process while maintaining strict policy enforcement, we can significantly reduce manual review times and ensure high-quality database operations. The metrics outlined above will provide visibility into our progress and help us continuously improve the system.

#### Common Mistakes/Follow-Ups
Over-Reliance on Automation: Ensuring that automated systems are not too rigid and allow for human judgment when necessary.
Inadequate Policy Enforcement: Failing to implement robust policy enforcement mechanisms can lead to non-compliant changes.
Insufficient Monitoring: Not setting up adequate monitoring and alerting can result in unnoticed issues.

**Comparison:**

```
Policy Type	Approval Time (min)	Incident Rate
Read-Only Changes	1	0.02%
Non-Critical Write	5	0.03%
Critical Changes	60	0.10%
```

**Code:**

```sql
-- Example of a simple policy enforcement rule using a database trigger
CREATE TRIGGER enforce_schema_policy
BEFORE UPDATE ON my_table
FOR EACH ROW
BEGIN
    IF NEW.column1 < 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Negative values are not allowed';
    END IF;
END;
```

**Example input / output:**

```text
Input:
- A schema change request with a non-critical write operation.
Output:
- Automated approval within 5 minutes, with no manual intervention required.
- Monitoring dashboard showing the change was compliant and had no negative impact on service reliability.
```

**Data:**

```json
{
  "policies": {
    "read_only_changes": {
      "approval_time": 1,
      "incident_rate": 0.02
    },
    "non_critical_write_changes": {
      "approval_time": 5,
      "incident_rate": 0.03
    },
    "critical_changes": {
      "approval_time": 60,
      "incident_rate": 0.1
    }
  },
  "compliance_metrics": {
    "total_changes": 1200,
    "non_compliant_changes": 5,
    "compliance_percentage": 99.6%
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Schema Change Request
6. Is Read-Only?
7. Automated Approval
8. Is Critical?
9. Approval Bot Review Time
10. Automated Approval
11. MANUAL CAB Approval
12. Monitoring Dashboard

