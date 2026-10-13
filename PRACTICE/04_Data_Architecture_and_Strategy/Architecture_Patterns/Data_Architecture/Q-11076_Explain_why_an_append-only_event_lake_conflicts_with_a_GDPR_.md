---
type: "practice"
difficulty: "Deep"
---

# Explain why an append-only event lake conflicts with a GDPR/DPDP deletion request, and describe a practical approach your team could implement to actually delete a user.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In data engineering, we often encounter situations where our data architecture is built around certain assumptions about how data should be managed and accessed. An append-only event lake is a common architectural choice that suits scenarios requiring real-time processing and event tracking. However, this approach conflicts with the requirements of GDPR (General Data Protection Regulation) and DPDP (Data Privacy and Protection), particularly when it comes to deleting user data. This answer will explain why such a conflict exists and propose a practical solution for deletion while maintaining compliance.

#### Triaging the Question

The core issue here is understanding how an append-only event lake's design differs from the requirements of GDPR/DPDP, which necessitate the ability to delete personal data. The question requires us to break down the fundamental principles of both architectures and then propose a strategy that aligns with legal obligations while leveraging existing systems.

#### First-Principles Solution

Understanding Append-Only Event Lakes:

An append-only event lake is structured such that once data is written, it cannot be modified or deleted. This design ensures consistency and prevents accidental overwrites.
Data is stored as a series of immutable events, which are aggregated into smaller batches before being loaded into a data warehouse.

GDPR/DPDP Requirements:

GDPR and DPDP require the ability to delete personal data (Right to be Forgotten) within specific timeframes.
Deletion must be complete and irreversible, ensuring that all traces of the data are removed.

Conflict Between Append-Only Lakes and Data Deletion:

The append-only nature of event lakes means that once a user's event is recorded, it cannot be deleted or modified without breaking the integrity of the data.
This conflict arises because GDPR/DPDP mandates the ability to delete personal data, which is not compatible with an immutable data storage model.
Proposed Solution:

Data Mesh Architecture:

Introduce a Data Mesh architecture that allows for more granular control over data, including the ability to delete specific records.
In this approach, the event lake remains as the source of truth but is complemented by a mesh network where each node (e.g., business teams) can manage and manipulate their own copies of the data.

Implementing Deletion:

Metadata Management: Implement a metadata layer that tracks all events related to a user, including deletions.
Event Sourcing: Use event sourcing techniques to capture every action taken on a record, allowing for the reconstruction of its history and eventual deletion.
Data Shredding: Design an ETL (Extract, Transform, Load) process that can shred or anonymize data, ensuring compliance with GDPR/DPDP.

Technical Implementation:

Event Lake Layer: Maintain the append-only event lake as the immutable source of truth.
Mesh Nodes: Each business unit maintains its own copy of the data and handles deletion requests.
Audit Trail: Ensure an audit trail is maintained at both the mesh node level and within the event lake metadata.

#### Example
Workflow:

When a delete request comes in, it triggers an ETL job that:
Identifies all events related to the user.
Marks these events as deleted in the metadata layer.
Shreds or anonymizes the data at the mesh node level.

#### Closing Summary
and Common Mistakes/Follow-Ups:

#### Common Mistakes

Failing to implement an audit trail, which is crucial for compliance and accountability.
Ignoring the need for granular control over data at each business unit, leading to inefficiencies in managing deletion requests.
Overlooking the importance of maintaining consistency between the event lake and mesh nodes.

#### Follow-Ups

How would you handle bulk delete requests?
What challenges might arise with scaling this solution across multiple business units?
How do you ensure that your implementation remains compliant over time as regulations evolve?

**Comparison:**

```
Event ID	Timestamp	User ID	Event Type
12345	2023-10-01T10:00	alice	login
67890	2023-10-01T15:00	alice	logout
```

**Code:**

```python
# Example Python code for marking events as deleted in metadata layer
def mark_event_as_deleted(event_id, user_id):
    # Update the metadata layer to mark the event as deleted
    pass

# Example ETL process for shredding data
def shred_data(user_id):
    # Shred or anonymize data at mesh node level
    pass
```

**Example input / output:**

```text
Input:
Delete user 'alice' from the system.

Output:
1. Mark all events of user 'alice' as deleted in metadata layer.
2. Shred or anonymize data at mesh node level for user 'alice'.
3. Audit trail updated to reflect deletion actions.
```

**Data:**

```json
{
  "events": [
    {"id": "12345", "timestamp": "2023-10-01T10:00:00Z", "user_id": "alice", "event_type": "login"},
    {"id": "67890", "timestamp": "2023-10-01T15:00:00Z", "user_id": "alice", "event_type": "logout"}
  ],
  "metadata": {
    "user_id": "alice",
    "deletion_events": [
      {"id": "98765", "timestamp": "2023-10-02T14:00:00Z", "status": "completed"}
    ]
  }
}
```

**Flow:**

1. Start Delete Request
2. Identify Events Related to User
3. Mark Events as Deleted inMetadata Layer
4. Shred or Anonymize Data atMesh Node Level
5. Audit Trail Updated

