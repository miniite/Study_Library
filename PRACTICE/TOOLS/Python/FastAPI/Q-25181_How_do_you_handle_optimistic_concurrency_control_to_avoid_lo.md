---
type: "practice"
difficulty: "Expert"
---

# How do you handle optimistic concurrency control to avoid lost updates in a FastAPI CRUD endpoint?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Timeouts and Resource Protection](<../../../../TOOLS/Python/FastAPI/Timeouts_and_Resource_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address optimistic concurrency control (OCC) in a FastAPI CRUD endpoint, we need to ensure that our application prevents "lost updates" by detecting concurrent modifications and handling them appropriately. OCC is particularly useful in scenarios where multiple clients can modify the same data simultaneously, ensuring consistency without requiring explicit locking mechanisms.

In this context, FastAPI provides robust tools for building web APIs, including handling HTTP requests and responses efficiently. Let's break down how to implement OCC from first principles.

First, we need to understand that OCC typically involves checking a version number or timestamp stored in the database when a client updates data. If the version number has changed since the client last fetched it (indicating another update), we reject the request as a "lost update." Otherwise, we proceed with the update.

Here’s a step-by-step approach to implementing OCC in FastAPI:

Fetch Data: When a client requests an item for editing, fetch the latest version of the data along with its version number or timestamp.
Client Modifies Data: The client modifies the fetched data and includes the version number or timestamp as part of the update request.
Server Validation: On the server side, validate that the version number or timestamp in the incoming request matches the current version stored in the database.
Conflict Resolution: If the versions don’t match, return an error indicating a "conflict" and instructing the client to fetch the latest data and retry the update.

#### Triaging the Question

The core of this question is about understanding how to implement optimistic concurrency control using FastAPI. The interviewer might be probing your knowledge of database transactions, version numbers, and how to handle conflicts in a distributed system. Key assumptions include:

You are familiar with RESTful APIs.
You understand HTTP methods (GET/POST/PATCH/DELETE).
You can integrate external libraries if needed.

#### First Principles Solution

Let's consider a CRUD operation where we need to update an item identified by its ID. We'll use a version number for optimistic concurrency control.

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import sqlite3

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str
    version: int  # Version number or timestamp

db_connection = sqlite3.connect("items.db")
cursor = db_connection.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS items (id INTEGER PRIMARY KEY, name TEXT, version INTEGER)")

def get_item_by_id(item_id: int):
    cursor.execute("SELECT * FROM items WHERE id=?", (item_id,))
    result = cursor.fetchone()
    if not result:
        return None
    return {"id": result[0], "name": result[1], "version": result[2]}

@app.get("/items/{item_id}")
def read_item(item_id: int):
    item = get_item_by_id(item_id)
    if item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"id": item["id"], "name": item["name"], "version": item["version"]}

@app.put("/items/{item_id}")
def update_item(item_id: int, item_data: Item):
    fetched_item = get_item_by_id(item_id)
    if fetched_item is None:
        raise HTTPException(status_code=404, detail="Item not found")

    # Client should include version number
    current_version = fetched_item["version"]
    incoming_version = item_data.version

    if current_version != incoming_version:
        return {"detail": "Version conflict detected. Please fetch the latest data and try again."}

    cursor.execute("UPDATE items SET name=?, version=? WHERE id=?", (item_data.name, current_version + 1, item_id))
    db_connection.commit()
    return {"id": item_id, "name": item_data.name, "version": current_version + 1}

@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    cursor.execute("DELETE FROM items WHERE id=?", (item_id,))
    db_connection.commit()
    return {"detail": f"Item {item_id} deleted"}

#### Closing Summary
and Common Mistakes

This solution ensures that updates are only processed if they match the current version in the database, effectively preventing lost updates. However, it’s important to note that this is a simplified example. In production systems, you might need to handle more complex scenarios such as multiple concurrent edits or large datasets.

#### Common mistakes
include:

Not properly handling exceptions and edge cases.
Not updating the version number correctly.
Failing to validate the incoming data properly.
Self-Check

The solution is correct and complete. It covers the basics of OCC in FastAPI, including fetching, validating, and updating items while ensuring consistency.

**Code:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import sqlite3

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str
    version: int  # Version number or timestamp

db_connection = sqlite3.connect("items.db")
cursor = db_connection.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS items (id INTEGER PRIMARY KEY, name TEXT, version INTEGER)")

def get_item_by_id(item_id: int):
    cursor.execute("SELECT * FROM items WHERE id=?", (item_id,))
    result = cursor.fetchone()
    if not result:
        return None
    return {"id": result[0], "name": result[1], "version": result[2]}

@app.get("/items/{item_id}")
def read_item(item_id: int):
    item = get_item_by_id(item_id)
    if item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"id": item["id"], "name": item["name"], "version": item["version"]}

@app.put("/items/{item_id}")
def update_item(item_id: int, item_data: Item):
    fetched_item = get_item_by_id(item_id)
    if fetched_item is None:
        raise HTTPException(status_code=404, detail="Item not found")

    # Client should include version number
    current_version = fetched_item["version"]
    incoming_version = item_data.version

    if current_version != incoming_version:
        return {"detail": "Version conflict detected. Please fetch the latest data and try again."}

    cursor.execute("UPDATE items SET name=?, version=? WHERE id=?", (item_data.name, current_version + 1, item_id))
    db_connection.commit()
    return {"id": item_id, "name": item_data.name, "version": current_version + 1}

@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    cursor.execute("DELETE FROM items WHERE id=?", (item_id,))
    db_connection.commit()
    return {"detail": f"Item {item_id} deleted"}
```

