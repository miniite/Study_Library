---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a WebSocket connection and a regular HTTP request in a FastAPI app?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a FastAPI application, both WebSocket connections and regular HTTP requests are used to communicate with clients, but they serve different purposes and operate differently under the hood. A WebSocket connection allows bidirectional real-time communication between the client and server, while a regular HTTP request is one-way and typically stateless.

To begin, let's break down what each of these types of connections entails and how they are implemented in FastAPI.

#### Triage
Understanding WebSocket vs Regular HTTP Requests

Question: What is the difference between a WebSocket connection and a regular HTTP request in a FastAPI app?

What the question probes: The interviewer might be interested in understanding your grasp of real-time communication, asynchronous processing, and state management in FastAPI.

#### Assumptions
We assume that you have a basic understanding of HTTP requests and are familiar with FastAPI as an asynchronous web framework.

#### First-Principles Solution
WebSocket Connection

A WebSocket connection establishes a persistent connection between the client and server. This allows for real-time, bidirectional communication where both the client and server can send data at any time without waiting for a request to complete. In FastAPI, this is achieved using the WebSocket class.

How it works: Once established, the WebSocket connection remains open until explicitly closed by either party.

#### Example
use case: Real-time chat applications, live updates in dashboards, or multiplayer games where instant communication is crucial.
Regular HTTP Request

A regular HTTP request is a one-way interaction between the client and server. It involves sending a request (e.g., GET, POST) to the server, which then processes it and sends back a response. The connection closes immediately after the response is sent.

How it works: Each HTTP request is independent and stateless unless cookies or sessions are used.

#### Example
use case: Fetching data from an API, submitting forms, or loading web pages.

#### Implementation
in FastAPI

To implement both types of connections in a FastAPI application:

WebSocket Example
PYTHON
from fastapi import WebSocket, FastAPI

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()

    while True:
        data = await websocket.receive_text()
        print(f"Received message: {data}")

        if data == "close":
            break
Regular HTTP Request Example
PYTHON
from fastapi import FastAPI, Depends

app = FastAPI()

@app.get("/items/")
async def read_items(q: str | None = None):
    return {"q": q}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, WebSocket connections in FastAPI provide real-time, bidirectional communication, ideal for applications requiring continuous interaction. Regular HTTP requests are used for stateless one-way communication, common in typical web interactions.

#### Common mistakes

Forgetting to close the WebSocket connection properly.
Misunderstanding the stateful nature of WebSockets versus the stateless nature of regular HTTP requests.

#### Follow-ups
The interviewer might ask about performance considerations, security aspects, or how you would handle large amounts of data in a real-time application using WebSockets.

**Code:**

```
WebSocket example code

from fastapi import WebSocket, FastAPI

app = FastAPI()

@app.websocket("/ws") async def websocket_endpoint(websocket: WebSocket): await websocket.accept()

CODE
Copy
while True:
    data = await websocket.receive_text()
    print(f"Received message: {data}")

    if data == "close":
        break
CODE
Copy

```python
# Regular HTTP request example code
from fastapi import FastAPI, Depends

app = FastAPI()

@app.get("/items/")
async def read_items(q: str | None = None):
    return {"q": q}
```

