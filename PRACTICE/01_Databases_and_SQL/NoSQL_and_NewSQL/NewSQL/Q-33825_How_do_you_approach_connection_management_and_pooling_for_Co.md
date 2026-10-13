---
type: "practice"
difficulty: "Deep"
---

# How do you approach connection management and pooling for CockroachDB at 100k+ concurrent app connections, given per-connection memory and the gateway-node model, and where does a proxy like pgbouncer help or hurt?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ NewSQL Evaluation and Resilience](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with managing connection pools in a high-concurrency environment using CockroachDB, specifically within its "gateway-node" model. The key challenge is to ensure efficient and reliable management of connections while adhering to the memory constraints imposed by per-connection overhead. We'll explore how connection pooling works, the role of proxies like pgbouncer, and why they can either help or hinder the system.

Connection management in CockroachDB involves balancing between gateway nodes (which act as load balancers) and the actual database nodes where data resides. Each client connection is managed by a gateway node before being routed to an appropriate database node, adding a layer of complexity to traditional connection pooling strategies.

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant Client
    participant GatewayNode
    participant DatabaseNode
    Client->>GatewayNode: Establish Connection Request
    GatewayNode->>DatabaseNode: Route and Manage Connection
```

**Flow:**

1. Yes
2. No
3. Client
4. Connection Request toGateway
5. Gateway Node Route Selection
6. Is Pool Full?
7. Create New Connection
8. Reuse Existing Connectionfrom Pool
9. Send Query via Gateway toDatabase Node
10. Database Node Response
11. Gateway Node ForwardResponse to Client

