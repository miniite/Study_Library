---
type: "practice"
difficulty: "Deep"
---

# Explain how HTTP/2 and gRPC change gateway design vs HTTP/1.1; how do you proxy gRPC, handle multiplexing, and deal with load balancing long-lived connections?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Load Balancing and Zone Aware Routing](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, I will explain how HTTP/2 and gRPC impact API gateway design compared to HTTP/1.1, focusing on proxying, multiplexing, and load balancing of long-lived connections.

TL;DR: HTTP/2 introduces a single multiplexed connection per host, which changes the way we handle request/response cycles and improves performance over HTTP/1.1. gRPC is an open-source remote procedure call (RPC) framework that enables efficient, high-performance communication between distributed systems. It uses Protocol Buffers for defining service interfaces and supports streaming, making it ideal for scenarios requiring bidirectional communication or long-lived connections.

#### Triaging the Question
The question asks about how HTTP/2 and gRPC affect gateway design. Specifically, it probes our understanding of multiplexing in HTTP/2, the role of a proxy in gRPC, and strategies for load balancing during long-lived connections. We need to consider both protocol-level changes and architectural implications.

#### Assumptions

The API gateway is responsible for receiving incoming requests from clients, routing them to appropriate services, and returning responses.
HTTP/2 multiplexes multiple streams over a single TCP connection, while gRPC uses Protocol Buffers for efficient data serialization/deserialization.
Load balancing refers to distributing traffic across multiple backend servers to ensure optimal resource utilization and availability.

#### Solution
from First Principles:

HTTP/2 Multiplexing:

HTTP/1.1 required each request to open a separate connection, leading to overhead in establishing and tearing down connections. In contrast, HTTP/2 allows multiple requests to be sent over a single TCP connection using streams.
This multiplexing capability can significantly reduce latency for consecutive requests from the same client, as it avoids re-opening connections.

gRPC Proxying:

gRPC messages are serialized and deserialized using Protocol Buffers, which allows for efficient binary data exchange between services. The API gateway must handle these binary payloads.
When proxying gRPC, the gateway needs to decode incoming requests and encode outgoing responses back into Protocol Buffer format.

Handling Multiplexing in Gateway Design:

For HTTP/2, the gateway should be able to manage a pool of connections and distribute them efficiently among backend services. This can be achieved through connection pooling mechanisms.
In gRPC scenarios, since each service may have its own set of long-lived streams (e.g., bidirectional streaming), the gateway must handle these connections carefully.

Load Balancing Long-Lived Connections:

For HTTP/2 and gRPC both support multiplexing, load balancing in a gateway becomes more complex. The gateway needs to manage a large number of concurrent streams per backend service.
Techniques like round-robin, least-connections, or consistent hashing can be used to distribute these streams across multiple backend instances.

#### Closing Summary
+ Common Mistakes/Follow-Ups: To effectively design an API gateway for HTTP/2 and gRPC, it's crucial to understand the underlying protocols and their implications. Misunderstanding the multiplexing nature of HTTP/2 or not correctly handling Protocol Buffers in gRPC can lead to performance issues or data corruption. Additionally, improper load balancing strategies can result in uneven distribution of traffic, leading to underutilized resources or service downtime.

Some common mistakes include:

Not optimizing for connection pooling and reusing connections.
Failing to properly handle the framing and serialization/ deserialization of Protocol Buffers.
Ignoring the need for advanced load balancing techniques when dealing with a high volume of concurrent streams.

#### Follow-ups
might involve discussing specific implementation details, such as how to implement efficient connection management in the gateway or what kind of metrics should be used to monitor and fine-tune load balancing strategies.

**Comparison:**

| Service | Backend Instances | Load Balancing Strategy |
|---------|------------------|------------------------|
| Service1| 3                | Round-Robin            |
| Service2| 4                | Least-Connections      |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Client] --> B[API Gateway];
    B --> C[Backend Service 1];
    B --> D[Backend Service 2];
```

**Example input / output:**

```text
Input (HTTP/2 Request):
POST /v1/service HTTP/2
Host: example.com
Content-Type: application/grpc

Body:
{ "data": [1, 2, 3] }

Output (HTTP/2 Response):
HTTP/2 200 OK
Content-Type: application/grpc

Body:
{ "result": [4, 5, 6] }
```

**Data:**

```json
{
  "client_request": {
    "method": "POST",
    "path": "/v1/service",
    "headers": {"content-type": "application/grpc"},
    "body": { "data": [1, 2, 3] }
  },
  "gateway_response": {
    "status_code": 200,
    "headers": {"content-type": "application/grpc"},
    "body": { "result": [4, 5, 6] }
  }
}
```

**Flow:**

1. Client Request
2. API Gateway
3. Backend Service (HTTP/2)
4. Backend Service (gRPC)
5. Response Handling & Pooling
6. Stream Management &Response Handling

