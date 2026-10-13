---
type: "practice"
difficulty: "Intermediate"
---

# How would you define a FastAPI endpoint with path and query parameters, and how are they validated by type hints?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When defining an endpoint in FastAPI, we often need to include both path parameters and query parameters to make our API more flexible and powerful. Path parameters are part of the URL itself and typically define specific resources or actions, while query parameters provide additional filtering options via URL query strings.

#### Triaging the Question

The interviewer is probing your understanding of how to structure endpoints in FastAPI using type hints for validation, ensuring that your code is both readable and robust against invalid inputs. We'll assume you are familiar with basic FastAPI concepts, including defining routes, handling requests, and working with Pydantic models.

#### First-Principles Solution

In a FastAPI application, we define an endpoint by creating a route using the @app.get decorator for HTTP GET requests (similar decorators exist for other methods like POST). To incorporate path parameters, you pass them as arguments to your function. For query parameters, these are automatically extracted from the URL and can be handled with type hints in Python.

#### Example
Defining an Endpoint
PYTHON
from fastapi import FastAPI, Path, Query
from typing import Optional

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int = Path(..., description="The ID of the item to get"), q: Optional[str] = Query(None, alias="query-param", min_length=3)):
    return {"item_id": item_id, "q": q}

In this example:

item_id is a path parameter. The Path annotation includes validation logic (e.g., the value must be an integer).
q is a query parameter. The Query annotation provides default values and additional metadata like description and validation rules.

#### Closing Summary
+ Common Mistakes

In summary, defining FastAPI endpoints with both path and query parameters involves using type hints to validate inputs effectively. Common mistakes include:

Lack of Validation: Not using annotations or validation methods can lead to runtime errors.
Improper Use of Parameters: Mixing up which data should be passed as path vs. query parameters, leading to confusing URLs.
Overlooking Optional Parameters: Forgetting that some query parameters might not always be provided and handling them accordingly.

Always ensure your code is well-documented with clear descriptions for each parameter and thoroughly tested to handle various edge cases and invalid inputs.

**Code:**

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int = Path(..., description="The ID of the item to get"), q: Optional[str] = Query(None, alias="query-param", min_length=3)):
    return {"item_id": item_id, "q": q}
```

