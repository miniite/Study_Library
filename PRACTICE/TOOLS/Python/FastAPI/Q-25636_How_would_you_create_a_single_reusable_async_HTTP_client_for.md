---
type: "practice"
difficulty: "Intermediate"
---

# How would you create a single reusable async HTTP client for LLM calls in FastAPI lifespan instead of one per request?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand how to create an asynchronous HTTP client that is reused across multiple requests within the lifecycle of a FastAPI application rather than creating a new instance per request. This approach can significantly improve performance and resource management.

#### Triage

The interviewer is probing your understanding of asynchronous programming in FastAPI, the concept of dependency injection, and the ability to manage resources efficiently. We assume that the LLM (Language Model) calls are frequent enough to justify this optimization. The key idea here is leveraging FastAPI's lifespan events or global state management.

#### First-Principles Solution

First, let’s outline the steps:

Define the HTTP Client: Create a reusable asynchronous client using httpx which supports async requests.
Integrate with FastAPI Lifespan Events: Use the on_startup and on_shutdown events to initialize and close the client properly.
Dependency Injection: Ensure the client is passed as a dependency into your API routes.
Step-by-Step Solution
1. Define the HTTP Client

We will use the httpx.AsyncClient for our asynchronous HTTP calls. This client supports reusability and is compatible with FastAPI.

PYTHON
from fastapi import FastAPI, Depends
import httpx

app = FastAPI()

# Define a global variable to store the AsyncClient
client = None

class Client:
    def __init__(self):
        self.client = httpx.AsyncClient()

async def get_client() -> httpx.AsyncClient:
    return client.client

@app.on_event("startup")
async def startup_event():
    global client
    client = Client()
    print("Client initialized")

@app.on_event("shutdown")
async def shutdown_event():
    await client.client.aclose()
    print("Client closed")
2. Use Dependency Injection in API Routes

Now, we will inject the get_client dependency into our route handlers.

PYTHON
from fastapi import APIRouter

router = APIRouter()

@router.get("/llm/{query}")
async def call_llm(query: str, client: httpx.AsyncClient = Depends(get_client)):
    response = await client.post("https://your-llm-endpoint.com/api", json={"query": query})
    return response.json()
3. Testing the Route

We can test this route by calling it multiple times and checking if the client is reused.

BASH
curl -X GET "http://127.0.0.1:8000/llm/query1"
curl -X GET "http://127.0.0.1:8000/llm/query2"

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that the HTTP client is reused across multiple requests, improving performance by avoiding the overhead of creating a new client instance for each request. The httpx.AsyncClient handles retries and timeouts automatically, making it robust.

#### Common Mistakes

Not properly closing the client in the shutdown event.
Forgetting to initialize the client in the startup event.
Incorrectly managing the lifecycle of the async client.

If you were asked about additional features or optimizations, you might discuss handling retries, caching responses, or integrating with FastAPI middleware for better error handling and logging.

**Code:**

```python
from fastapi import FastAPI, Depends
import httpx

app = FastAPI()

class Client:
    def __init__(self):
        self.client = httpx.AsyncClient()

async def get_client() -> httpx.AsyncClient:
    return client.client

@app.on_event("startup")
async def startup_event():
    global client
    client = Client()
    print("Client initialized")

@app.on_event("shutdown")
async def shutdown_event():
    await client.client.aclose()
    print("Client closed")

@router.get("/llm/{query}")
async def call_llm(query: str, client: httpx.AsyncClient = Depends(get_client)):
    response = await client.post("https://your-llm-endpoint.com/api", json={"query": query})
    return response.json()
```

**Example input / output:**

```text
# Sample output from the first request
{"response": "Sample LLM Response 1"}

# Sample output from the second request
{"response": "Sample LLM Response 2"}
```

