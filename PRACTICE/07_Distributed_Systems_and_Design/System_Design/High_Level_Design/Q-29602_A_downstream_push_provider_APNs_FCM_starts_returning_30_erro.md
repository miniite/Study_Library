---
type: "practice"
difficulty: "Deep"
---

# A downstream push provider (APNs/FCM) starts returning 30% errors — how does your notification service degrade gracefully, retry, and avoid a retry storm that makes it worse?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we need to design a robust notification service that can handle 30% errors from downstream push providers like APNs or FCM. The key is to ensure graceful degradation, efficient retries, and prevent retry storms, all while maintaining the reliability of delivering notifications.

#### Triage

The core question revolves around designing a resilient system for handling failures in external services. Specifically:

Graceful Degradation: Ensuring that even with high error rates, we can still deliver as many notifications as possible without crashing.
Efficient Retries: Implementing retries to handle transient errors but avoiding infinite loops or resource exhaustion.
Avoid Retry Storms: Preventing a sudden surge in retry requests when the initial failure occurs, which could overwhelm the system.

#### Assumptions
The notification service needs to send notifications in batches (e.g., every minute).
There is an underlying database or storage that tracks the state of each message.
We have control over the retry logic and can implement backoff mechanisms.
Errors from push providers are typically temporary, like network issues or transient server problems.

#### Solution
Step 1: Message Tracking

We need a way to track which messages were sent and their status (sent, failed). A simple database table will suffice:

SQL
CREATE TABLE notifications (
    id BIGINT PRIMARY KEY,
    recipient VARCHAR(255),
    message TEXT,
    status ENUM('SENT', 'FAILED') DEFAULT 'SENT',
    last_attempt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Step 2: Retry Logic

For each batch of messages, we attempt to send them. If a push provider returns an error (e.g., 30% error rate), the message is marked as failed and retried later.

PYTHON
def send_notifications(batch):
    failures = []
    for msg in batch:
        try:
            # Attempt to send the notification
            response = send_to_push_provider(msg)
            if response.status_code == 2xx:  # Assuming success codes are 200-299
                update_notification_status(msg.id, 'SENT')
            else:
                failures.append(msg)
        except Exception as e:
            # Log error and mark the message as failed
            logger.error(f"Error sending {msg}: {e}")
            update_notification_status(msg.id, 'FAILED')

    return failures

def send_to_push_provider(message):
    response = push_provider.send(message)
    if response.status_code < 300:
        return response
    else:
        raise PushProviderException("Server error")

def update_notification_status(notification_id, status):
    # Update the database to reflect new status
    db.update('UPDATE notifications SET status=%s, last_attempt=CURRENT_TIMESTAMP WHERE id=%s', (status, notification_id))
Step 3: Backoff Mechanism

Implement a backoff strategy to manage retries. This prevents rapid flooding of retry requests.

PYTHON
def exponential_backoff(retries):
    return min(2**retries * random.uniform(0.5, 1.5), 60)  # Cap at 60 seconds

class PushProviderException(Exception):
    pass

def retry_send(message, max_retries=3):
    for attempt in range(max_retries + 1):
        try:
            response = send_to_push_provider(message)
            if response.status_code < 300:  # Success
                return response
        except PushProviderException as e:
            logger.error(f"Retry {attempt} failed with {e}")
            time.sleep(exponential_backoff(attempt))
    raise PushProviderException("Max retries reached")
Step 4: Graceful Degradation and Capacity Management

Ensure the system can handle varying load. For example, during a high error rate period, it might be necessary to reduce the batch size or limit concurrent requests.

PYTHON
def process_notifications(batch_size=10):
    while True:
        batch = get_next_batch(batch_size)
        if not batch:  # No more batches available
            break
        failures = send_notifications(batch)
        for msg in failures:
            retry_send(msg, max_retries=3)

def get_next_batch(batch_size):
    batch = db.get_unsent_messages(batch_size)
    return [Notification(**msg) for msg in batch]
Step 5: Monitoring and Metrics

Set up monitoring to track the success rate, error rate, and backoff intervals. Use alerts to notify of potential issues.

PYTHON
def setup_monitoring():
    # Example using Prometheus or Grafana
    registry = CollectorRegistry()
    process_notifications_batch_size.set(batch_size)
    success_rate.set(90)  # Initial estimate

    start_http_server(8000, registry=registry)

# Start monitoring in background thread
Thread(target=setup_monitoring).start()

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key is to handle failures gracefully by tracking messages, implementing a backoff mechanism for retries, and managing load during high error periods. Common mistakes include not handling transient errors correctly or overloading the system with too many retries.

#### Follow-up

How would you integrate this solution into an existing microservices architecture?
What additional metrics would you track to optimize performance further?

**Comparison:**

```
id	recipient	message	status	last_attempt
1	user@example.com	{"title": "Reminder", "body": "Meeting in 30 mins"}	SENT	2023-10-15 14:00:00
2	user2@example.com	{"title": "Update", "body": "New project details available"}	FAILED	2023-10-15 14:00:00
```

**Code:**

```python
def send_to_push_provider(message):
    response = push_provider.send(message)
    if response.status_code < 300:
        return response
    else:
        raise PushProviderException("Server error")

def update_notification_status(notification_id, status):
    db.update('UPDATE notifications SET status=%s, last_attempt=CURRENT_TIMESTAMP WHERE id=%s', (status, notification_id))

def exponential_backoff(retries):
    return min(2**retries * random.uniform(0.5, 1.5), 60)

class PushProviderException(Exception):
    pass

def retry_send(message, max_retries=3):
    for attempt in range(max_retries + 1):
        try:
            response = send_to_push_provider(message)
            if response.status_code < 300:  # Success
                return response
        except PushProviderException as e:
            logger.error(f"Retry {attempt} failed with {e}")
            time.sleep(exponential_backoff(attempt))
    raise PushProviderException("Max retries reached")

def process_notifications(batch_size=10):
    while True:
        batch = get_next_batch(batch_size)
        if not batch:  # No more batches available
            break
        failures = send_notifications(batch)
        for msg in failures:
            retry_send(msg, max_retries=3)

def get_next_batch(batch_size):
    batch = db.get_unsent_messages(batch_size)
    return [Notification(**msg) for msg in batch]
```

**Example input / output:**

```text
Input:
- Batch of 10 messages

Output:
- Messages successfully sent are marked as 'SENT'
- Failed messages are retried with exponential backoff until max retries or success
- If a message fails after max retries, it is logged and considered failed for now.
```

**Data:**

```sql
CREATE TABLE notifications (
    id BIGINT PRIMARY KEY,
    recipient VARCHAR(255),
    message TEXT,
    status ENUM('SENT', 'FAILED') DEFAULT 'SENT',
    last_attempt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. Success
5. Failed
6. Get Next Batch
7. Is Batch Empty?
8. Messaging Done
9. Send Notifications
10. Are there failures?
11. Retry Failures with Backoff
12. Check Status
13. H

