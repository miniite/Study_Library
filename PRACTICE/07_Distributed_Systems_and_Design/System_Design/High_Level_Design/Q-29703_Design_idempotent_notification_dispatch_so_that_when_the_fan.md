---
type: "practice"
difficulty: "Deep"
---

# Design idempotent notification dispatch so that when the fanout worker crashes and its message is redelivered the user does not receive a duplicate push — where do you place the dedup key and what's its TTL?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotent notification dispatch system, we need to ensure that users receive notifications only once even if messages are redelivered due to a worker crash. The core idea is to use a deduplication key and Time-To-Live (TTL) mechanism. Let's break down the solution step by step.

#### Triage

The question asks us to design an idempotent notification dispatch system where users do not receive duplicate push notifications if messages are redelivered due to a worker crash. This means we need to ensure that each message is processed exactly once, even when it gets re-delivered.

#### Assumptions
and Key Idea
We will use a distributed system with multiple workers handling the notifications.
Messages can be redelivered if a worker crashes.
The goal is to avoid duplicate notifications reaching the user.

#### Solution
Deduplication Key: The deduplication key should be unique for each notification and should be stored in a place where it can be checked before sending the push notification.
TTL (Time-To-Live): The TTL ensures that old messages are eventually removed from the storage, preventing them from being processed multiple times.
Placement of Dedup Key
Database: Store the deduplication key in a database table that keeps track of sent notifications.
Message Queue: Store the deduplication key in the metadata of the message when it is enqueued for processing.
Step-by-Step Solution

Generate and Store Dedup Key:

When sending a notification, generate a unique ID (deduplication key).
Store this ID in both the database and the message queue metadata.

Check Before Processing:

Before processing a message, check if the deduplication key already exists in the database.
If it does not exist, proceed with sending the notification and mark the key as sent by updating the database.

Update TTL:

Set a TTL for the entry in the database to ensure that old keys are eventually removed.
A common value might be 1 day (86400 seconds).

#### Example
Flow
When a worker receives a notification message, it extracts the deduplication key from the message metadata.
It then checks if this key exists in the database.
If not, it sends the push notification and updates the database to mark the key as sent.
The TTL ensures that after 1 day, the entry is automatically removed.

#### Closing Summary

By using a distributed deduplication mechanism with a dedicated storage (database) for tracking notifications and setting an appropriate TTL, we can ensure idempotence in our notification dispatch system. This approach prevents duplicate push notifications from reaching users even if messages are redelivered due to worker crashes.

#### Common Mistakes/Follow-Ups
Forgetting the Dedup Key: Not including a unique key that can be used for deduplication.
Inadequate TTL Setting: Setting too short or long a TTL, which could lead to incorrect behavior (too frequent cleanup vs. potential duplicates).
Message Retransmission: Failing to handle retransmissions properly in the database check logic.

**Comparison:**

| Dedup Table Schema |
|--------------------|
| **Key** | **Expire Time** |
| string  | timestamp       |

**Code:**

```python
# Example Code for Deduplication Key Handling

import uuid
from datetime import timedelta

def send_notification(user_id, message):
    # Generate a unique dedup key
    dedup_key = str(uuid.uuid4())

    # Check if the key is already in the database
    if not is_key_in_database(dedup_key):
        # Send push notification to user
        send_push_to_user(user_id, message)

        # Update the database with the new key and set TTL
        insert_into_dedup_table(dedup_key, expire_time=timedelta(days=1))
```

**Example input / output:**

```text
- **Input**: `send_notification(user_id='user123', message='Hello user!')`
- **Output**:
  - If the key is not in the database, a push notification is sent to 'user123' and the key is inserted into the dedup table with an expiration of 1 day.
```

**Data:**

```json
{
  "dedup_table": {
    "key": "string",
    "expire_time": "timestamp"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Receive Message
5. Is Key in DB?
6. Drop Message
7. Send Push Notification?
8. Insert Dedup Key into DB
9. Set TTL
10. Dedup Key Exists

