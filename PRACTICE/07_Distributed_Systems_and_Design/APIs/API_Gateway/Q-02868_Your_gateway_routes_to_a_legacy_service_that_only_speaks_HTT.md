---
type: "practice"
difficulty: "Deep"
---

# Your gateway routes to a legacy service that only speaks HTTP/1.1 without keep-alive, and the gateway opened a new TCP+TLS connection per request exhausting ephemeral ports; how do you detect and fix connection reuse?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of routing requests from an API Gateway to a legacy HTTP/1.1 service that doesn't support keep-alive, we need to ensure efficient use of TCP ephemeral ports and maintain connection reuse. The current setup opens a new TLS+TCP connection for every request, which leads to excessive port exhaustion.

#### Triage

The core issue here is connection management between the API Gateway (HTTP 2/3) and the legacy service (HTTP/1.1 without keep-alive). We need to find a way to reuse connections where possible, while ensuring that we handle the limitations of the legacy service gracefully. This involves:

Detecting when the legacy service does not support keep-alive.
Implementing connection pooling or intelligent connection management in the API Gateway.
Handling connection exhaustion and failure modes.

#### Assumptions
The legacy service can only handle one request per connection (no keep-alive).
The API Gateway is using HTTP 2/3, which might support keep-alive by default.
We need to ensure that requests are routed efficiently without exhausting ephemeral ports.

#### Solution
from First Principles
Step-by-Step Approach
Connection Management: Implement a connection pool in the API Gateway to manage and reuse connections to the legacy service.
Detection Mechanism: Develop a mechanism to detect when a request is sent to the legacy service that does not support keep-alive.
Connection Pooling Strategy: Use a strategy that minimizes the number of new connections created, such as reusing an existing connection if available.
Detailed Implementation

Connection Pool Initialization:

Initialize a pool of TCP+TLS connections to the legacy service when the API Gateway starts up or at regular intervals.
PYTHON
import httpx

# Example initialization for 30 concurrent connections
connection_pool = [httpx.AsyncClient(base_url="https://legacy-service.com") for _ in range(30)]

Request Handling:

When a request is received, check if there are any free connections in the pool.
PYTHON
async def handle_request(request):
    # Check if a connection is available in the pool
    free_connection = next((conn for conn in connection_pool if not conn.is_closed), None)

    if free_connection:
        # Use an existing connection from the pool
        response = await free_connection.send(httpx.Request("GET", request.url))
        return response
    else:
        # Create a new connection and use it
        new_conn = httpx.AsyncClient(base_url="https://legacy-service.com")
        try:
            response = await new_conn.send(httpx.Request("GET", request.url))
            # Add the new connection to the pool after handling the request
            connection_pool.append(new_conn)
        finally:
            free_connection.close()
        return response

Connection Reuse and Management:

Ensure that connections are closed properly and added back to the pool when they are no longer needed.
PYTHON
# Example of closing a connection after use
async def close_connections():
    for conn in connection_pool:
        if not conn.is_closed:
            await conn.aclose()

Failure Handling:

Implement robust error handling to manage cases where connections are lost or fail.
PYTHON
async def handle_failure(request, response):
    # Handle retries and failures gracefully
    for attempt in range(3):
        try:
            return await handle_request(request)
        except Exception as e:
            print(f"Connection failure: {e}, retrying...")
    raise Exception("Failed to connect after multiple attempts")

Monitoring and Metrics:

Monitor the health of connections and pool size to ensure optimal performance.
PYTHON
# Example logging metrics
import logging

def monitor_connections():
    for i, conn in enumerate(connection_pool):
        if not conn.is_closed:
            logging.info(f"Connection {i} is active")
        else:
            logging.warning(f"Connection {i} is closed")

monitor_connections()

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a connection pool with intelligent reuse, we can mitigate the issue of excessive ephemeral port exhaustion. However, it's crucial to handle failures gracefully and ensure that connections are managed efficiently.

#### Common Mistakes

Failing to close connections properly after use.
Not handling connection failure modes robustly.
Overloading the API Gateway by not limiting the number of concurrent connections.

#### Follow-Ups

How would you optimize this solution for higher concurrency?
What strategies would you employ if the legacy service had a different keep-alive behavior?

