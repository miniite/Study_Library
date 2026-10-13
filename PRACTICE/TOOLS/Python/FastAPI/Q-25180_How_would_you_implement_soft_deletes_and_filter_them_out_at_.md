---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement soft deletes and filter them out at the query layer in a FastAPI plus SQLAlchemy service?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database Access and Tool Calls](<../../../../TOOLS/Python/FastAPI/Database_Access_and_Tool_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement soft deletes and filter them out at the query layer in a FastAPI service using SQLAlchemy, we need to ensure that deleted records are not returned by default but can be queried or filtered out if needed. This involves updating the delete() method of our models to mark records as "deleted" rather than actually removing them from the database. We'll also add a query parameter to control whether these soft-deleted records should be included in the results.

#### Triage

The question is asking how to implement and manage soft deletes using FastAPI with SQLAlchemy, specifically focusing on ensuring that deleted records are filtered out by default but can still be accessed if required. The core idea here is to update our delete logic and query filters to accommodate this behavior.

#### First Principles Solution
Model Update: Ensure the model has a deleted_at field to track when a record was deleted.
Delete Logic: When deleting a record, set its deleted_at timestamp instead of removing it entirely from the database.
Query Layer Filter: Add an optional query parameter in your API endpoints that allows filtering out soft-deleted records.
Step-by-Step Solution
1. Model Update

First, we modify our SQLAlchemy model to include a deleted_at field:

PYTHON
from sqlalchemy import Column, DateTime, Integer, String, func
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    deleted_at = Column(DateTime, nullable=True)

    def delete(self):
        # Soft delete: set the deleted_at timestamp
        self.deleted_at = datetime.utcnow()
2. Delete Logic Update

Update the delete() method in our model to mark a record as soft-deleted:

PYTHON
def delete(self):
    self.deleted_at = datetime.utcnow()
    session.commit()  # Assuming 'session' is an active SQLAlchemy session
3. Query Layer Filter

Next, we need to modify our queries to filter out soft-deleted records by default. We'll add a query parameter (e.g., include_deleted) that allows us to include or exclude deleted records.

PYTHON
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from .database import get_session

app = FastAPI()

def get_users(include_deleted: bool = False, session: Session = Depends(get_session)):
    query = session.query(User)

    if not include_deleted:
        # Filter out soft-deleted records by default
        query = query.filter_by(deleted_at=None)

    return query.all()
4. Endpoint Implementation

Finally, we implement the API endpoint to allow users to fetch all users with or without soft-deleted ones:

PYTHON
@app.get("/users")
async def get_users(include_deleted: bool = False):
    session = get_session()  # Assuming `get_session` is defined as a dependency
    result = await get_users(include_deleted, session)
    return {"users": result}

#### Closing Summary

In summary, to implement soft deletes in FastAPI with SQLAlchemy, we need to update our model and query logic. We add a deleted_at field and modify the delete method to set this timestamp instead of removing records. By default, queries filter out deleted records unless explicitly asked to include them.

#### Common Mistakes/Follow-Ups
Forgetting to handle soft deletes in all places: Ensure that any part of your application interacting with users (e.g., APIs) properly accounts for the deleted_at field.
Not considering performance implications: Soft-deleted records can still consume space and potentially impact query performance, so it's important to monitor and optimize accordingly.

For more complex scenarios, you might want to consider using PostgreSQL's ON DELETE CASCADE or similar features to manage related deletions.

**Comparison:**

```
Rendering diagram…
```

**Code:**

```

from sqlalchemy import Column, DateTime, Integer, String, func from sqlalchemy.orm import relationship from sqlalchemy.ext.declarative import declarative_base from datetime import datetime

Base = declarative_base()

class User(Base): tablename = 'users'

CODE
Copy
id = Column(Integer, primary_key=True)
name = Column(String, nullable=False)
deleted_at = Column(DateTime, nullable=True)

def delete(self):
    self.deleted_at = datetime.utcnow()
CODE
Copy

```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from .database import get_session

app = FastAPI()

def get_users(include_deleted: bool = False, session: Session = Depends(get_session)):
    query = session.query(User)

    if not include_deleted:
        query = query.filter_by(deleted_at=None)

    return query.all()
PYTHON
Copy
@app.get("/users")
async def get_users(include_deleted: bool = False):
    session = get_session()  # Assuming `get_session` is defined as a dependency
    result = await get_users(include_deleted, session)
    return {"users": result}
```

**Example input / output:**

```text
# Querying with include_deleted=False (default)
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "deleted_at": null
    }
  ]
}

# Querying with include_deleted=True
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "deleted_at": null
    },
    {
      "id": 2,
      "name": "Jane Smith",
      "deleted_at": "2023-10-05T14:30:00"
    }
  ]
}
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "deleted_at": null
    },
    {
      "id": 2,
      "name": "Jane Smith",
      "deleted_at": "2023-10-05T14:30:00"
    }
  ]
}
```

**Flow:**

1. Rendering diagram…

