---
type: "practice"
difficulty: "Expert"
---

# Design a right-to-be-forgotten deletion architecture that must purge a user across 60 services CDC streams backups and a warehouse within a legal deadline — how do you orchestrate the fan-out prove completeness and handle backups you can't selectively edit?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an architecture that purges a user across 60 services, CDC streams, backups, and a warehouse within a legal deadline, we need to orchestrate a series of operations in a way that ensures data consistency and integrity while meeting the strict timelines required. The key challenge is handling the fan-out of deletion requests and ensuring that all relevant data sources are updated correctly.

#### Triaging the Question

The core problem here involves:

Fan-Out: Distributing a single delete request to multiple services.
Data Consistency: Ensuring that all related data in various systems (CDC streams, backups, warehouse) is purged simultaneously or at least within an acceptable timeframe.
Legal Compliance: Meeting the legal deadline for purging user data.

#### Assumptions
and Constraints
There are 60 services with CDC streams.
Backups cannot be selectively edited; they must be deleted entirely.
The architecture must handle large volumes of data efficiently.
Legal deadlines are critical, affecting the overall system's reliability and compliance.
Step-by-Step Solution

Event Sourcing: Implement event sourcing for each service to ensure that every change is recorded as a series of events. This will help in tracking deletions and ensuring no user data lingers.

Centralized Deletion Service: Create a centralized deletion service responsible for coordinating the purge process. The service will receive the initial delete request and fan-out the necessary operations to all relevant systems.

Event Bus: Utilize an event bus (like Kafka or RabbitMQ) to facilitate real-time communication between services. This allows for asynchronous processing, reducing latency and improving scalability.

Delete Operations:

For each service with a CDC stream, trigger a delete operation.
For backups, schedule their deletion post-cleanup of all data references.
Notify the warehouse system to perform a full purge if necessary.

Audit and Compliance: Maintain an audit log to track every action taken during the purging process. This ensures compliance with legal requirements.

Time Management:

Schedule backups for deletion after all other operations are complete.
Use time-based triggers to ensure no data is missed due to system delays.
Detailed Implementation

Event Sourcing:

Yes
No
User Deletion Request
Is Event Sourced?
Record Events in Each Service
Implement Event Sourcing

Centralized Deletion Service:

This service will coordinate the purge process.
It will receive a request and fan-out to all relevant services.

Event Bus:

CDC Event
CDC Event
Delete Backup
Full Purge
Deletion Request
Centralized Deletion Service
Service A
Service A CDC Stream
Service B
Service B CDC Stream
Backup Service
Backup Storage
Warehouse Service
Warehouse Database

Data Consistency:

Use transactions where possible to ensure atomicity.
For CDC streams, implement a mechanism to backfill any missing data.

Legal Compliance:

Implement strict time constraints for each operation.
Use distributed locks to avoid race conditions during critical operations.

Audit and Logs:

MARKDOWN
| Action      | Timestamp  |
|-------------|------------|
| User Deletion Request Received | 2023-10-01T14:00:00Z |
| Service A CDC Event Sent       | 2023-10-01T14:00:05Z |
| Service B CDC Event Sent       | 2023-10-01T14:00:10Z |
| Backup Storage Deleted         | 2023-10-01T14:05:00Z |
| Warehouse Database Purged      | 2023-10-01T14:10:00Z |

#### Closing Summary

The architecture involves a centralized deletion service coordinating the purge process via an event bus, ensuring data consistency and legal compliance. By leveraging event sourcing and strict time constraints, we can efficiently manage deletions across multiple services and data stores.

#### Common Mistakes/Follow-Ups
Not Implementing Event Sourcing: This can lead to inconsistencies if not all systems are updated.
Lack of Time Management: Missing critical deadlines due to delays in processing.
Selective Backup Editing: Attempting to selectively edit backups rather than deleting them entirely.

**Comparison:**

| Action      | Timestamp  |
|-------------|------------|
| User Deletion Request Received | 2023-10-01T14:00:00Z |
| Service A CDC Event Sent       | 2023-10-01T14:00:05Z |
| Service B CDC Event Sent       | 2023-10-01T14:00:10Z |
| Backup Storage Deleted         | 2023-10-01T14:05:00Z |
| Warehouse Database Purged      | 2023-10-01T14:10:00Z |

**Code:**

```python
# Example Python Code for Centralized Deletion Service

import asyncio
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')

async def delete_user(user_id):
    # Fan-out to all services
    await producer.send('delete_user', value=user_id.encode())
```

**Example input / output:**

```text
Input:
- User Deletion Request

Output:
- Service A CDC Event Sent
- Service B CDC Event Sent
- Backup Storage Deleted
- Warehouse Database Purged
```

**Data:**

```json
{
  "services": [
    { "name": "Service A", "cdc_stream": true },
    { "name": "Service B", "cdc_stream": false }
  ],
  "backups": {
    "type": "full",
    "schedule": "daily"
  },
  "warehouse": {
    "mode": "purge_on_deletion",
    "schedule": "hourly"
  }
}
```

**Flow:**

1. Yes
2. No
3. CDC Event
4. CDC Event
5. Delete Backup
6. Full Purge
7. User Deletion Request
8. Is Event Sourced?
9. Record Events in Each Service
10. Implement Event Sourcing
11. Centralized Deletion Service
12. Service A
13. Service A CDC Stream
14. Service B
15. Service B CDC Stream
16. Backup Service
17. Backup Storage
18. Warehouse Service
19. Warehouse Database

