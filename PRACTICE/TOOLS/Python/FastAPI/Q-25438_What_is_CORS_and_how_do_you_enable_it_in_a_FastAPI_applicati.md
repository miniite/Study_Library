---
type: "practice"
difficulty: "Intermediate"
---

# What is CORS and how do you enable it in a FastAPI application?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

CORS (Cross-Origin Resource Sharing) is a security feature that allows web applications to make cross-domain requests while ensuring client-side JavaScript can only access specific resources from designated domains. In the context of a FastAPI application, enabling CORS helps by configuring which origins are allowed to request data from your API.

To enable CORS in a FastAPI application, you need to install starlette-cors, an optional dependency for FastAPI, and then configure it according to your needs. Let’s break down this process step by step.

#### Triaging the Question

The question is asking about how to implement CORS in a FastAPI application. The interviewer might be probing into:

Your understanding of what CORS does.
How you would integrate starlette-cors with FastAPI.
Any best practices or potential pitfalls related to enabling CORS.

#### First Principles Solution

First, ensure that starlette-cors is installed in your project:

BASH
pip install starlette-cors

Next, import and configure the middleware in your FastAPI application. You can do this by creating a custom CORS object from starlette.middleware.cors.

Here’s an example of how to enable CORS for your FastAPI app:

Install Dependencies:

BASH
pip install fastapi uvicorn starlette-cors

Create Your FastAPI Application:

PYTHON
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Enable CORS for all origins
origins = ["*"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def read_root(request: Request):
    return {"message": "Hello, World!"}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, enabling CORS in a FastAPI application involves using the CORSMiddleware provided by Starlette. The middleware allows you to specify which origins are allowed to access your API, as well as other settings such as allowing credentials or specific HTTP methods.

#### Common Mistakes

Forgetting to install starlette-cors.
Not properly configuring the allowed origins.
Disabling necessary security features like allow_credentials if not needed.

For follow-ups, the interviewer might ask:

What are the implications of setting origins=["*"]?
How can you fine-tune CORS settings for more granular control over different domains and methods?

**Code:**

```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Enable CORS for all origins
origins = ["*"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def read_root(request: Request):
    return {"message": "Hello, World!"}
```

**Example input / output:**

```text
# Example input and output
request = Request({"method": "GET", "url": "/"})
response = await read_root(request)
print(response)  # Output: {'message': 'Hello, World!'}
```

