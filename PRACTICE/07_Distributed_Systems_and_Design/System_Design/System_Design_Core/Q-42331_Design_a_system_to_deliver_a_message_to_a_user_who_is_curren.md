---
type: "practice"
difficulty: "Intermediate"
---

# Design a system to deliver a message to a user who is currently offline.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Chat History Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Chat_History_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system to deliver a message to a user who is currently offline, we need to ensure that messages are stored and delivered seamlessly when the user comes back online. This involves several key components: a reliable storage mechanism, an efficient delivery queue, and robust retry logic. Let's break down the design from first principles.

#### Triage

The core of this question is about designing a system for offline message handling. The interviewer is probing your understanding of distributed systems, queuing mechanisms, data persistence, and failover strategies. We need to make assumptions such as network stability, user device reliability, and storage constraints.

#### First-Principles Solution
Message Storage: Store messages in a reliable database or file system.
User State Tracking: Maintain a user state indicating whether the user is online/offline.
Message Queueing: Implement a queue to manage incoming messages that need to be delivered.
Offline Message Handling Logic:
When a message comes in, check if the recipient is offline.
If the user is offline, store the message and add it to the queue for later delivery.
If the user is online, send the message immediately.
Step-by-Step Breakdown

Message Storage Mechanism:

Choose a robust database system like PostgreSQL or Redis with persistence enabled to store messages reliably.

User State Tracking:

Use a service that maintains an up-to-date state of each user, indicating whether they are online/offline.

Message Queueing:

Implement a message queue such as RabbitMQ or Kafka to handle the delivery logic efficiently. This ensures messages can be processed in order and retries can be managed effectively.

Offline Message Handling Logic:

When a new message is received:
PYTHON
if user.is_offline():
    store_message_in_queue(message)
else:
    send_immediate(message)

Retries & Backoff Strategy:

Implement exponential backoff and retries in case the delivery fails due to network issues or service unavailability.

Error Handling & Logging:

Ensure comprehensive logging of every message state transition for debugging purposes.

#### Closing Summary

The key design elements are a reliable storage mechanism, user state tracking, an efficient queueing system, offline handling logic, and robust retry mechanisms. This ensures messages can be delivered reliably even when the user is offline.

#### Common Mistakes
& Follow-Ups
Mistake 1: Not considering edge cases such as what happens if the user is online but temporarily disconnected.

#### Follow-Up
Discuss how to handle transient network issues and ensure a smooth delivery experience for users.
Mistake 2: Overlooking storage constraints or choosing an inappropriate database system.

#### Follow-Up
Explore different database options like PostgreSQL, Redis, or a distributed file system like HDFS and discuss their trade-offs.

**Comparison:**

```
User ID	Is Online
1	No
2	Yes
Message ID	User ID	Content
101	1	Hello, this is a test message.
102	2	This is another test message.
```

**Code:**

```python
class User:
    def __init__(self, is_online=False):
        self.is_online = is_online

def store_message_in_queue(message):
    # Simulate storing message in queue
    print(f"Message stored for user: {message}")

def send_immediate(message):
    # Simulate sending message immediately
    print(f"Immediate delivery of message: {message}")

def handle_message(user, message):
    if user.is_offline():
        store_message_in_queue(message)
    else:
        send_immediate(message)

# Example usage
user = User(is_online=False)  # User is offline
handle_message(user, "Hello, this is a test message.")
```

**Example input / output:**

```text
Input:
- User with id 1 (offline)
- Message content: "Hello, this is a test message."

Output:
Message stored for user: Hello, this is a test message.

Explanation:
Since the user is offline, the system stores the message in the queue.
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "isOnline": false
    },
    {
      "id": 2,
      "isOnline": true
    }
  ],
  "messages": [
    {
      "id": 101,
      "userId": 1,
      "content": "Hello, this is a test message."
    },
    {
      "id": 102,
      "userId": 2,
      "content": "This is another test message."
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Checks Online Status
4. Is User Offline?
5. Store Message in Queue
6. Send Message Immediately
7. Queue Stores Message
8. Message Sent to User

