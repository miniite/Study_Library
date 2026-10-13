---
type: "practice"
difficulty: "Intermediate"
---

# You want the gateway to emit an audit log of every admin config change for compliance; how do you architect this so the audit trail is tamper-evident and survives a gateway restart or redeploy?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Idempotency and Org Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Idempotency_and_Org_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: To ensure that an API Gateway logs all admin configuration changes in a tamper-evident manner, we need to design a system that captures these events at the source (the admin UI or configuration management tool) and stores them in a secure, immutable log. This involves leveraging event sourcing and perhaps integrating with a blockchain for additional security. The solution should also handle gateway restarts and redeployments gracefully.

Question Triage: The interviewer is probing your understanding of logging mechanisms, tamper-evident design principles, and the robustness of the system under deployment changes. We will assume that "admin config change" refers to modifications made through a UI or API by authorized users, such as updating routes, modifying authentication settings, etc.

Step-by-Step Solution:

Event Sourcing: Implement an event-sourced architecture where every admin configuration change generates an immutable log entry. This can be achieved using technologies like AWS CloudTrail for AWS services or custom logging mechanisms with a database that supports ACID transactions and immutability.

Blockchain Integration (Optional): For enhanced security, integrate the audit logs into a blockchain network. Blockchain provides a tamper-evident ledger where every transaction is timestamped and linked to the previous block, ensuring data integrity.

Audit Log Storage: Store these log entries in a secure, immutable storage layer such as an S3 bucket with versioning enabled (AWS), a dedicated database table with row-level locks, or even a blockchain node. Ensure that this storage is accessible only by trusted entities and is configured to maintain the logs indefinitely.

Gateway Configuration Changes: For changes made directly in the API Gateway console, use CloudTrail or AWS Config to automatically capture these events and store them in the designated audit log system.

Handling Gateway Restart/Redeploy: Use a sidecar container approach with Kubernetes to run an audit log collector alongside the API Gateway. This ensures that even if the gateway needs to be restarted or redeployed, the logs will not be lost because they are stored externally.

Monitoring and Alerting: Implement monitoring for the health of both the API Gateway and the audit logging system. Set up alerts to notify the team if any part of this system experiences downtime or data loss.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key is to ensure that the logs are immutable and accessible only by trusted entities.
A common mistake is relying solely on in-memory caches for logging, which can lead to data loss during restarts.
Another pitfall is not properly securing the audit log storage, leading to unauthorized access or tampering.

#### Follow-ups
might include discussions on how to optimize performance, scaling considerations, and potential integration with other security tools like SIEM systems.

**Comparison:**

```
Component	Description
Event Sourcing	Captures every admin config change as an immutable log entry.
Blockchain Integration	Provides enhanced security by ensuring data integrity through blockchain nodes.
Secure Storage	Stores logs in a versioned S3 bucket or dedicated database table.
```

**Code:**

```python
# Example of capturing admin config changes using AWS CloudTrail
import boto3

def capture_config_change(event):
    cloudtrail = boto3.client('cloudtrail')
    response = cloudtrail.put_event_selectors(
        TrailName='MyApiGatewayAuditTrail',
        EventSelectors=[
            {
                'ReadWriteType': 'All',
                'DataResources': [
                    {
                        'Type': 'AWS::ApiGateway::RestApi',
                        'Values': ['*']
                    }
                ]
            }
        ]
    )
```

**Example input / output:**

```text
Input: Admin changes authentication type for route 1.
Output: AuditTrailLog created with timestamp, event type, user agent, and action details.
```

**Data:**

```json
{
  "AuditTrailLog": {
    "Timestamp": "2023-10-05T14:48:00Z",
    "EventType": "AdminConfigurationChange",
    "UserAgent": "admin-ui/1.0",
    "ActionDetails": {
      "ChangedFields": ["auth_type", "route_1"]
    }
  }
}
```

**Flow:**

1. Admin UI/API
2. Event Generation
3. Event Sourced Logs
4. Audit Trail Log Storage (S3,Database)
5. Kubernetes Sidecar forMonitoring
6. CloudTrail/AWS ConfigCapture

