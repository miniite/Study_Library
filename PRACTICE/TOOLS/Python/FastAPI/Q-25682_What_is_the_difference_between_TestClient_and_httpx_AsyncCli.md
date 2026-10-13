---
type: "practice"
difficulty: "Deep"
---

# What is the difference between TestClient and httpx AsyncClient for testing FastAPI apps?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Async vs Sync Endpoints](<../../../../TOOLS/Python/FastAPI/Async_vs_Sync_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between TestClient and httpx AsyncClient in the context of testing FastAPI applications, we need to first grasp their primary purposes, how they operate, and the trade-offs involved.

#### Triage

What is being tested?

Both tools are used for testing FastAPI applications.
The main difference lies in their intended use cases: TestClient is designed specifically for synchronous tests, whereas httpx AsyncClient supports both synchronous and asynchronous testing, making it more versatile.

Key Assumptions:

We assume familiarity with FastAPI, the Python web framework.
Both tools are part of the FastAPI ecosystem but serve different purposes due to their handling of concurrency.

#### First Principles Solution
TestClient

TestClient is a tool provided by FastAPI that simulates client requests in a synchronous manner. It's part of the fastapi.testclient module and is specifically designed for testing FastAPI applications without involving any external HTTP clients or servers. Here’s how it works:

Synchronous Testing: It operates in a synchronous context, meaning all tests are run sequentially.
Mocking Dependencies: Since it runs within the same process as your application code, dependencies can be easily mocked to simulate different scenarios.
Ease of Use: It's straightforward to use for integration and unit testing, leveraging FastAPI’s own routing and dependency injection mechanisms.
httpx AsyncClient

httpx AsyncClient, on the other hand, is a powerful HTTP client that supports both synchronous and asynchronous operations. Here’s what sets it apart:

Concurrent Testing: It can be used for concurrent or asynchronous testing, which is useful when you want to simulate multiple clients hitting your FastAPI service simultaneously.
Real-World Scenarios: You can test how your application handles real-world client requests and network conditions more accurately.
Flexibility: Since it’s not tied to the FastAPI framework specifically, it can be used in a wider variety of testing scenarios.
Full Solution

Setup and Installation

BASH
pip install fastapi[all] httpx pytest

Using TestClient for Synchronous Testing

PYTHON
from fastapi import FastAPI, Depends, HTTPException
from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id not in range(10):
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": f"Item {item_id}"}

client = TestClient(app)
response = client.get("/items/5")
assert response.status_code == 200
assert response.json() == {"item": "Item 5"}

Using httpx AsyncClient for Asynchronous Testing

PYTHON
import asyncio
from fastapi import FastAPI, Depends, HTTPException
import httpx

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in range(10):
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": f"Item {item_id}"}

client = httpx.AsyncClient()
response = await client.get("http://localhost:8000/items/5")
assert response.status_code == 200
assert response.json() == {"item": "Item 5"}
await client.aclose()

# Running tests using pytest-asyncio
import pytest

@pytest.mark.asyncio
async def test_read_item():
    client = httpx.AsyncClient()
    response = await client.get("http://localhost:8000/items/5")
    assert response.status_code == 200
    assert response.json() == {"item": "Item 5"}
    await client.aclose()

#### Closing Summary
and Common Mistakes
Common Mistake: Using TestClient for asynchronous testing where real concurrency is needed. While TestClient is sufficient for synchronous tests, it can lead to false positives or negatives when simulating concurrent requests.
Best Practice: Use httpx AsyncClient for more realistic and concurrent testing scenarios, especially in production-like environments.

By understanding these differences, you can choose the appropriate tool based on your specific testing needs, ensuring robust and accurate validation of your FastAPI application.

**Code:**

```
TestClient example

from fastapi import FastAPI, Depends, HTTPException from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/items/{item_id}") def read_item(item_id: int): if item_id not in range(10): raise HTTPException(status_code=404, detail="Item not found") return {"item": f"Item {item_id}"}

client = TestClient(app) response = client.get("/items/5") assert response.status_code == 200 assert response.json() == {"item": "Item 5"}

CODE
Copy

```python
# httpx AsyncClient example
import asyncio
from fastapi import FastAPI, Depends, HTTPException
import httpx

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in range(10):
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": f"Item {item_id}"}

client = httpx.AsyncClient()
response = await client.get("http://localhost:8000/items/5")
assert response.status_code == 200
assert response.json() == {"item": "Item 5"}
await client.aclose()

# pytest-asyncio example
import pytest

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in range(10):
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": f"Item {item_id}"}

@pytest.mark.asyncio
async def test_read_item():
    client = httpx.AsyncClient()
    response = await client.get("http://localhost:8000/items/5")
    assert response.status_code == 200
    assert response.json() == {"item": "Item 5"}
    await client.aclose()
```

