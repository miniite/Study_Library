---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design the org-wide policy for cost-anomaly response ownership: should the anomaly page the owning team, the FinOps team, or a central SRE-style cost-on-call rotation? Argue the routing model for a 500-team org and the escalation path when the owner won't act.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Usage-based Cost Surprises](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Usage-based_Cost_Surprises.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

Designing an effective org-wide policy for responding to cost anomalies is crucial in managing cloud costs across multiple teams. For a 500-team organization, we need to balance responsiveness and accountability by determining which entity (owning team, FinOps team, or SRE-style cost-on-call rotation) should be responsible for handling cost anomalies. This design must ensure timely actions are taken while maintaining clear lines of communication and escalation paths.

#### Triaging the Question

The question is probing into understanding how to implement a robust response system for cost anomalies that ensures timely and effective action across a large organization. We need to consider several factors:

Ownership: Who should be responsible for investigating and addressing cost anomalies?
Escalation Path: What happens when the initial owner fails to act within a given timeframe?
Scalability: How does this model scale as the number of teams increases?

#### First Principles Solution

To design an effective policy, we need to break down the problem into smaller components and build up a solution from first principles.

Ownership Model
Owning Team
The owning team is typically the team that created or manages the resources generating the cost anomaly.
They have the most context about the services they operate and can quickly identify issues like misconfigurations, inefficient usage patterns, or unexpected expenses.
FinOps Team
FinOps teams are cross-functional teams responsible for managing financial aspects of cloud resources.
They have a broader perspective on cost trends across multiple teams but lack detailed knowledge of individual services.
Central SRE-Style Cost-On-Call Rotation
A dedicated team that rotates on-call responsibility can ensure round-the-clock monitoring and rapid response to anomalies.
This model balances the need for timely responses with the risk of overburdening any single team.

Given a 500-team organization, it is impractical to have each team manage their own cost anomalies. Therefore, we should rely on the owning teams as the primary responders. However, to ensure that all cost anomalies are addressed and escalated when necessary, we will implement an escalation path involving both FinOps and SRE-style rotations.

Escalation Path
Initial Response (Owning Team)
If a cost anomaly is detected in a service managed by team A, the owning team (team A) should be notified immediately via a custom alert system.
Timed Follow-Up (FinOps)
After 30 minutes to one hour without an update from the owning team, FinOps should automatically be paged and provided with details about the anomaly.
Central SRE-Style Rotation (Cost-On-Call)
If there is no action after another hour or if the FinOps team fails to resolve the issue, a central SRE-style cost-on-call rotation will take over.
Detailed Workflow
Detection and Notification
Utilize a cloud-native monitoring solution like AWS CloudWatch, Azure Monitor, or Google Stackdriver.
Set up custom alerts that notify the owning team via email, Slack, or other communication channels.
Initial Response Time
The owning team has 30 minutes to acknowledge and begin investigating the anomaly.

#### Follow-Up
(FinOps)
If no action is taken within 1 hour of initial notification, FinOps should be paged automatically via a custom alert system.
Central SRE Rotation
After another 1-2 hours, or if FinOps fails to resolve the issue, a central SRE-style rotation will step in and take over.
Resolution and Closure
The central team works with the owning team to diagnose the root cause and implement corrective actions.
Once resolved, the owning team should document the findings and share them with relevant stakeholders.

#### Closing Summary

In summary, for a 500-team organization, we recommend an ownership model where each team is responsible for their own cost anomalies. This ensures that teams remain accountable and have the necessary context to act quickly. However, to prevent issues from becoming critical, an escalation path involving FinOps and a central SRE-style rotation is essential.

#### Common Mistakes
and Follow-Ups
Ignoring Ownership: Overloading FinOps with too many responsibilities can overwhelm them.
Overly Rigid Escalation Paths: Flexibility in the escalation process can be crucial; some anomalies might require immediate attention, while others can wait a bit longer.
Lack of Communication Tools: Ensuring that communication channels are robust and reliable is critical for timely responses.

To address these concerns, we should regularly review and refine our policies based on actual incidents and feedback from teams. This iterative process will help us continuously improve the effectiveness of our cost-anomaly response system.

**Comparison:**

```
Team	Response Time (Minutes)
Owning	30
FinOps	60
SRE	90
```

**Code:**

```python
# Example code for a simple alert system using AWS Lambda and SNS
import json
from boto3 import client

def lambda_handler(event, context):
    # Get details from event (anomaly information)
    anomaly_details = event['Details']

    # Notify owning team via SNS
    sns_client = client('sns')
    response = sns_client.publish(
        TopicArn='arn:aws:sns:<region>:<account-id>:teamA-anomalies',
        Message=json.dumps({'default': json.dumps(anomaly_details)}),
        Protocol='json'
    )

    # Automatic escalation after 1 hour
    import time
    time.sleep(3600)  # Sleep for 1 hour

    # Notify FinOps team via SNS
    response = sns_client.publish(
        TopicArn='arn:aws:sns:<region>:<account-id>:finops-anomalies',
        Message=json.dumps({'default': json.dumps(anomaly_details)}),
        Protocol='json'
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Anomaly notification sent')
    }
```

**Example input / output:**

```text
# Example input for anomaly detection system
{
  "timestamp": "2023-10-01T14:30:00Z",
  "team_id": "teamA",
  "service_name": "web-app",
  "cost_anomaly": {
    "current_cost": "$500/day",
    "expected_cost": "$100/day"
  }
}

# Example output for initial response
{
  "timestamp": "2023-10-01T14:30:00Z",
  "team_id": "teamA",
  "message": "Anomaly detected in web-app service. Current cost is $500/day, expected $100/day."
}
```

**Data:**

```yaml
# Example YAML for SRE rotation schedule
rotation_schedule:
  - name: "SRE-Team-A"
    start_time: 09:00
    end_time: 17:00
  - name: "SRE-Team-B"
    start_time: 18:00
    end_time: 23:59
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Anomaly Detected
6. Owning Team Acknowledges?
7. Investigate and Resolve
8. FinOps Notified in 30 mins
9. FinOps Resolves Anomaly?
10. Anomaly Resolved
11. SRE Rotated On-Call TeamActivated
12. Investigate and Resolve withSRE Team
13. Anomaly Resolved

