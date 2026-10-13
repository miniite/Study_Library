---
type: "practice"
difficulty: "Intermediate"
---

# How do you group related endpoints together using APIRouter in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with organizing related endpoints into separate modules to keep our FastAPI application modular and maintainable. The key concept here is using APIRouter from FastAPI, which allows us to group multiple endpoints under a single namespace. This approach not only enhances code organization but also eases testing and deployment.

#### Triaging the Question

The question essentially asks how to use APIRouter in FastAPI for grouping related endpoints. The interviewer might be probing your understanding of:

How to structure modular applications.
The benefits of using routers in FastAPI.
Best practices for organizing code.
Your ability to demonstrate knowledge through a practical example.

#### First Principles Solution

To organize related endpoints, we'll create an APIRouter instance and add our routes to it. Then, this router will be mounted under a specific path in the main application, making sure that all grouped endpoints have a common namespace. This is particularly useful when dealing with subdomains or different functionalities within the same API.

Step-by-Step Solution
Define the Router: Create an APIRouter for each group of related endpoints.
Add Endpoints to the Router: Define routes and their corresponding handler functions.
Mount the Router in Main Application: Mount the router under a specific path in the main FastAPI application.

Here’s how we can implement this:

PYTHON
from fastapi import APIRouter, Depends

# Step 1: Define the router for user-related endpoints
user_router = APIRouter(prefix="/users")

@user_router.get("/")
async def read_users():
    return [{"username": "johndoe"}, {"username": "janedoe"}]

@user_router.post("/")
async def create_user():
    return {"message": "User created"}

# Step 2: Define the router for post-related endpoints
post_router = APIRouter(prefix="/posts")

@post_router.get("/")
async def read_posts():
    return [{"title": "My first post"}, {"title": "Another post"}]

@post_router.post("/")
async def create_post():
    return {"message": "Post created"}

# Step 3: Mount the routers in the main application
from fastapi import FastAPI

app = FastAPI()

# Mount user router under /users
app.include_router(user_router)

# Mount post router under /posts
app.include_router(post_router)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using APIRouter in FastAPI helps organize related endpoints into separate modules. This approach not only improves code organization but also makes the application more maintainable and testable. Common mistakes include:

Forgetting to prefix routes with a common path.
Not properly importing routers when mounting them.

For follow-ups, you might be asked about performance considerations, how to handle authentication for these grouped endpoints, or ways to manage dependencies within each router module.

**Code:**

```python
from fastapi import APIRouter, Depends

# Define the router for user-related endpoints
user_router = APIRouter(prefix="/users")

@user_router.get("/")
async def read_users():
    return [{"username": "johndoe"}, {"username": "janedoe"}]

@user_router.post("/")
async def create_user():
    return {"message": "User created"}

# Define the router for post-related endpoints
post_router = APIRouter(prefix="/posts")

@post_router.get("/")
async def read_posts():
    return [{"title": "My first post"}, {"title": "Another post"}]

@post_router.post("/")
async def create_post():
    return {"message": "Post created"}

# Mount the routers in the main application
from fastapi import FastAPI

app = FastAPI()

# Mount user router under /users
app.include_router(user_router)

# Mount post router under /posts
app.include_router(post_router)
```

