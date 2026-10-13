---
type: "practice"
difficulty: "Intermediate"
---

# How would you expose a soft-delete pattern in a Flask API so records are marked deleted rather than removed?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Request Parsing and Validation](<../../../../TOOLS/Python/Flask/Request_Parsing_and_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To expose a soft-delete pattern in a Flask API, we need to ensure that records are marked as deleted instead of being completely removed from the database. This involves updating the relevant record to include a "deleted" flag or timestamp, and ensuring the API handles these flags appropriately when fetching data. The core idea is to maintain consistency with the business logic while providing robust APIs for different use cases.

In this context, we're triaging the question by understanding that the goal is to implement soft deletion in an efficient manner using Flask. The key assumptions are:

We have a database model (e.g., SQLAlchemy) that represents our data.
Our API will provide endpoints to mark records as deleted and fetch only non-deleted records.

We'll start from first principles by defining the necessary steps, then implement them in code and finalize with best practices and potential edge cases.

Database Model: We need a model that includes a deleted_at timestamp (or is_deleted flag) to mark records as soft deleted.
API Endpoints: We'll create endpoints for marking records as deleted and fetching non-deleted records.
Business Logic: Ensure the business logic is correctly applied when interacting with the database.

Let's break this down step-by-step:

Step 1: Database Model

We use SQLAlchemy to define our model:

PYTHON
from datetime import datetime
from sqlalchemy import Column, DateTime, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    deleted_at = Column(DateTime, nullable=True)

    def is_deleted(self):
        return self.deleted_at is not None
Step 2: API Endpoints

Next, we'll create the Flask routes and business logic for handling soft deletion:

Route to Mark Records as Deleted
PYTHON
from flask import Blueprint, request

user_api = Blueprint('user_api', __name__)

@user_api.route('/users/<int:user_id>/delete', methods=['POST'])
def mark_user_as_deleted(user_id):
    user = User.query.get_or_404(user_id)
    user.deleted_at = datetime.utcnow()
    db.session.commit()
    return {"message": f"User {user_id} marked as deleted."}
Route to Fetch Non-Deleted Records
PYTHON
@user_api.route('/users', methods=['GET'])
def get_users():
    users = User.query.filter_by(deleted_at=None).all()
    return [{"id": user.id, "name": user.name} for user in users]
Step 3: Business Logic and Edge Cases

Ensure that the business logic is correctly applied:

Soft Deletion: Update deleted_at instead of deleting.
Fetching Data: Use filters to exclude deleted records.

Potential edge cases include handling concurrent modifications, ensuring transactional integrity, and managing soft delete in complex queries (e.g., pagination).

#### Closing Summary
+ Common Mistakes

In summary, implementing a soft-delete pattern in a Flask API involves updating the deleted_at timestamp or using an is_deleted flag to mark records as deleted. We need to ensure that our database model and API endpoints handle these flags correctly.

#### Common mistakes
include:

Incorrectly Removing Records: Accidentally removing instead of marking as deleted.
Ignoring Soft Deletes in Queries: Failing to filter out soft deleted records when fetching data.
Concurrency Issues: Handling concurrent modifications properly to avoid race conditions.

By following these steps and considering the edge cases, we can implement a robust soft-delete pattern in our Flask API.

**Comparison:**

| User ID | Name   | Deleted At             |
|---------|--------|-----------------------|
| 1       | Alice  | null                  |
| 2       | Bob    | 2023-10-05T14:48:00Z  |

**Code:**

```

from flask import Blueprint, request from sqlalchemy import create_engine, Column, DateTime, Integer, String, Boolean from sqlalchemy.ext.declarative import declarative_base from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class User(Base): tablename = 'users'

CODE
Copy
id = Column(Integer, primary_key=True)
name = Column(String(100))
deleted_at = Column(DateTime, nullable=True)

def is_deleted(self):
    return self.deleted_at is not None

user_api = Blueprint('user_api', name)

engine = create_engine('sqlite:///example.db') Session = sessionmaker(bind=engine)

@user_api.route('/users/int:user_id/delete', methods=['POST']) def mark_user_as_deleted(user_id): user = User.query.get_or_404(user_id) user.deleted_at = datetime.utcnow() db.session.commit() return {"message": f"User {user_id} marked as deleted."}

@user_api.route('/users', methods=['GET']) def get_users(): session = Session() users = session.query(User).filter_by(deleted_at=None).all() session.close() return [{"id": user.id, "name": user.name} for user in users]

CODE
Copy

```python
from sqlalchemy.orm import scoped_session, sessionmaker

Base.metadata.create_all(engine)
session_factory = scoped_session(sessionmaker(bind=engine))

def get_user(user_id):
    session = session_factory()
    try:
        return session.query(User).get(user_id)
    finally:
        session.close()

# Example usage in Flask routes
```

**Example input / output:**

```text
Input:
POST /users/2/delete

Output:
{"message": "User 2 marked as deleted."}

Input:
GET /users

Output:
[
    {"id": 1, "name": "Alice", "deleted_at": null},
    {"id": 2, "name": "Bob", "deleted_at": "2023-10-05T14:48:00Z"}
]
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "deleted_at": null},
    {"id": 2, "name": "Bob", "deleted_at": "2023-10-05T14:48:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Mark User as Deleted
6. Is user_id valid?
7. Update deleted_at to currenttimestamp
8. Commit transaction
9. Return success message
10. Return error message
11. Fetch Non-Deleted Users
12. Check if session is closed
13. Filter by non-deleted records
14. Close session
15. Return user data
16. Reopen session and fetch data

