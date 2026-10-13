---
type: "practice"
difficulty: "Intermediate"
---

# Why might you prefer Server-Sent Events over a plain JSON response for streaming LLM tokens from FastAPI to a browser?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether Server-Sent Events (SSE) are preferable over plain JSON responses in FastAPI when streaming Large Language Model (LLM) tokens to a browser, let's first understand what SSE is and its benefits.

#### Overview
of the Question: This question is asking about the practical advantages of using Server-Sent Events (SSE) compared to regular HTTP requests for sending real-time data from an API like FastAPI. The core idea is to highlight scenarios where SSE can outperform plain JSON responses, particularly in terms of performance and resource efficiency.

#### Triage
The question aims to understand the trade-offs between these two methods. We need to consider factors such as network bandwidth usage, client-server latency, and the complexity of implementation. Key assumptions include the use of FastAPI for the server-side logic and a modern browser on the client side.

#### First-Principles Solution

Understanding SSE vs JSON:

SSE: Allows the server to push data to the client over a single HTTP connection, with minimal overhead.
JSON Response: Involves setting up a new HTTP request for each chunk of data sent from the server.

Latency and Bandwidth:

SSE reduces latency by maintaining an open connection, which means fewer round trips between the client and server.
JSON responses require establishing a new connection for each chunk, increasing latency.

Resource Efficiency:

SSE uses less bandwidth because it operates over a single persistent HTTP connection, reducing overhead compared to multiple requests.
JSON responses can lead to higher bandwidth usage as each request has its own headers and body, which adds to the data transferred per message.

#### Implementation
Complexity:

SSE is straightforward to implement in FastAPI since it’s built on top of Python's ASGI framework, supporting WebSockets and Event Source (SSE) out-of-the-box.
JSON responses are more common but require setting up multiple HTTP requests, which can be less efficient.

Use Case Scenarios:

SSE is ideal for real-time applications like chat or long-running tasks where frequent updates are required without high network usage.
JSON might be preferred in cases where the application requires more control over request handling or when dealing with one-off data fetches.

#### Closing Summary
and Common Mistakes/Follow-Ups: In conclusion, Server-Sent Events can provide a more efficient and lower-latency solution for streaming LLM tokens compared to regular JSON responses. However, it's crucial to consider the specific use case requirements before deciding which method to implement. For example, if your application needs high-frequency updates and low latency, SSE might be preferable; otherwise, plain JSON could suffice.

#### Common mistakes
include overestimating the complexity of implementing SSE or underestimating the benefits of reducing round trips in terms of performance and resource usage. Always evaluate both methods against the specific requirements of your project to make an informed decision.

**Comparison:**

```
Scenario	SSE	JSON
Real-Time Updates	High	Lower
Bandwidth Usage	Low	Higher
Complexity	Low	Medium-High
Use Case	Chat, Long-Running Tasks	One-Off Data Fetches
```

**Code:**

```python
# Example FastAPI implementation with Server-Sent Events
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/sse")
async def sse_endpoint(websocket: WebSocket):
    await websocket.accept()

    # Simulate LLM token streaming
    for i in range(100):  # 100 tokens
        await websocket.send_text(f"Token {i}")
```

**Example input / output:**

```

Example input-output for streaming tokens using SSE:

Client Request: GET /sse
Server Response:
PLAINTEXT
Copy
EventSource: {"data": "Token 0"}
EventSource: {"data": "Token 1"}
...
EventSource: {"data": "Token 99"}
```

**Data:**

```json
{
  "tokens": [
    "Token 0",
    "Token 1",
    "Token 2",
    ...
    "Token 99"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Client Connects
4. Is SSE Used?
5. Open Persistent Connection
6. Send Each Token Over ThisConnection
7. Establish New HTTP Requestfor Each Token
8. Close Connection After AllTokens Sent
9. Handle Multiple Requests andResponses

