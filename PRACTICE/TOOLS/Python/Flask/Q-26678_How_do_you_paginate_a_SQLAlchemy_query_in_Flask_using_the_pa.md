---
type: "practice"
difficulty: "Intermediate"
---

# How do you paginate a SQLAlchemy query in Flask using the paginate() helper?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Flask-SQLAlchemy](<../../../../TOOLS/Python/Flask/Flask-SQLAlchemy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To paginate a SQLAlchemy query in Flask, we can use the paginate() method provided by SQLAlchemy’s Query class. This method is particularly useful when dealing with large datasets and allows us to fetch only a subset of results at a time. Here's a step-by-step guide on how to implement this:

TL;DR: Use the paginate() method from SQLAlchemy's Query object to handle pagination efficiently, specifying page number and items per page.
Understanding the Question: The interviewer is probing your understanding of SQLAlchemy’s query capabilities and pagination techniques in Flask. They are also interested in how you manage state (like current page and items-per-page) and potentially, handling edge cases.

#### First Principles Solution
Setup Environment: Ensure you have a Flask application set up with SQLAlchemy configured properly to connect to your database.
Define Model: Suppose we have a model User defined in SQLAlchemy as follows:
PYTHON
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
Query and Paginate:
PYTHON
from flask import request, jsonify

@app.route('/users', methods=['GET'])
def get_users():
    page = int(request.args.get('page', 1))  # Default to page 1 if not provided
    per_page = int(request.args.get('per_page', 20))  # Default to 20 items per page

    users = User.query.paginate(page, per_page, error_out=False)

    response = {
        'items': [user.to_dict() for user in users.items],
        'has_next': users.has_next,
        'has_prev': users.has_prev,
        'total_pages': users.pages,
        'current_page': users.page
    }

    return jsonify(response), 200

#### Closing Summary
and Common Mistakes/Follow-Ups:
Ensure you handle the error_out=False parameter to avoid raising an exception if no results are found for a given page.
Be prepared to discuss edge cases such as invalid pages or per-page values, and how your implementation deals with them (e.g., using default values).

#### Common mistakes
include not handling query parameters correctly or forgetting to convert user input from the request to integers before passing it to paginate().

**Comparison:**

```
Parameter	Description
page	The page number to fetch (default is 1)
per_page	Number of items per page (default is 20)
```

**Code:**

```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))

@app.route('/users', methods=['GET'])
def get_users():
    page = int(request.args.get('page', 1))
    per_page = int(request.args.get('per_page', 20))

    users = User.query.paginate(page, per_page, error_out=False)

    response = {
        'items': [user.to_dict() for user in users.items],
        'has_next': users.has_next,
        'has_prev': users.has_prev,
        'total_pages': users.pages,
        'current_page': users.page
    }

    return jsonify(response), 200
```

**Example input / output:**

```text
Input:
GET /users?page=1&per_page=5

Response:
{
    "items": [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ],
    "has_next": true,
    "has_prev": false,
    "total_pages": 2,
    "current_page": 1
}
```

**Data:**

```json
{
    "items": [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ],
    "has_next": true,
    "has_prev": false,
    "total_pages": 2,
    "current_page": 1
}
```

**Flow:**

1. Yes
2. Yes
3. User GET request
4. Is 'page' parameter set?
5. Is 'per_page' parameter set?
6. Convert page and per_page tointegers
7. Query User with paginate()
8. Generate response object
9. Return JSON response

