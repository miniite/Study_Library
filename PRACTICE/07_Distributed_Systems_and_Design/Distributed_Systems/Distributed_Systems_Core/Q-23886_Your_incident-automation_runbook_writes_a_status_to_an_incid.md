---
type: "practice"
difficulty: "Deep"
---

# Your incident-automation runbook writes a status to an incident DB and posts to a chat channel as two separate writes, and a crash between them left the incident marked resolved but nobody notified. Explain this dual-write hazard and how a transactional outbox fixes the drift.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Dead Letter Queues and Outbox](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Dead_Letter_Queues_and_Outbox.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, the dual-write problem refers to scenarios where multiple write operations are required to achieve a coherent state, such as updating an incident database and notifying stakeholders via a chat channel. When these writes are performed separately without proper coordination, a failure between the two can lead to inconsistent states. This is precisely what happened in your scenario: while one part of the system marked the incident as resolved, another failed to notify anyone.

To understand this more deeply, let’s break down the problem from first principles and explore how a transactional outbox can mitigate such hazards.

**Comparison:**

```
Write Operation	Status
Database Update	Success
Chat Channel Notification	Failure
Resulting State	Incident marked resolved but no notification sent.
```

**Code:**

```python
# Example function for writing status to an incident DB and posting to a chat channel
def update_incident(incident_id, resolved):
    # Step 1: Update incident database
    db.update_status(incident_id, resolved)

    # Step 2: Post notification to chat channel
    slack.post_message(f"Incident {incident_id} is now {'resolved' if resolved else 'open'}")
```

**Example input / output:**

```text
Given the input:
- `incident_id`: 12345
- `resolved`: true

The function would attempt to perform two separate writes:
1. Update the database with status resolved.
2. Post a message to the chat channel.

However, if there is an unexpected crash between these steps, only one of them may succeed while the other fails, leading to inconsistent states.
```

**Data:**

```json
{
  "incident_id": 12345,
  "resolved": true
}
```

**Flow:**

1. Incident Update Request
2. Database Write
3. Update Incident Status
4. Crash
5. Notification Failed
6. Database Updated
7. Notify Chat Channel
8. Inconsistent State: Resolvedbut Unnotified

