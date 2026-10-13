---
type: "practice"
difficulty: "Deep"
---

# You expose WebSocket and SSE endpoints through the gateway alongside REST; how does long-lived connection handling differ, and what timeout and load-balancing pitfalls do you watch for?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle WebSocket and Server-Sent Events (SSE) effectively within an API Gateway, it’s crucial to understand how these long-lived connections differ from typical HTTP REST endpoints. The key differences involve handling timeouts, ensuring reliable delivery of data, managing load balancing, and avoiding common pitfalls.

API Gateways manage a wide array of protocols and can route requests between clients and backend services efficiently. When exposing WebSocket and SSE endpoints, we need to consider the unique challenges these protocols present. WebSockets maintain persistent connections, allowing real-time bidirectional communication, while SSEs provide a way for the server to push data to the client.

#### Triaging the Question

The question is probing how you manage long-lived connections in an API Gateway context. Specifically, it asks about handling timeouts and load balancing. Here are some key assumptions:

The API Gateway supports both WebSocket and SSE protocols.
There might be a mix of short-lived HTTP requests alongside these long-lived connections.

#### First Principles Solution

Firstly, let's break down the problem into its components:

Handling WebSockets:

Persistent Connections: Unlike REST endpoints which are stateless, WebSocket connections can persist for extended periods.
Timeouts: You must set appropriate timeouts to avoid hanging connections and ensure resource management.

Handling Server-Sent Events (SSE):

Pushing Data Reliably: Ensure reliable delivery of events from the server to the client.
Load Balancing: Distribute traffic efficiently across backend servers while maintaining state awareness.
Step-by-Step Solution

Timeouts for WebSocket:

Implement a heartbeat mechanism where clients and servers periodically check on each other's status.
Use connection-level timeouts to close idle connections automatically.

#### Example
JAVASCRIPT
// Pseudo-code for WebSocket timeout handling
let lastActivity = Date.now();

#### cons
t HEARTBEAT_INTERVAL = 10 * 1000; // 10 seconds

ws.onmessage = (event) => {
  lastActivity = Date.now();
  processMessage(event.data);
};

setInterval(() => {
  if ((Date.now() - lastActivity) > HEARTBEAT_INTERVAL) {
    ws.close(); // Close connection if no activity
  }
}, HEARTBEAT_INTERVAL / 2);

Reliable Data Delivery for SSE:

Implement a robust error handling mechanism to reconnect clients in case of disconnection.
Ensure the server can handle reconnections gracefully without duplicating data.

Load Balancing Considerations:

Use sticky sessions (session affinity) to ensure that WebSocket and SSE connections are routed to the same backend server.
Implement health checks on backend servers to ensure they remain available for handling long-lived connections.

#### Example
PYTHON
# Pseudo-code for load balancer setup
from lb import LoadBalancer

lb = LoadBalancer()
lb.add_backend('http://backend1', 'websocket')
lb.add_backend('http://backend2', 'websocket')

client_id = "ws-001"
backend_server = lb.get_backend_for(client_id)

#### Closing Summary

In summary, handling WebSocket and SSE endpoints in an API Gateway requires managing long-lived connections with appropriate timeouts, ensuring reliable data delivery, and implementing load balancing strategies. Careful design is necessary to maintain the integrity of these connections while efficiently scaling your backend services.

#### Common Mistakes/Follow-Ups
Pitfall: Failing to implement heartbeats can lead to zombie WebSocket connections.

#### Follow-Up
How would you handle SSL/TLS termination in an API Gateway for secure WebSocket and SSE connections?

**Comparison:**

```
WebSocket	SSE
Persistent connection	Server pushes data to client
Heartbeat mechanism for timeouts	Reliability mechanisms for disconnections
```

**Code:**

```javascript
// Pseudo-code for WebSocket timeout handling
let lastActivity = Date.now();
const HEARTBEAT_INTERVAL = 10 * 1000; // 10 seconds

ws.onmessage = (event) => {
  lastActivity = Date.now();
  processMessage(event.data);
};

setInterval(() => {
  if ((Date.now() - lastActivity) > HEARTBEAT_INTERVAL) {
    ws.close(); // Close connection if no activity
  }
}, HEARTBEAT_INTERVAL / 2);
```

**Example input / output:**

```text
Input:
POST /setup-websocket

Output:
HTTP 201 Created
Location: ws://localhost/websocket-connection?client_id=ws-001

Request:
GET /events

Response:
HTTP 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache, no-store, must-revalidate
Expires: 0
Pragma: no-cache

data: {"event": "update", "payload": "new data"}
```

**Data:**

```yaml
# Example configuration for LoadBalancer
load_balancer:
  backends:
    - id: "ws-001"
      url: "http://backend1"
      protocol: "websocket"
    - id: "ws-002"
      url: "http://backend2"
      protocol: "websocket"
```

**Flow:**

1. WebSocket Connection
2. Heartbeat Mechanism
3. Close Connection if No Activity
4. SSE Endpoint
5. Reliable Data Delivery
6. Load Balancer
7. Sticky Sessions for WebSockets
8. Health Checks on Backends

