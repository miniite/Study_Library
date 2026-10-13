---
type: "practice"
difficulty: "Intermediate"
---

# How would you gracefully handle a client disconnect in a FastAPI WebSocket endpoint and clean up resources?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To gracefully handle a client disconnect in a FastAPI WebSocket endpoint, we need to ensure that any resources associated with the WebSocket are properly cleaned up when the connection is unexpectedly terminated or closed by the client. This involves registering an event handler for the on_disconnect event and ensuring that the cleanup process is idempotent.

#### Triage

The question is asking about handling a client disconnect in a FastAPI WebSocket endpoint, which implies understanding how to manage asynchronous operations and resource management in websockets. The interviewer is probing your knowledge of FastAPI's WebSocket capabilities, event handling mechanisms, and best practices for resource cleanup. We will make the following assumptions:

The WebSocket connection can be gracefully terminated by the client.
Resources associated with a WebSocket session need to be cleaned up.

#### First-Principles Solution

In FastAPI, we can handle a client disconnect by registering an on_disconnect event handler. This handler is called when a WebSocket connection is unexpectedly closed or terminated. We will use a combination of asynchronous context managers and state management to clean up resources. Here’s the step-by-step solution:

Define a WebSocket endpoint: Create a FastAPI route that handles WebSocket connections.
Register an on_disconnect event handler: This function will be called when the client disconnects.
Use async context managers: Ensure any external resources are properly managed and released.
Code Example

Let's start by defining our WebSocket endpoint and handling the on_disconnect event:

PYTHON
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from starlette.websockets import WebSocketState

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()

    # Register on_disconnect handler
    async with register_on_disconnect(websocket) as cleanup_task:
        while True:
            try:
                message = await websocket.receive_text()
                print(f"Received message: {message}")
            except WebSocketDisconnect:
                break

    await cleanup_task  # Ensure the cleanup task runs after disconnection

async def register_on_disconnect(websocket: WebSocket):
    state = None
    async with asyncio.Lock():
        if not hasattr(websocket, 'disconnect_cleanup'):
            websocket.disconnect_cleanup = asyncio.create_task(run_cleanup())

    return websocket.disconnect_cleanup

async def run_cleanup():
    # Simulate cleanup process
    print("Cleanup task running")
    await asyncio.sleep(1)
    print("Cleanup completed")

#### Closing Summary
and Common Mistakes/Follow-Ups
Graceful Disconnection Handling: Ensure the on_disconnect event handler is correctly set up to handle unexpected disconnections.
Resource Management: Use context managers or locks to manage shared resources properly.
Async/Await Patterns: Understand how to use async with and await effectively in asynchronous functions.

#### Common mistakes
include failing to register the disconnect handler, not handling the WebSocketDisconnect exception properly, or forgetting to clean up resources before the next request.

Closing Thoughts

Handling client disconnections gracefully is crucial for maintaining application stability. By leveraging FastAPI's built-in mechanisms and best practices in async programming, we can ensure that our WebSocket endpoints are robust and efficient.

