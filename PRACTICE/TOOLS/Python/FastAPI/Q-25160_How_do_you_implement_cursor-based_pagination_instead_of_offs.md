---
type: "practice"
difficulty: "Deep"
---

# How do you implement cursor-based pagination instead of offset pagination in a FastAPI list endpoint?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cursor Pagination](<../../../../TOOLS/Python/FastAPI/Cursor_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To implement cursor-based pagination in a FastAPI list endpoint, we need to maintain state between requests using cursors or tokens. This approach ensures efficient and scalable data retrieval without relying on offsets, which can lead to performance issues with large datasets.

#### Triaging the Question

The question is asking how to replace offset-based pagination with cursor-based pagination in a FastAPI API. The primary goal here is to ensure that we handle infinite scrolling or paginated lists more efficiently by maintaining state between requests through cursors or tokens. This method helps in fetching the next set of records without having to scan all previous items, making it suitable for large datasets.

#### Assumptions
and First-Principles Reasoning
State Between Requests: We need a way to keep track of the last item seen in the previous request.
Cursor Tokens: These are unique identifiers or hashes that represent specific positions in the data set.
Efficiency: Cursor-based pagination should be more efficient, especially for large datasets.
Step-by-Step Solution
1. Define the Cursor Token

A cursor token can be a primary key or any unique identifier of an item in your database. For simplicity, let's assume we're using the id field as our cursor.

PYTHON
class Item(Base):
    id: int = Field(..., primary_key=True)
    name: str
2. FastAPI Endpoint for List and Cursor

We'll define a FastAPI endpoint that accepts a query parameter cursor. This parameter will be used to fetch the next set of items based on the cursor provided.

PYTHON
from fastapi import FastAPI, Query
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str

items = {
    i.id: {"id": i.id, "name": f"Item {i.id}"}
    for i in [Item(id=1), Item(id=2), Item(id=3), ...]
}

@app.get("/items/")
async def list_items(cursor: Optional[int] = None):
    if cursor:
        next_items = {}
        # Logic to fetch items after the given cursor
        for item_id, item_data in items.items():
            if item_id > cursor:
                next_items[item_id] = item_data
        return next_items
    else:
        return items  # Return all items if no cursor is provided
3. Handling Edge Cases and Failure Modes
Empty Cursor: If the cursor value is invalid or does not exist, we should handle this gracefully by returning an empty list.
Large Datasets: In real scenarios with large datasets, consider using pagination tokens that include additional metadata like a timestamp to ensure uniqueness.
4. Performance Considerations

For very large databases, we can use more advanced techniques such as secondary indexes or cursor-based queries directly in the database query itself:

SQL
SELECT * FROM items WHERE id > :cursor ORDER BY id LIMIT 10;

This SQL query ensures that only the relevant records are fetched from the database.

#### Closing Summary

In summary, implementing cursor-based pagination involves maintaining state between requests through unique identifiers or tokens. This approach is more efficient and scalable than offset-based pagination, especially for large datasets. By using FastAPI and custom logic, we can easily implement this feature while handling edge cases and ensuring performance.

#### Common Mistakes
and Follow-Ups
Overlooking Edge Cases: Not handling invalid or missing cursor values.
Ignoring Performance: Failing to consider database-specific optimizations that can significantly improve performance.
Not Considering Token Expiry: In scenarios with high transaction volumes, tokens might need expiration times to avoid memory leaks.

**Code:**

```python
from fastapi import FastAPI, Query
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str

items = {
    i.id: {"id": i.id, "name": f"Item {i.id}"}
    for i in [Item(id=1), Item(id=2), Item(id=3)]
}

@app.get("/items/")
async def list_items(cursor: Optional[int] = None):
    if cursor:
        next_items = {}
        for item_id, item_data in items.items():
            if item_id > cursor:
                next_items[item_id] = item_data
        return next_items
    else:
        return items  # Return all items if no cursor is provided
```

**Example input / output:**

```text
# Example Input
{
  "cursor": 2
}

# Example Output
{
  "3": {"id": 3, "name": "Item 3"}
}
```

