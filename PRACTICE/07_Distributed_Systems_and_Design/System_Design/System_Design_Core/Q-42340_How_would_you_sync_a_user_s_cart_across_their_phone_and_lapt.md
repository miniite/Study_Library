---
type: "practice"
difficulty: "Intermediate"
---

# How would you sync a user's cart across their phone and laptop in real time?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue vs Synchronous Calls](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To synchronize a user’s cart across multiple devices (phone and laptop) in real time, we need to design a system that ensures data consistency and low latency while handling potential network issues. The core challenge is ensuring that updates on one device are seamlessly propagated to the other without causing conflicts or performance degradation.

#### Triage

The question essentially asks how to maintain a consistent state of a user’s cart across multiple devices in real time. This involves understanding the underlying architecture, choosing the right technology stack, and implementing mechanisms for data synchronization and conflict resolution.

Key Assumptions:
User Authentication: Each device must be authenticated using a secure mechanism.
Network Latency: Network conditions can introduce delays or failures; we need to handle these gracefully.
Conflict Resolution: In the event of concurrent updates, a strategy is needed to resolve conflicts.

#### First Principles Solution
Step 1: Data Storage

We will use a centralized database for storing cart data. This could be a NoSQL database like MongoDB or a relational database like PostgreSQL, depending on the specific requirements and scale.

User Device
CDB (Central Database)
Step 2: Real-time Synchronization

To ensure real-time updates, we can use WebSocket technology to push cart state changes from the server to all connected devices. Alternatively, a publish-subscribe model using a message broker like RabbitMQ or Kafka can be employed.

User Device
CDB (Central Database)
Message Broker
WebSocket Server
Step 3: Conflict Resolution

When devices are updated concurrently, a conflict resolution mechanism is necessary. This can be as simple as applying the latest update or more complex like merging changes.

User Device
CDB (Central Database)
Message Broker
WebSocket Server
Conflict Resolver
Step 4: Authentication and Authorization

Ensure that only authenticated users can modify their cart. This is typically handled by a secure login mechanism, such as OAuth or JWT.

User Device
CDB (Central Database)
Authentication Service

#### Closing Summary

In summary, the system design for real-time cart synchronization across multiple devices involves using a centralized database for storing cart data and leveraging WebSocket technology or a message broker to enable real-time updates. Conflict resolution is crucial to handle concurrent updates gracefully. Additionally, robust authentication ensures that only authorized users can modify their carts.

#### Common Mistakes
and Follow-Ups
Not Handling Network Issues: Failing to account for network outages or high latency can lead to inconsistent states.
Lack of Conflict Resolution Strategy: Without a proper conflict resolution mechanism, concurrent updates may result in data corruption.
Ignoring Performance Considerations: The design should consider the performance implications of real-time updates and large-scale deployments.

**Comparison:**

```
Device	Action
Phone	Updates cart item quantity
Laptop	Receives update via WebSocket
```

**Code:**

```python
# Example WebSocket Client Code
import websocket

def on_message(ws, message):
    print(f"Received cart update: {message}")

ws = websocket.WebSocketApp("wss://example.com/cart", on_message=on_message)
ws.run_forever()
```

**Example input / output:**

```
Input: User updates cart on phone
Output: Cart update pushed to laptop via WebSocket
JSON
Copy
{
  "userId": 12345,
  "cartItems": [
    {"id": 1, "quantity": 3},
    {"id": 2, "quantity": 1}
  ]
}
```

**Data:**

```json
{
  "userId": 12345,
  "cartItems": [
    {"id": 1, "quantity": 3},
    {"id": 2, "quantity": 1}
  ]
}
```

**Flow:**

1. User Device
2. CDB (Central Database)
3. Message Broker
4. WebSocket Server
5. Conflict Resolver
6. User Device

