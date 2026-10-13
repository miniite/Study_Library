---
type: "practice"
difficulty: "Deep"
---

# Your app fires LISTEN/NOTIFY for real-time updates but notifications get dropped when clients reconnect; explain the delivery semantics and when you'd move to a durable queue.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with PostgreSQL's LISTEN/NOTIFY mechanism for real-time updates in our application. The issue is that notifications get dropped when clients reconnect, which can lead to loss of critical data. To understand the root cause and potential solutions, let’s break down the delivery semantics and explore when a durable queue might be necessary.

TL;DR

PostgreSQL's LISTEN/NOTIFY mechanism uses an in-memory buffer for delivering notifications. This means that if a client reconnects after receiving some notifications but before all were delivered, those missed notifications will be lost. To address this, we need to understand the semantics and explore durable queue alternatives.

#### Triaging the Question
Semantics: The question asks us to explain the delivery semantics of LISTEN/NOTIFY.

#### Assumptions
Clients are reconnecting frequently.
Notifications are being lost during reconnections.
Key Points:
We need to understand how LISTEN/NOTIFY works under the hood.
Explore scenarios where durable storage is necessary.

#### First-Principles Solution
Delivery Semantics of LISTEN/NOTIFY

PostgreSQL's LISTEN and NOTIFY commands work by maintaining an in-memory buffer for each listener. When a NOTIFY command is issued, all listeners are notified via the operating system's signal mechanism (e.g., Unix signals). The key points to consider are:

In-Memory Buffer: Listeners store notifications temporarily in memory.
No Persistent Storage: Notifications are not stored persistently; they rely on the client receiving them before the connection is closed.

When a client reconnects, it re-listens for new notifications but cannot receive previously missed ones. This can lead to data loss if critical updates were issued during the disconnection period.

When to Move to a Durable Queue

Given the limitations of LISTEN/NOTIFY, moving to a durable queue is necessary in scenarios where:

Critical Data Loss: Losing notifications could result in application state inconsistencies.
Frequent Reconnections: Clients reconnect frequently, increasing the likelihood of missed messages.
Full Solution

To ensure that critical updates are not lost during client disconnections, we need to implement a more robust solution. Here’s a step-by-step approach:

Capture Notifications: Implement an intermediary queue or storage layer that captures notifications.
Persist Notifications: Store NOTIFY commands in a durable queue (e.g., Redis, Kafka).
Client Subscription: Clients subscribe to the durable queue instead of relying on LISTEN/NOTIFY.
Notification Handling: When clients reconnect, they can consume messages from the durable queue.

#### Example
Implementation

Let’s consider using Redis as a durable queue:

Redis Setup:

Use Redis to store notifications.
SH
redis-server &

PostgreSQL Triggers:

Create triggers to push NOTIFY commands to Redis.
SQL
CREATE OR REPLACE FUNCTION notify_to_redis() RETURNS TRIGGER AS $$
  BEGIN
    -- Simulate pushing to Redis
    EXECUTE 'redis-cli PUBLISH notification_channel ' || NEW.message;
    RETURN NEW;
  END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER notify_trigger
AFTER INSERT ON notifications_table
FOR EACH ROW EXECUTE FUNCTION notify_to_redis();

Client Subscription:

Clients subscribe to Redis for updates.
JAVASCRIPT

#### cons
t client = redis.createClient();

client.on('connect', () => {

#### cons
ole.log('Connected to Redis');
  client.subscribe('notification_channel');

  client.on('message', (channel, message) => {

#### cons
ole.log(`Received: ${message}`);
    // Process the notification here.
  });
});

process.on('SIGINT', async () => {
  await client.quit();
  process.exit(0);
});

#### Closing Summary

In summary, PostgreSQL's LISTEN/NOTIFY mechanism is not suitable for scenarios where critical updates need to be guaranteed. Implementing a durable queue can help ensure that notifications are delivered reliably even during client reconnections.

#### Common Mistakes
and Follow-Ups
Mistake: Relying solely on LISTEN/NOTIFY without considering its limitations.

#### Follow-Up
Discuss potential trade-offs between using a durable queue (e.g., increased complexity, cost) vs. the benefits of guaranteed delivery.

**Comparison:**

```
ID	Message
1	update
Connection State	Action
Connected	Subscribe to Redis channel.
Disconnected	Reconnect and resubscribe.
```

**Code:**

```

CREATE OR REPLACE FUNCTION notify_to_redis() RETURNS TRIGGER AS $$ BEGIN EXECUTE 'redis-cli PUBLISH notification_channel ' || NEW.message; RETURN NEW; END; $$ LANGUAGE plpgsql;

CREATE TRIGGER notify_trigger AFTER INSERT ON notifications_table FOR EACH ROW EXECUTE FUNCTION notify_to_redis();

CODE
Copy

```javascript
const client = redis.createClient();

client.on('connect', () => {
  console.log('Connected to Redis');
  client.subscribe('notification_channel');

  client.on('message', (channel, message) => {
    console.log(`Received: ${message}`);
    // Process the notification here.
  });
});

process.on('SIGINT', async () => {
  await client.quit();
  process.exit(0);
});
```

**Example input / output:**

```
Input: INSERT INTO notifications_table (message) VALUES ('update')
Output: Redis publishes the message to the notification_channel.
SH
Copy
redis-cli PUBLISH notification_channel update
JAVASCRIPT
Copy
client.on('connect', () => {
  console.log('Connected to Redis');
  client.subscribe('notification_channel');

  client.on('message', (channel, message) => {
    console.log(`Received: ${message}`);
  });
});
```

**Data:**

```json
{
  "notifications_table": {
    "columns": [
      { "name": "id", "type": "SERIAL" },
      { "name": "message", "type": "TEXT" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Client Connects
4. Is Client Disconnecting?
5. Reconnect and Resubscribe
6. Listen for Notifications
7. Notify to Redis
8. Redis Pushes Notification
9. Client Subscribes to Channel

