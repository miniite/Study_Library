---
type: "practice"
difficulty: "Expert"
---

# Your FastAPI service leaks a growing number of TIME_WAIT and open sockets to an upstream API; how do you trace it back to httpx client lifecycle and connection-pool misconfiguration?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Timeouts and Resource Protection](<../../../../TOOLS/Python/FastAPI/Timeouts_and_Resource_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a growing number of TIME_WAIT sockets and open connections in your FastAPI service, we need to dive into the details of the httpx library's client lifecycle and connection pool management. This problem is likely rooted in misconfigured or improperly managed connections within your application.

#### Triage

The core question here is how to trace down a potential issue with the httpx client configuration in FastAPI, which might be causing an accumulation of TIME_WAIT sockets and open connections. The interviewer is probing into your understanding of network protocols, connection management, and debugging techniques. We will assume that the FastAPI service is using the httpx.AsyncClient for making HTTP requests to an upstream API.

#### First-Principles Solution
Understand the Issue: TIME_WAIT sockets represent a state where a TCP connection is being cleaned up but hasn't been completely closed yet. This can lead to resource leaks if not managed properly.
Inspect the Client Configuration: Review how httpx.AsyncClient is instantiated and used in your FastAPI service.
Analyze Connection Pooling: Check if you are reusing connections or creating new ones on each request.
Logging and Debugging: Add detailed logging to track connection states, reuse patterns, and any potential errors.
Detailed Steps

Review Client Configuration:

Ensure that the httpx.AsyncClient is configured with appropriate timeouts and keep-alive settings.
Check if you are using a persistent client instance or creating new clients for each request.

Connection Pool Management:

Understand how connection pooling works in httpx. By default, httpx uses a connection pool to manage HTTP connections efficiently.
Adjust the maximum number of concurrent connections and idle timeout settings if necessary.

Logging and Tracing:

Implement logging at critical points where connections are opened, reused, or closed.
Use tools like Prometheus for monitoring socket usage and connection states.

Testing with Stress Tests:

Run stress tests to simulate high traffic conditions and observe the behavior of TIME_WAIT sockets and open connections.

#### Example
Code
PYTHON
from fastapi import FastAPI
import httpx

app = FastAPI()

# Configure httpx client
client = httpx.AsyncClient(
    base_url="http://upstream-api.com",
    timeout=httpx.Timeout(10.0, connect=5.0),
    http2=True,
)

@app.post("/submit")
async def submit(data: dict):
    try:
        response = await client.post("/endpoint", json=data)
        response.raise_for_status()
        # Process the response
        return {"status": "success", "data": response.json()}
    finally:
        await client.aclose()

# Add logging to track connection states and errors
import logging

logging.basicConfig(level=logging.DEBUG)

@app.on_event("shutdown")
async def shutdown():
    logging.info(f"Closing httpx client: {client}")
    await client.aclose()

#### Closing Summary
and Common Mistakes/Follow-Ups

By carefully reviewing the httpx client configuration, understanding connection pooling, and adding detailed logging, you can trace back the root cause of TIME_WAIT sockets and open connections. Common mistakes include not properly managing client instances, ignoring default settings that might be inadequate for production environments, and insufficient error handling.

In follow-ups, the interviewer might ask about specific tools or libraries to monitor network performance, or how to handle different types of upstream APIs with varying requirements.

**Code:**

```python
from fastapi import FastAPI
import httpx

app = FastAPI()

client = httpx.AsyncClient(
    base_url="http://upstream-api.com",
    timeout=httpx.Timeout(10.0, connect=5.0),
    http2=True,
)

@app.post("/submit")
async def submit(data: dict):
    try:
        response = await client.post("/endpoint", json=data)
        response.raise_for_status()
        return {"status": "success", "data": response.json()}
    finally:
        await client.aclose()

import logging

logging.basicConfig(level=logging.DEBUG)

@app.on_event("shutdown")
async def shutdown():
    logging.info(f"Closing httpx client: {client}")
    await client.aclose()
```

**Example input / output:**

```text
Request:
POST /submit HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{
  "key": "value"
}

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "success",
  "data": {
    "result": "expected_value"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Is client reused?
6. Configure persistent client
7. Monitor connection states
8. Create new client for eachrequest
9. Maintain idle timeout settings
10. Handle shutdown properly
11. Log critical points
12. Add detailed logging
13. Debugging and tracingconnections

