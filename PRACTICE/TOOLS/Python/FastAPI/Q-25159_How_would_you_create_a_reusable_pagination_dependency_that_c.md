---
type: "practice"
difficulty: "Intermediate"
---

# How would you create a reusable pagination dependency that can be shared across many list endpoints in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cursor Pagination](<../../../../TOOLS/Python/FastAPI/Cursor_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for creating a reusable pagination dependency in FastAPI, let's start by understanding what this entails and then walk through the steps to implement it from first principles.

Firstly, pagination is a common requirement when dealing with large datasets that don't fit within a single HTTP response. Instead of fetching all data at once, we fetch a limited amount (a "page") of data based on specified page number and limit. A reusable dependency in FastAPI can encapsulate this logic so it can be applied consistently across different endpoints.

#### Triaging the Question

The core of the question is about creating a generic pagination mechanism that can be reused across multiple list endpoints. This means we need to ensure our solution is flexible enough to work with various data sources and query parameters, yet simple enough to use seamlessly in different contexts.

#### Assumptions
The application uses FastAPI as the web framework.
Each endpoint that requires pagination has a similar structure for its responses (e.g., it includes items and metadata about pagination).
We will assume a standard pagination parameter set (page and limit).
Step-by-Step Solution
1. Define the Dependency

A dependency in FastAPI can be defined using the Depends keyword, which allows us to inject dependencies into our route functions.

PYTHON
from fastapi import Depends, Query, HTTPException
from pydantic import BaseModel
from typing import Optional, List
2. Create a Pagination Model

We need a model that represents the pagination parameters and their defaults.

PYTHON
class PageParams(BaseModel):
    page: int = 1
    limit: int = 10

    def offset(self) -> int:
        return (self.page - 1) * self.limit
3. Implement a Pagination Dependency Function

This function will extract the pagination parameters from the query and handle any validation or error cases.

PYTHON
def get_page_params(page: int = Query(1, alias="page"), limit: int = Query(10, alias="limit")) -> PageParams:
    return PageParams(page=page, limit=limit)
4. Use the Dependency in a Route

Now we can use this dependency function to apply pagination logic in our route functions.

PYTHON
from fastapi import APIRouter, Depends

router = APIRouter()

@router.get("/items")
async def list_items(page_params: PageParams = Depends(get_page_params)):
    # Fetch items from the database or any data source here using page_params.offset() and page_params.limit()

    items = []  # Placeholder for actual item fetching
    total_count = 100  # Assume there are 100 items in total

    return {
        "items": items,
        "total_pages": (total_count + page_params.limit - 1) // page_params.limit,
        "page_number": page_params.page,
        "limit_per_page": page_params.limit
    }

#### Closing Summary
and Common Mistakes

This approach ensures that pagination logic is decoupled from the business logic of each endpoint, making it easier to maintain and test. However, there are a few common pitfalls:

Incorrect Default Values: Ensure default values for page and limit are appropriate for your use case.
Edge Cases Handling: Consider what happens when page or limit is negative, zero, or exceeds the maximum allowed value.
Error Handling: Properly handle cases where no items are found or an unexpected error occurs during data fetching.

#### Example
Dry-Run

Let's simulate a request and response:

PLAINTEXT
Request:
GET /items?page=2&limit=15

Response:
{
    "items": [item_1, item_2, ..., item_15],
    "total_pages": 7,
    "page_number": 2,
    "limit_per_page": 15
}

#### Common Mistakes/Follow-Ups
Ensure the dependency function is tested independently.

#### Cons
ider pagination for nested or complex data structures.
Handle caching strategies if applicable to improve performance.

**Code:**

```python
from fastapi import Depends, Query, HTTPException
from pydantic import BaseModel
from typing import Optional, List

class PageParams(BaseModel):
    page: int = 1
    limit: int = 10

    def offset(self) -> int:
        return (self.page - 1) * self.limit

def get_page_params(page: int = Query(1, alias="page"), limit: int = Query(10, alias="limit")) -> PageParams:
    return PageParams(page=page, limit=limit)

from fastapi import APIRouter, Depends

router = APIRouter()

@router.get("/items")
async def list_items(page_params: PageParams = Depends(get_page_params)):
    items = []  # Placeholder for actual item fetching
    total_count = 100  # Assume there are 100 items in total

    return {
        "items": items,
        "total_pages": (total_count + page_params.limit - 1) // page_params.limit,
        "page_number": page_params.page,
        "limit_per_page": page_params.limit
    }
```

**Flow:**

1. Yes
2. No
3. Yes
4. Receive Request
5. Is pagination required?
6. List Items with Pagination
7. Are there more items to fetch?
8. Return Response
9. List Next Page of Items

