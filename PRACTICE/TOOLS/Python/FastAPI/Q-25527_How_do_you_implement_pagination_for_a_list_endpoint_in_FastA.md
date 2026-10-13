---
type: "practice"
difficulty: "Intermediate"
---

# How do you implement pagination for a list endpoint in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cursor Pagination](<../../../../TOOLS/Python/FastAPI/Cursor_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Pagination is a common requirement when dealing with large datasets, and FastAPI provides several built-in features to handle this efficiently. In this solution, we'll walk through how to implement pagination using FastAPI by leveraging its routing mechanisms and query parameters.

#### Triage

The question asks for implementing pagination in a list endpoint of a FastAPI application. The interviewer is probing your understanding of:

How to define routes in FastAPI.
Use of query parameters.
Handling slicing and pagination logic.
Error handling and validation.

We will assume that you have a basic understanding of FastAPI, and the data model is already defined with an ID field for identification purposes.

#### First-Principles Solution
Define the Route: We need to define a route in our FastAPI application that handles GET requests.
Query Parameters: Use query parameters to get the page and items_per_page from the user.
Pagination Logic: Slice the data based on the page number and items per page.
Error Handling: Ensure graceful handling of invalid inputs.

Let's start with a simplified data model for demonstration purposes:

PYTHON
from fastapi import FastAPI, Depends, Query
from pydantic import BaseModel

app = FastAPI()

# Assuming we have a list of items as our dataset
items = [{"id": i, "name": f"Item {i}"} for i in range(100)]

#### Implementation
Steps

Define the Route:

PYTHON
@app.get("/items/")
async def get_items(page: int = Query(default=1, ge=1), items_per_page: int = Query(default=5, ge=1)):
    start_index = (page - 1) * items_per_page
    end_index = min(start_index + items_per_page, len(items))
    paginated_items = items[start_index:end_index]
    return {"items": paginated_items}

Explanation:

page and items_per_page are query parameters with default values.
The formula (page - 1) * items_per_page calculates the starting index for slicing the list.
We ensure that we don't exceed the bounds of the data list by using a min function.
Full Solution
PYTHON
from fastapi import FastAPI, Depends, Query
from pydantic import BaseModel

app = FastAPI()

# Assuming we have a list of items as our dataset
items = [{"id": i, "name": f"Item {i}"} for i in range(100)]

@app.get("/items/")
async def get_items(page: int = Query(default=1, ge=1), items_per_page: int = Query(default=5, ge=1)):
    start_index = (page - 1) * items_per_page
    end_index = min(start_index + items_per_page, len(items))
    paginated_items = items[start_index:end_index]
    return {"items": paginated_items}

#### Closing Summary
and Common Mistakes

This implementation handles pagination by slicing the list based on user-defined page and items_per_page. It ensures that only a subset of the data is returned, making the response more manageable. Some common mistakes to avoid include:

Not handling invalid input for page or items_per_page.
Failing to slice correctly, which could lead to incorrect pagination.
Overlooking edge cases like when there are fewer items than requested.

#### Common Mistakes
Invalid Input Handling: Ensure that the query parameters are within valid ranges (e.g., page number should be at least 1 and items_per_page should be at least 1).
Edge Cases: Handle scenarios where the total number of items is less than the requested items_per_page.

**Comparison:**

```
Input	Output
GET /items/?page=2&items_per_page=5	{"items": [{"id": 10, "name": "Item 10"}, {"id": 11, "name": "Item 11"}, {"id": 12, "name": "Item 12"}, {"id": 13, "name": "Item 13"}, {"id": 14, "name": "Item 14"}]}
```

**Code:**

```python
from fastapi import FastAPI, Depends, Query
from pydantic import BaseModel

app = FastAPI()

# Assuming we have a list of items as our dataset
items = [{"id": i, "name": f"Item {i}"} for i in range(100)]

@app.get("/items/")
async def get_items(page: int = Query(default=1, ge=1), items_per_page: int = Query(default=5, ge=1)):
    start_index = (page - 1) * items_per_page
    end_index = min(start_index + items_per_page, len(items))
    paginated_items = items[start_index:end_index]
    return {"items": paginated_items}
```

**Example input / output:**

```text
Request:
GET /items/?page=2&items_per_page=5

Response:
{
  "items": [
    { "id": 10, "name": "Item 10" },
    { "id": 11, "name": "Item 11" },
    { "id": 12, "name": "Item 12" },
    { "id": 13, "name": "Item 13" },
    { "id": 14, "name": "Item 14" }
  ]
}
```

**Data:**

```json
{
  "items": [
    { "id": 0, "name": "Item 0" },
    { "id": 1, "name": "Item 1" },
    // ... (98 more items)
    { "id": 99, "name": "Item 99" }
  ]
}
```

**Flow:**

1. GET /items/ endpoint
2. (page - 1) * items_per_page
3. min(start_index +items_per_page, len(items))
4. paginated_items =items[start_index:end_index]
5. Return paginated_items

