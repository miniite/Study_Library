---
type: "practice"
difficulty: "Basic"
---

# How does FastAPI automatically generate interactive API documentation and what tools does it use?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

FastAPI is a modern Python web framework that uses Pydantic for data validation and serialization, and Starlette as its ASGI (Application Server Gateway Interface) server. It automatically generates interactive API documentation through a combination of OpenAPI specification and the fastapi.openapi.utils.get_openapi function. This integration allows developers to easily create well-documented APIs without writing additional boilerplate code.

FastAPI uses Pydantic models to define request and response schemas, which are then used to generate the appropriate schema for the OpenAPI documentation. When you run your FastAPI application with uvicorn, it serves both the API endpoints and the documentation simultaneously through a Swagger UI or ReDoc interface.

To dive into the specifics of how this works, let's break it down from first principles.

**Code:**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = 10.5

@app.get("/")
async def read_root():
    return {"Hello": "World"}

@app.post("/items/")
async def create_item(item: Item):
    return item
```

**Example input / output:**

```text
The `Item` model defined in the code is used to validate and serialize incoming requests. When you run the application, the API documentation will automatically be generated based on these models.

For example, if you access `/items/` with a POST request containing an item object, it will be validated according to the schema defined by the `Item` model.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is FastAPI Model Used?
5. Generate OpenAPI Schemafrom Pydantic Models
6. Return Documentation and APIEndpoints
7. No Documentation Generated,Only API Endpoints
8. End

