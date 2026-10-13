---
type: "practice"
difficulty: "Deep"
---

# How do you implement Server-Sent Events for a live updates feed in FastAPI and when is SSE better than WebSockets?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To implement Server-Sent Events (SSE) in FastAPI, we need to use the FastAPI framework's capabilities along with a custom event stream handler. SSE is better than WebSockets when you need to push updates from the server to the client without requiring a full page reload and where bidirectional communication isn't necessary.

#### Triage
The question asks us to implement an SSE mechanism in FastAPI for live updates, while also comparing it with WebSockets. We need to understand the core concepts of both technologies and their use cases before diving into the implementation details.

#### First-Principles Solution

Understanding Server-Sent Events (SSE):

SSE is a simple protocol that allows servers to push updates to clients via HTTP.
It uses long-lived HTTP connections, and the client can request periodic data updates from the server.
SSE works well in scenarios where real-time updates are needed but only from the server to the client.

Setting Up FastAPI for SSE:

We'll define a route that returns an event stream using FastAPI.
The client will listen to this endpoint, and we’ll push events to it over time.

Code Implementation:

PYTHON
from fastapi import FastAPI, WebSocket
import asyncio

app = FastAPI()

@app.get("/sse")
async def sse_feed():
    async def event_generator():
        for i in range(10):
            await asyncio.sleep(2)  # Simulate delay
            yield f"data: {i}\n\n"

    return {"stream": event_generator()}

Client-Side Handling (Simplified):

The client will use JavaScript to listen to the SSE stream.
JAVASCRIPT

#### cons
t eventSource = new EventSource("/sse");
eventSource.onmessage = function(event) {

#### cons
ole.log("Message received:", event.data);
};

When to Use SSE vs WebSockets:

SSE is better when:
The server needs to push updates without needing a full page reload.
There’s no need for bidirectional communication between the client and server.
The application doesn’t require real-time, interactive experiences like multiplayer games or chat applications.
WebSockets are preferred in scenarios where:
Real-time bidirectional communication is required (e.g., chat apps, online gaming).
Lower latency is crucial for performance-sensitive applications.

#### Closing Summary

We've seen how to implement SSE in FastAPI by defining a custom event stream handler.
SSE is suitable for scenarios where the server pushes updates to the client without requiring bidirectional communication or real-time, interactive experiences.

#### Common Mistakes/Follow-Ups

Mistake 1: Forgetting to handle exceptions and closing connections properly in asynchronous code.

#### Follow-Up
Discussing how to manage long-lived connections more robustly using WebSocket for scenarios that require bidirectional communication.

**Comparison:**

| Scenario | SSE            | WebSocket       |
|----------|----------------|----------------|
| Pushing updates from server to client | ✔️              | ✔️              |
| Bidirectional communication | ❌              | ✔️              |
| Real-time interaction (e.g., chat) | ❌              | ✔️              |
| Lower latency requirement | ❌              | ✔️              |

**Code:**

```python
from fastapi import FastAPI, WebSocket
import asyncio

app = FastAPI()

@app.get("/sse")
async def sse_feed():
    async def event_generator():
        for i in range(10):
            await asyncio.sleep(2)  # Simulate delay
            yield f"data: {i}\n\n"

    return {"stream": event_generator()}
```

**Example input / output:**

```text
# Example input and output for the FastAPI SSE route

input:
GET /sse HTTP/1.1
Host: localhost:8000

output:
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: 0
\n\n
data: 1
\n\n
...
data: 9
\n\n
```

**Data:**

```json
{
  "event_source": {
    "url": "/sse",
    "client_code": "const eventSource = new EventSource('/sse');\neventSource.onmessage = function(event) {\ncolor.log('Message received:', event.data);\n};"
  }
}
```

**Flow:**

1. Rendering diagram…

