---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between returning a Pydantic model, a dict, and an explicit Response object from a FastAPI route?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a FastAPI route to return data, you have multiple options depending on your requirements and how you want to handle responses. The differences between returning a Pydantic model, a dict, or an explicit Response object are significant in terms of validation, type hints, and customization.

Firstly, let's clarify what these options entail:

Returning a Pydantic Model: This involves using FastAPI’s built-in support for Pydantic models to define the structure of your data.
Returning a Dict: Simply returning a Python dictionary from your route handler.
Using an Explicit Response Object: Directly constructing and returning a Response object, giving you full control over the HTTP response.

#### Triaging the Question

The question is asking about the differences in implementation when using these three approaches in FastAPI routes. The interviewer might be probing for understanding of type hints, validation, error handling, and the level of control each method provides.

#### Assumptions
Assume we are working with a simple example where the response needs to include some user data.
We will compare the structure, readability, and flexibility of these three approaches.
Detailed Solution
1. Returning a Pydantic Model

Pydantic models provide a powerful way to validate input data and define a clear schema for your response objects. Here’s an example:

PYTHON
from fastapi import FastAPI
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: str

app = FastAPI()

@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    # Simulate fetching user data from a database or another source.
    return {"id": 1, "name": "John Doe", "email": "john@example.com"}

In this example:

User is defined as a Pydantic model.
The route specifies that the response should be of type User.
This approach ensures that the data returned strictly adheres to the schema defined in User.
2. Returning a Dict

Returning a Python dictionary directly simplifies the code but lacks the validation and structure provided by Pydantic models.

PYTHON
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    # Simulate fetching user data from a database or another source.
    return {"id": 1, "name": "John Doe", "email": "john@example.com"}

In this example:

The route returns a Python dictionary directly.
There is no automatic validation of the response schema.
3. Using an Explicit Response Object

Using Response objects provides full control over the HTTP response. This can be useful for custom headers, status codes, or more complex responses.

PYTHON
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/users/{user_id}", status_code=201)
async def get_user(user_id: int, response: Response):
    # Simulate fetching user data from a database or another source.
    return {"id": 1, "name": "John Doe", "email": "john@example.com"}

In this example:

The route explicitly returns a Response object.
This approach offers the most flexibility but requires more boilerplate code.

#### Closing Summary
and Common Mistakes/Follow-Ups

Returning a Pydantic model is ideal when you need robust validation and clear schema definitions. Using a dict simplifies the code at the cost of reduced type safety and validation. Explicit Response objects provide full control over the response but can be more verbose.

#### Common mistakes
include:

Overlooking validation when using dicts.
Failing to manage HTTP status codes and headers when using explicit responses.

For follow-ups, you might discuss performance implications, error handling strategies, or how these choices affect testing and maintenance.

**Code:**

```python
from fastapi import FastAPI
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: str

app = FastAPI()

@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    return {"id": 1, "name": "John Doe", "email": "john@example.com"}

from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"id": 1, "name": "John Doe", "email": "john@example.com"}

from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/users/{user_id}", status_code=201)
async def get_user(user_id: int, response: Response):
    return {"id": 1, "name": "John Doe", "email": "john@example.com"}
```

**Example input / output:**

```text
Request:
GET /users/1

Response (Pydantic model):
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}

Response (Dict):
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}

Response (Explicit Response object):
HTTP/1.1 201 OK
Content-Type: application/json

{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}
```

