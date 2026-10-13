---
type: "practice"
difficulty: "Deep"
---

# A POST endpoint occasionally creates duplicate records when the client retries after a timeout. How would you make it safe?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent POST Requests](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Idempotent_POST_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure a POST endpoint does not create duplicate records on retry, we need to implement a mechanism that checks if the record already exists before insertion. We'll use an IDempotent API pattern and leverage HTTP headers or database constraints for this purpose.

#### Triaging the Question

The problem is about ensuring idempotency in a REST API where retries after timeouts might result in duplicate records. The interviewer is probing our understanding of state management, data integrity, and best practices for handling concurrent requests. We need to ensure that each request can be retried without causing side effects.

#### Assumptions
Request IDs: The client should provide a unique identifier (ID) with the request.
Database Schema: The database schema must support querying by this ID.
Retry Logic: The client has retry logic in place, which could be due to network timeouts or other issues.

#### First-Principles Solution
Client-Side Request ID:
Ensure the client generates a unique request ID (e.g., UUID) and sends it as part of the POST request.
Server-Side Check:
The server checks if a record with the provided request ID already exists in the database before creating a new one.
Step 1: Client-Side Implementation

The client should add a unique identifier to each request, e.g., via an X-Request-ID header or as part of the JSON payload. Here’s an example using headers:

JSON
{
  "requestId": "123e4567-e89b-12d3-a456-426614174000",
  "data": {
    "name": "John Doe"
  }
}
Step 2: Server-Side Implementation

The server should validate if a record with the provided request ID already exists. If it does, the server can return an appropriate response without creating a new record.

PYTHON
from flask import Flask, request

app = Flask(__name__)

@app.route('/create_user', methods=['POST'])
def create_user():
    # Extract the requestId from the headers or payload
    request_id = request.headers.get('X-Request-ID')  # Assuming it's in headers

    # Check if the record already exists
    user_exists = User.query.filter_by(request_id=request_id).first()

    if user_exists:
        return {"message": "Record already exists"}, 208  # Already reported response code

    # If not, proceed with creating the new user
    data = request.json['data']
    user = User(name=data['name'], request_id=request_id)
    db.session.add(user)
    db.session.commit()

    return {"message": "User created successfully"}, 201

if __name__ == '__main__':
    app.run(debug=True)
Step 3: Handling Concurrent Requests
Ensure that the database constraints (e.g., unique index on request_id) enforce data integrity.
Use appropriate transaction isolation levels to handle concurrent requests safely.

#### Closing Summary

By implementing a mechanism to check for existing records before insertion, we ensure that each request is idempotent. This approach prevents duplicate creation of records during client retries and maintains data integrity. It's crucial to consider the side effects of retries on system state and use HTTP status codes appropriately (like 208 Already Reported) to inform the client about the situation.

#### Common Mistakes/Follow-Ups
Ignoring Data Integrity: Failing to enforce unique constraints in the database.
Not Handling Edge Cases: Not considering scenarios where multiple clients might generate the same request ID.
Improper Error Handling: Returning generic error messages without providing useful information for debugging or retry logic.
Triangulation:

Would the solution be different if we were dealing with a PUT endpoint instead of POST? How would this affect the idempotency of the operation?

**Comparison:**

```
User	id	name	request_id
```

**Code:**

```python
from flask import Flask, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'
db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    request_id = db.Column(db.String(36), unique=True, nullable=False)  # Unique constraint

@app.route('/create_user', methods=['POST'])
def create_user():
    request_id = request.headers.get('X-Request-ID')

    user_exists = User.query.filter_by(request_id=request_id).first()

    if user_exists:
        return {"message": "Record already exists"}, 208

    data = request.json['data']
    user = User(name=data['name'], request_id=request_id)
    db.session.add(user)
    db.session.commit()

    return {"message": "User created successfully"}, 201
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "requestId": "123e4567-e89b-12d3-a456-426614174000",
  "data": {
    "name": "John Doe"
  }
}

Output:

JSON
Copy
{
  "message": "Record already exists"
}
HTTP Status Code: 208 Already Reported
```

**Data:**

```yaml
User:
  - id: Integer (Primary Key)
  - name: String(50) (Not Nullable)
  - request_id: String(36) (Unique, Not Nullable)
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client POSTs Request
6. Request ID Exists?
7. Return 208 Already Reported
8. Create New User
9. Database Insertion Successful?
10. Return 201 Created
11. Rollback Transaction, RetryLater

