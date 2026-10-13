---
type: "practice"
difficulty: "Intermediate"
---

# How would you expose a WebSocket chat endpoint in FastAPI that streams LLM replies in real time?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To expose a WebSocket chat endpoint in FastAPI that streams LLM (Language Model) replies in real-time, we need to use FastAPI's WebSocket functionality. This involves setting up an endpoint that handles WebSocket connections and integrates with the language model to stream responses back to clients.

#### Triaging the Question

The question is asking for a method to create a WebSocket chat endpoint using FastAPI that can receive requests from clients, process them through a Language Model (LLM), and send real-time updates back to the clients. The interviewer might be probing your understanding of:

How WebSockets work in a FastAPI application.
Integration with an LLM for generating responses.
Handling asynchronous operations in FastAPI.

#### First Principles Solution
Setup FastAPI Application: Start by setting up a basic FastAPI application.
Create WebSocket Endpoint: Define a WebSocket endpoint that accepts connections and manages the communication flow.
Integrate Language Model: Use an LLM to generate responses based on the incoming requests.
Stream Responses: Send the generated responses back to the client in real time.
Step-by-Step Solution
1. Setup FastAPI Application

First, install FastAPI and a WebSocket dependency like starlette-websockets:

SH
pip install fastapi[all] starlette-websockets

Next, set up your FastAPI application:

PYTHON
from fastapi import FastAPI, WebSocket
import asyncio

app = FastAPI()

@app.websocket("/chat")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    # Add logic to handle incoming requests and stream responses here.
2. Create WebSocket Endpoint

Define the /chat endpoint where WebSocket connections will be established:

PYTHON
@app.websocket("/chat")
async def chat(websocket: WebSocket):
    await websocket.accept()

    while True:
        try:
            data = await websocket.receive_text()
            # Call LLM to generate response based on `data`
            response = await call_llm(data)
            await websocket.send_text(response)
        except WebSocketDisconnect:
            break
3. Integrate Language Model (LLM)

Assume you have a function call_llm that interacts with the LLM and returns a response. Here’s an example using a hypothetical language model API:

PYTHON
async def call_llm(message: str) -> str:
    # Simulate calling an LLM to get a response
    import time
    await asyncio.sleep(2)  # Simulating delay
    return f"Response to {message}"
4. Stream Responses

The above code sets up a WebSocket connection and continuously sends responses back to the client based on incoming messages.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Forgetting to handle WebSocketDisconnect exceptions, not properly managing asynchronous operations, or incorrectly handling message streaming.

#### Follow-Ups
The interviewer might ask about scaling the solution, integrating with different LLMs, or optimizing performance.
Final Thoughts

Ensure you cover all aspects of the WebSocket connection lifecycle and handle potential errors gracefully. Understanding how to integrate external services like language models while maintaining real-time communication is key in this scenario.

