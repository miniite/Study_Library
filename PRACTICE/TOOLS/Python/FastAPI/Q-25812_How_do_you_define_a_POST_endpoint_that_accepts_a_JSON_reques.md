---
type: "practice"
difficulty: "Basic"
---

# How do you define a POST endpoint that accepts a JSON request body in FastAPI?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define a POST endpoint in FastAPI that accepts a JSON request body, we'll start by understanding the basics of what FastAPI is and how it works. FastAPI is a modern web framework for building APIs with Python 3.7+ using type hints, which allows for automatic OpenAPI generation and interactive API docs. Here's a step-by-step guide to setting up this endpoint.

Firstly, let's break down the question: we need to create a POST route that will accept JSON data in its request body. The process involves defining the schema of the incoming data, creating the endpoint function, and testing it. We'll use simple types for demonstration purposes, but in real-world scenarios, more complex schemas might be needed.

#### Assumptions

You have FastAPI installed and a basic understanding of Python.
You are familiar with using Python's type hints to define schemas.

Now let's dive into the implementation:

Define the Schema: We need to specify the structure of the JSON data we expect in the request body. This is done by defining a Pydantic model (a class with type annotations).

Create the Endpoint Function: The function will be decorated as an endpoint, accepting the defined schema as its argument.

Test the Endpoint: Finally, we'll test our endpoint to ensure it works as expected.

Here's how you can do it:

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

# Define a Pydantic model for the schema of the incoming JSON data
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = 10.5

@app.post("/items/")
async def create_item(item: Item):
    # This function will be called when a POST request is made to /items/
    return {"received": item.dict()}

This setup ensures that the create_item function only processes requests with JSON bodies matching the Item schema. If the incoming data doesn't match, FastAPI will automatically generate an error response.

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = 10.5

@app.post("/items/")
async def create_item(item: Item):
    return {"received": item.dict()}
```

**Example input / output:**

```

To test the endpoint, you can send a POST request with JSON data:

JSON
Copy
{
    "name": "Foo",
    "description": "A very nice Item",
    "price": 35.4,
    "tax": 10.5
}

The expected response should be:

JSON
Copy
{
    "received": {
        "name": "Foo",
        "description": "A very nice Item",
        "price": 35.4,
        "tax": 10.5
    }
}
```

