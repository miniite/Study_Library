---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide between a layer-4 and a layer-7 load balancer for a service that needs path-based routing but also handles long-lived WebSocket connections?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between a Layer-4 (L4) and a Layer-7 (L7) load balancer in your scenario, we need to understand what each type of load balancer offers and how they can meet your specific requirements. The core decision revolves around the capabilities needed for path-based routing and handling long-lived WebSocket connections.

#### Triage
Understanding the Question

The question is probing our understanding of load balancing concepts, specifically Layer-4 vs Layer-7, and their implications on service architecture. We need to make assumptions about the nature of traffic (e.g., HTTP/HTTPS requests) and the type of services involved (WebSocket). We'll also consider factors like performance, security, and cost.

#### First Principles Solution
Path-Based Routing: This requires a Layer-7 load balancer because it needs to inspect the application layer to understand the URL path and route traffic accordingly.
Long-Lived WebSocket Connections: These require a connection-oriented approach, which is better handled by a Layer-4 load balancer due to its ability to maintain connections over time without closing them.
Step-by-Step Reasoning

Path-Based Routing:

Layer-7 Load Balancers: These inspect the application layer and can route traffic based on URL paths, headers, etc.
Layer-4 Load Balancers: These primarily handle transport-layer protocols (TCP/UDP) and make decisions based on source IP, destination IP, port numbers, and protocols.

Handling WebSocket Connections:

WebSocket Protocol: This is a bi-directional communication protocol over a single TCP connection, which can be long-lived.
Layer-4 Load Balancers: These are better suited for WebSocket because they maintain the state of each connection and ensure that the same client always communicates with the same backend server.

Hybrid Approach:

In practice, you might use a Layer-7 load balancer for path-based routing and then route the WebSocket connections to a Layer-4 load balancer specifically configured for handling these long-lived connections.
Alternatively, some modern L7 load balancers have features that allow them to handle WebSocket connections as well.

#### Closing Summary

In summary, you would typically use an L7 load balancer for path-based routing and another (or the same) layer-4 load balancer for WebSocket connections. This setup ensures both traffic is managed effectively while maintaining the state of long-lived connections.

#### Common Mistakes/Follow-Ups
Assuming a Single Solution: Not recognizing that different types of load balancing might be needed for different parts of your service.
Overlooking Connection State Management: Failing to understand how connection states are maintained in Layer-4 versus L7 load balancers.
Ignoring WebSocket-specific Requirements: Assuming all L7 load balancers can handle WebSocket connections, which may not always be the case.

**Comparison:**

```
Load Balancer Type	Capabilities	Path-Based Routing	Long-Lived WebSocket
Layer-7	URL path, headers, SSL	Yes	No
Layer-4	TCP/UDP, connection state	No	Yes
```

**Code:**

```python
# Example configuration for an L7 and L4 load balancer setup

l7_config = {
    "name": "path_based_routing_lb",
    "type": "L7",
    "rules": [
        {"path": "/api/v1/user", "to_backend": "user_service"},
        {"path": "/chat/ws", "to_backend": "websocket_service"}
    ]
}

l4_config = {
    "name": "websocket_connection_lb",
    "type": "L4",
    "keep_alive": True,
    "connections": ["backend1", "backend2"]
}
```

**Example input / output:**

```text
Input:
- HTTP request to "/api/v1/user"
- WebSocket connection to "/chat/ws"

Output:
- Path-based routing LB routes the HTTP request to "user_service"
- L4 load balancer manages the WebSocket connection, maintaining state and routing it to one of "backend1" or "backend2"
```

**Data:**

```json
{
  "load_balancers": [
    {
      "name": "path_based_routing_lb",
      "type": "L7",
      "rules": [
        {"path": "/api/v1/user", "to_backend": "user_service"},
        {"path": "/chat/ws", "to_backend": "websocket_service"}
      ]
    },
    {
      "name": "websocket_connection_lb",
      "type": "L4",
      "keep_alive": true,
      "connections": ["backend1", "backend2"]
    }
  ]
}
```

**Flow:**

1. HTTP Request
2. L7 Load Balancer
3. User Service
4. WebSocket Connection
5. L4 Load Balancer
6. Backend Server 1 or 2

