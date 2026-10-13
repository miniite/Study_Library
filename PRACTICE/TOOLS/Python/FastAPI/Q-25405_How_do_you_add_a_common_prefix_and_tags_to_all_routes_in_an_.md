---
type: "practice"
difficulty: "Basic"
---

# How do you add a common prefix and tags to all routes in an APIRouter?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a common prefix and tags to all routes in an APIRouter using FastAPI, we need to understand how FastAPI handles routers and their configurations. FastAPI allows us to create modular APIs by breaking down our API into multiple routers. Each router can have its own route definitions, but there is no out-of-the-box support for adding common prefixes or tags across all routes in a single router.

TL;DR

We can achieve this by creating an APIRouter and using the include_router method to add the prefix and tags to each sub-router. This approach allows us to maintain modularity while applying global configurations consistently.

#### Triaging the Question

The question is asking how to apply a common prefix and tags across all routes defined in an APIRouter. The key here is understanding that FastAPI’s routers are modular, but their behavior can be customized using certain methods. We need to ensure our solution is both efficient and easy to manage.

#### Assumptions
You have basic knowledge of FastAPI and its router system.
You understand the concept of adding tags and prefixes in an API.
You want a clean and maintainable approach without cluttering your codebase with repetitive configurations.

#### Solution
from First Principles

To add a common prefix, we can use the prefix parameter when defining our routers. For tags, FastAPI automatically detects tags based on function names or explicitly using the @api_operation decorator. However, if we want to apply tags globally, we might need to define them manually.

Here’s how you can do it:

Create a base router with common prefix and tags:
Include sub-routers under this base router:
PYTHON
from fastapi import FastAPI, APIRouter

# Create the main application instance
app = FastAPI()

# Base router with common prefix and tags
base_router = APIRouter(prefix="/api/v1", tags=["Base Router Tags"])

# Example sub-router
sub_router = APIRouter(prefix="/subpath", tags=["Sub-Router Tags"])

@base_router.get("/")
async def read_root():
    return {"message": "This is the base router route"}

@base_router.get("/subroute")
async def read_sub_route():
    return {"message": "This is a sub-route under the base router"}

# Include the sub-router in the main app
app.include_router(base_router)

# Optionally, you can include another sub-router
app.include_router(sub_router)

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we created an APIRouter with a common prefix and tags. We then included sub-routers under it to maintain modularity while applying consistent configurations. A common mistake might be not understanding the need for including routers separately in the main application instance.

#### Follow-ups

How would you handle different authentication mechanisms for these routes?
What if you wanted to add different prefixes or tags dynamically based on certain conditions?

**Code:**

```python
from fastapi import FastAPI, APIRouter

# Create the main application instance
app = FastAPI()

# Base router with common prefix and tags
base_router = APIRouter(prefix="/api/v1", tags=["Base Router Tags"])

@base_router.get("/")
async def read_root():
    return {"message": "This is the base router route"}

@base_router.get("/subroute")
async def read_sub_route():
    return {"message": "This is a sub-route under the base router"}

# Example sub-router
sub_router = APIRouter(prefix="/subpath", tags=["Sub-Router Tags"])

@app.get("/")
async def main_root():
    return {"message": "Main application root route"}
```

