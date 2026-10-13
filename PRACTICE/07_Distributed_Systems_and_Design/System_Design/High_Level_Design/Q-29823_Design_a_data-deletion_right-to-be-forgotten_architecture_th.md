---
type: "practice"
difficulty: "Expert"
---

# Design a data-deletion / right-to-be-forgotten architecture that must purge a user from 50 services, backups, caches, and analytics within a legal deadline — how do you prove completeness?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective data-deletion (right-to-be-forgotten) architecture that purges a user from 50 services, backups, caches, and analytics within a legal deadline, we need to ensure comprehensive coverage, tracking, and verification of deletions. This architecture should be modular, scalable, and resilient. Here’s how we can achieve this:

#### Overview
Our goal is to create a system that can systematically remove data associated with a user across multiple services while ensuring that all relevant copies are purged. We’ll use a combination of centralized coordination, asynchronous processing, and tracking mechanisms to ensure completeness.

#### Triage
The question asks how we can prove completeness in deleting the user's data. This is primarily about verifying that every instance of the user’s data has been successfully deleted. We need to consider both the technical implementation (services, backups, caches) and legal compliance requirements.

#### First-Principles Solution

Coordination Service: A centralized service will coordinate the deletion process across all services.
Service Adapter Layer: Each service will have a custom adapter that handles the specific deletion logic for its system.
Event-driven Architecture: Use events to trigger deletions and track their status.
Tracking Mechanism: Maintain a detailed log of all deletions, including timestamps, service names, and result statuses.

Step-by-Step Solution:

User Request Capture: When a user requests deletion, capture this request in the coordination service. This could be via an API call or a user interface.
Service Adapters Activation: The coordination service triggers event-driven processes for each relevant service using a message queue (e.g., Kafka).
Deletion Execution: Each service’s adapter then performs the necessary deletions, ensuring that data is removed from its own storage and any associated backups.
Tracking Deletions: The adapters log their actions in a tracking database. This includes detailed records of what was deleted, by whom, and when.
Verification and Reporting: Once all events are processed, the coordination service generates a report to verify that all required services have completed their tasks successfully.

#### Closing Summary

Our architecture ensures that every piece of data is tracked and verified as it is deleted from various systems.
We use event-driven processes for scalability and reliability.
Detailed logs provide traceability, which is critical for legal compliance and auditing purposes.

#### Common Mistakes/Follow-Ups

Failing to adequately track all instances of the user’s data across services can lead to incomplete deletions.
Not having a robust verification mechanism may result in false positives or negatives.
Overlooking edge cases, such as data stored in third-party services, could compromise the overall solution.

**Comparison:**

```
Service	Adapter	Logs
A	UserDeletionAdapter	Deleted 12345
B	UserDeletionAdapter	Failed to delete [details]
```

**Code:**

```python
# Example Adapter Code for a Service

class UserDeletionAdapter:
    def __init__(self, service_name):
        self.service_name = service_name
        # Initialize storage and logging mechanisms

    def delete_user_data(self, user_id):
        try:
            # Delete data from the primary storage
            self.storage.delete(user_id)
            # Log the deletion event
            self.logger.info(f"Deleted {user_id} from {self.service_name}")
            return True
        except Exception as e:
            self.logger.error(f"Failed to delete {user_id} from {self.service_name}: {e}")
            return False

# Example Coordination Service

class DataDeletionCoordinationService:
    def __init__(self):
        # Initialize event queue, adapters, and logging mechanisms
        pass

    def handle_deletion_request(self, user_id):
        for service in self.services:
            adapter = UserDeletionAdapter(service)
            success = adapter.delete_user_data(user_id)
            if not success:
                # Handle failure (e.g., retry or escalate)
                pass
```

**Example input / output:**

```text
Input:
User requests deletion of user_id 12345

Output:
- Service A logs: Deleted 12345 from ServiceA
- Service B logs: Failed to delete 12345 from ServiceB due to error [details]

Report:
Service A: Success
Service B: Failure (requires manual intervention)
```

**Data:**

```json
{
  "services": [
    {"name": "ServiceA", "adapter": "UserDeletionAdapter"},
    {"name": "ServiceB", "adapter": "UserDeletionAdapter"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Requests Deletion
4. Coordination Service QueuesRequest
5. C
6. Log Data Deletion
7. D
8. Check if Successful
9. Notify Coordination Service
10. Try Again or Escalate
11. Track Completion Status
12. Generate Final Report

