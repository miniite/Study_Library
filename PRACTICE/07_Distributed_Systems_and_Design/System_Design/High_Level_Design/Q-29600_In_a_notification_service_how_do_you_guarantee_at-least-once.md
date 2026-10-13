---
type: "practice"
difficulty: "Deep"
---

# In a notification service, how do you guarantee at-least-once delivery while deduplicating so a user never gets the same push twice after a retry?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure at-least-once delivery while deduplicating notifications, we need to design a system that reliably stores notification attempts and checks against these records before re-sending. This involves implementing distributed locks, maintaining an up-to-date history of sent notifications, and handling retries gracefully.

#### Triaging the Question

The core challenge here is ensuring that users receive each push notification exactly once even after retries while preventing duplicate delivery. The interviewer likely expects you to discuss a combination of state management, retry logic, and deduplication mechanisms.

#### Assumptions
Notification Type: Push notifications sent via a service like Firebase Cloud Messaging (FCM).
Retry Mechanism: A queue-based system for retries.
User Context: Each user has an identifier that uniquely identifies them.

#### Solution
Step 1: Distributed Locks and Retries
Distributed Locking: To handle concurrent access to the notification history, we can use distributed locks (e.g., Redis' setnx for atomic checks) to ensure only one instance of a process tries to send a notification at any given time.
Retry Queue: Implement a retry queue that captures failed notifications and retries them until successful.
Step 2: Notification History Management
Notification Store: A database or distributed storage system (e.g., PostgreSQL, Cassandra) where we log the history of sent notifications. This store must be able to handle high read/write loads.
Unique Identifier for Notifications: Each notification gets a unique ID that includes metadata like user ID and timestamp.
Step 3: Deduplication Logic
Check Before Sending: Before sending a notification, check if it has already been sent by querying the notification history using the distributed lock mechanism. If not found, send the notification.
Mark as Sent: Once a notification is successfully sent, mark its status in the history to prevent resending.
Step 4: Handling Retry Scenarios
Failed Delivery: If a notification fails (e.g., due to network issues), add it back into the retry queue with an appropriate delay before attempting re-sent.
Exponential Backoff: Implement exponential backoff for retries to avoid overwhelming the system and to ensure reliable delivery.
Step 5: Edge Cases
User Session Timeout: If a user logs out, remove their notifications from the retry queue to prevent them from being resent indefinitely.
System Maintenance: Ensure that maintenance operations do not interfere with the state of notifications. For instance, when performing backups or updates, ensure that the notification history is consistent and not lost.

#### Closing Summary

By combining distributed locks for concurrency control, maintaining a persistent history of sent notifications, implementing retry queues, and handling edge cases like user session timeouts and system maintenance, we can effectively guarantee at-least-once delivery while deduplicating push notifications to prevent redundant messaging. This solution balances reliability with efficient resource usage.

#### Common Mistakes
and Follow-Ups:
Failing to Use Distributed Locks: Without proper concurrency control, multiple instances could attempt to send the same notification simultaneously.
Ignoring Retry Backoff: Failing to implement backoff can lead to overwhelming the system during transient failures.
Not Handling User Session Expiry: Forgetting to clean up expired user sessions can result in unnecessary retry attempts.

#### Example
Code (Simplified):
PYTHON
import redis

def send_notification(user_id, notification_id):
    r = redis.Redis(host='localhost', port=6379, db=0)

    with r.lock(f'user_{user_id}_lock'):
        # Check if notification has been sent before
        if not r.exists(notification_id):
            # Send the actual notification (e.g., using FCM SDK)
            send_fcm_notification(user_id, notification_text)

            # Mark as sent in Redis
            r.setex(notification_id, 3600, 'sent')  # TTL: 1 hour

def send_fcm_notification(user_id, notification_text):
    # Placeholder for actual FCM sending logic
    pass
Self-Check:

The solution is correct and covers all necessary aspects of the problem.

**Code:**

```python
import redis

def send_notification(user_id, notification_id):
    r = redis.Redis(host='localhost', port=6379, db=0)

    with r.lock(f'user_{user_id}_lock'):
        # Check if notification has been sent before
        if not r.exists(notification_id):
            # Send the actual notification (e.g., using FCM SDK)
            send_fcm_notification(user_id, notification_text)

            # Mark as sent in Redis
            r.setex(notification_id, 3600, 'sent')  # TTL: 1 hour

def send_fcm_notification(user_id, notification_text):
    # Placeholder for actual FCM sending logic
    pass
```

**Data:**

```json
{
  "user_id": "u123",
  "notification_id": "n456"
}
```

