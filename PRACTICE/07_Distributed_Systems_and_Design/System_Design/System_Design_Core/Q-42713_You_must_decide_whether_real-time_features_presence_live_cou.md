---
type: "practice"
difficulty: "Deep"
---

# You must decide whether real-time features (presence, live counts, notifications) share one WebSocket infrastructure or each team runs its own. Argue the platform-vs-autonomy trade-off, and design the multiplexed connection layer if you centralize it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether real-time features like presence, live counts, and notifications should share a single WebSocket infrastructure or run independently by each team, we need to weigh the benefits of platform control against the advantages of autonomy. The central decision hinges on the trade-off between shared resources and independent development.

First, let's define our key assumptions:

Presence, live counts, and notifications are essential real-time features.
Each feature has its own unique requirements and data flow.
Different teams have varying technical expertise and development practices.

The core question is: Is it more beneficial to centralize these real-time features in a single WebSocket infrastructure for shared resource management, or should each team run their own independent WebSocket connections?

From first principles, we can analyze the trade-offs:

Platform Control vs Autonomy

Centralization (Platform Control):
Easier debugging and monitoring.
Simplified security and authentication mechanisms.
Centralized logging for better observability.
Reduced risk of conflicting configurations or protocols.
Decentralization (Autonomy):
Teams can implement features faster as they have full control over the infrastructure.
More flexibility to adapt to team-specific needs without platform-wide changes.

Resource Management

Centralizing WebSocket connections can lead to more efficient use of resources, such as shared server capacity and network bandwidth.
However, this requires careful management to avoid performance bottlenecks or denial-of-service attacks from misbehaving clients.

Security and Compliance

Centralization allows for uniform security policies across the platform, reducing the risk of security breaches.
Decentralized solutions may make it harder to enforce consistent security measures.

Scalability and Maintainability

A centralized solution can be easier to scale and maintain if properly designed with modularity in mind.
Decentralization might lead to duplicated efforts and inconsistencies, making maintenance more complex.

Development Workflow

Centralized infrastructure provides a common platform for all teams, streamlining development workflows.
Autonomous solutions give developers the freedom to innovate but may lead to fragmented development practices.

#### Cons
idering these factors, we decide that centralizing the WebSocket connections is the better choice. This approach allows us to leverage shared resources while maintaining control over security and performance. To design a robust multiplexed connection layer:

Multiplexing Strategy:

Implement multiplexing using WebSockets' inherent ability to handle multiple streams of data concurrently.
Use an event-driven architecture where different types of messages (presence, live counts, notifications) are tagged with unique identifiers.

Security and Authentication:

Enforce strict authentication mechanisms at the WebSocket level to ensure only authorized clients can connect and send/receive data.
Implement role-based access control (RBAC) for fine-grained permissions management.

Monitoring and Logging:

Centralize logging of all WebSocket activities for easier troubleshooting and auditing.
Set up real-time monitoring tools to track connection health, message throughput, and error rates.

Failover and Resilience:

Design the system with redundancy in mind, ensuring that a failure in one part does not bring down the entire infrastructure.
Implement load balancing techniques to distribute traffic evenly across multiple WebSocket servers.

By centralizing the WebSocket connections and implementing these strategies, we can achieve a balanced solution that maximizes platform efficiency while preserving team autonomy.

In conclusion, by centralizing real-time features into a single WebSocket infrastructure, we gain control over resources, security, and maintenance. The multiplexed connection layer ensures efficient handling of different types of messages while maintaining robustness and scalability.

**Code:**

```
MERMAID
Copy
graph TD;
    A[WebSocket Server] --> B1(Receive Presence)
    A --> B2(Receive Live Counts)
    A --> B3(Receive Notifications)
    B1 --> C1[Publish to Database]
    B2 --> C2[Publish to Database]
    B3 --> C3[Publish to Database]
    C1 --> D1[Send to Subscribers]
    C2 --> D2[Send to Subscribers]
    C3 --> D3[Send to Subscribers]
```

**Example input / output:**

```text
Input:
- WebSocket connection from client A for presence update
- WebSocket connection from client B for live count update

Output:
- Presence message sent to all subscribers
- Live count message sent to all subscribers
```

**Data:**

```json
{
  "presence": {
    "id": 1,
    "user_id": 100,
    "status": "online"
  },
  "live_count": {
    "id": 2,
    "product_id": 50,
    "quantity": 100
  },
  "notification": {
    "id": 3,
    "recipient_id": 200,
    "message": "New product available"
  }
}
```

