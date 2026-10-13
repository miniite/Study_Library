---
type: "practice"
difficulty: "Deep"
---

# How do you design a gateway that enforces a global maximum concurrent-connection limit to protect an origin database that can only handle N connections, and how does this differ from per-second rate limiting?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rate Limiting at the Gateway](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an API Gateway that enforces a global maximum concurrent connection limit, we need to ensure that the total number of simultaneous connections to our origin database doesn't exceed its capacity (N). This is crucial to prevent overloading and potential crashes due to excessive load. The approach differs from per-second rate limiting in terms of how it manages the flow of requests, focusing on the overall connection count rather than request frequency.

#### Triage

The interviewer is probing your understanding of API Gateway design principles, specifically around managing concurrent connections. We need to consider several aspects:

Connection Limits: Enforcing a hard limit on the number of open connections.
Load Balancing: Distributing requests across multiple instances if possible.
Failure Handling: Gracefully handling connection exhaustion.
Configuration and Monitoring: Setting up appropriate configurations and monitoring tools.

#### Assumptions
The origin database is a single point of failure; thus, we need to manage connections carefully.
We are dealing with HTTP/HTTPS requests.
The API Gateway should be resilient and capable of managing the global connection limit across multiple backends if needed.

#### Solution
Connection Pooling: Implement connection pooling in the API Gateway to reuse existing connections instead of creating new ones every time a request is made.
Queue Management: Use a message queue to buffer requests when the maximum number of concurrent connections is reached.
Load Balancing: Distribute incoming requests across multiple backend servers if available, thereby spreading the load and reducing individual connection counts.
Graceful Degradation: Implement mechanisms for handling errors gracefully, such as returning 503 Service Unavailable responses when all connections are in use.
Step-by-Step Solution

Connection Pooling

Use a connection pool library like HikariCP or PgBouncer to manage database connections.
Set up the maximum size of the connection pool equal to (N), the maximum number of concurrent connections the origin can handle.

Queue Management

Introduce an external message broker (e.g., RabbitMQ, Kafka) as a buffer for incoming requests.
When the connection limit is reached, new requests are enqueued and processed when a slot becomes available.

Load Balancing

Deploy multiple instances of your application behind a load balancer (e.g., Nginx, HAProxy).
Configure the load balancer to distribute requests among backend servers.
Use sticky sessions if you want to ensure that all requests from a single user go to the same instance.

Graceful Degradation

Implement circuit breaker patterns to detect and handle failures gracefully.
Return 503 Service Unavailable responses when the connection limit is hit to prevent further overload.
Comparison with Per-Second Rate Limiting

Per-second rate limiting focuses on controlling the number of requests per unit time, whereas global maximum concurrent connections manage the total number of open connections. Here’s how they differ:

Scalability: Rate limiting can be more flexible in handling bursts of traffic, but it may not prevent all connection-related issues.
Resource Utilization: Global connection limits ensure that resources are used efficiently by preventing simultaneous overloads, whereas rate limiting might allow too many connections if the burst is high enough.

#### Implementation
Complexity: Managing global connection limits requires more complex state management and queueing mechanisms compared to simple time-based rate limiting.

#### Closing Summary

Enforcing a global maximum concurrent connection limit in an API Gateway involves using connection pooling, message queuing, load balancing, and graceful degradation techniques. These strategies ensure that the origin database remains protected from overload while maintaining system responsiveness during high traffic periods. The design differs significantly from per-second rate limiting, focusing on managing overall connections rather than request frequency.

#### Common Mistakes/Follow-Ups
Ignoring Load Balancing: Failing to distribute requests across multiple instances can lead to unnecessary connection limits being hit.
Not Using Queuing Mechanisms: Buffering requests during high load is crucial for maintaining system stability.
Overlooking Graceful Degradation: Not implementing fail-safe mechanisms can result in complete service unavailability when the limit is reached.

**Comparison:**

| Scenario | Per-Second Rate Limiting | Global Max Concurrent Connections |
|----------|-------------------------|----------------------------------|
| Protection Against Bursts | Yes, but limited | No, may allow bursts if high enough |
| Resource Utilization | Efficient for short-term spikes | Ensures steady resource use |
| Complexity | Simple time-based logic | Complex state management and queuing |

**Code:**

```python
# Example of setting up a connection pool using HikariCP
from hikari import PoolSettings, SqlitePool

pool_settings = PoolSettings(
    driver="sqlite",
    database="/path/to/database.db",
    max_size=100  # Maximum number of concurrent connections
)

connection_pool = SqlitePool(pool_settings)
```

**Example input / output:**

```text
Input:
- HTTP request with a total of 201 requests per second.
- Origin database capacity is N = 100 connections.

Output:
- First 100 requests are processed immediately.
- Requests 101 to 201 are enqueued and processed in the order they were received when slots become available.
```

**Data:**

```json
{
  "connections": {
    "maxSize": 100,
    "queueLimit": 500,
    "timeout": 30000 // milliseconds
  }
}
```

**Flow:**

1. Yes
2. No
3. HTTP Request
4. Connection Pool Full?
5. Enqueue Request
6. Process Request
7. Wait for Slot

