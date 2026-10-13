---
type: "practice"
difficulty: "Intermediate"
---

# What HTTP methods map to which CRUD operations and how do you implement them in Flask?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Request Parsing and Validation](<../../../../TOOLS/Python/Flask/Request_Parsing_and_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a web application, understanding how HTTP methods correspond to CRUD (Create, Read, Update, Delete) operations is fundamental. This knowledge helps in designing RESTful APIs that are easy to understand and maintain. In this answer, we'll discuss which HTTP methods map to each CRUD operation and demonstrate their implementation using Flask.

#### Triaging the Question

The question asks about the mapping of HTTP methods to CRUD operations and how they can be implemented in Flask. The interviewer is probing your understanding of RESTful design principles and your ability to apply them using a popular Python framework like Flask. Assumptions include familiarity with basic HTTP methods (GET, POST, PUT, DELETE) and the concept of CRUD operations.

#### First-Principles Solution
Mapping HTTP Methods to CRUD Operations:
Create: Typically maps to POST. This method is used for creating a new resource on the server.
Read: Maps to GET. It retrieves information from the server based on given parameters, such as an ID or query string.
Update: Maps to PUT or PATCH. Both methods are used to update a resource. PUT is generally used for complete updates, while PATCH is used for partial updates.
Delete: Maps to DELETE. It removes a resource from the server.

#### Implementation
in Flask:

To implement these operations in Flask, you'll typically define routes using decorators like @app.route(), specifying the HTTP method and the corresponding function that performs the operation. Below is an example of how to set up such routes in Flask.

PYTHON
from flask import Flask, request

app = Flask(__name__)

# Example data storage (in a real application, this would be replaced by a database)
users = []

@app.route('/user', methods=['POST'])
def create_user():
    new_user = request.json  # Assuming JSON payload is used for creating a user
    users.append(new_user)   # Add the user to our in-memory list
    return {"message": "User created"}, 201

@app.route('/user/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = next((u for u in users if u['id'] == user_id), None)
    if not user:
        return {"error": "User not found"}, 404
    return user, 200

@app.route('/user/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    updated_data = request.json
    for i in range(len(users)):
        if users[i]['id'] == user_id:
            users[i] = {**users[i], **updated_data}  # Update the existing user
            return {"message": "User updated"}, 200
    return {"error": "User not found"}, 404

@app.route('/user/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    global users
    new_users = [u for u in users if u['id'] != user_id]
    users = new_users
    return {"message": "User deleted"}, 200

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes:

In the example above, we've implemented a simple RESTful API with Flask for managing users. The key points to remember are:

Use appropriate HTTP methods (POST, GET, PUT, DELETE) for CRUD operations.
Ensure proper error handling (e.g., returning 404 for non-existent resources).
Maintain clear and concise code structure.

#### Common mistakes
include using the wrong HTTP method, not validating input data correctly, or neglecting to handle edge cases such as missing required fields. Always consider security aspects like CSRF protection and validation of user inputs.

**Code:**

```python
from flask import Flask, request

app = Flask(__name__)

# Example data storage (in a real application, this would be replaced by a database)
users = []

@app.route('/user', methods=['POST'])
def create_user():
    new_user = request.json  # Assuming JSON payload is used for creating a user
    users.append(new_user)   # Add the user to our in-memory list
    return {"message": "User created"}, 201

@app.route('/user/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = next((u for u in users if u['id'] == user_id), None)
    if not user:
        return {"error": "User not found"}, 404
    return user, 200

@app.route('/user/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    updated_data = request.json
    for i in range(len(users)):
        if users[i]['id'] == user_id:
            users[i] = {**users[i], **updated_data}  # Update the existing user
            return {"message": "User updated"}, 200
    return {"error": "User not found"}, 404

@app.route('/user/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    global users
    new_users = [u for u in users if u['id'] != user_id]
    users = new_users
    return {"message": "User deleted"}, 200

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```
POST /user request example (creating a user):

{ "id": 1, "name": "John Doe", "email": "johndoe@example.com" }

Response: { "message": "User created" }

CODE
Copy

```json
# GET /user/<user_id> request example (fetching a user by ID):
{
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com"
}

Response:
{
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com"
}
JSON
Copy
# PUT /user/<user_id> request example (updating a user):
{
    "name": "Jane Doe"
}

Response:
{
    "message": "User updated"
}
JSON
Copy
# DELETE /user/<user_id> request example (deleting a user):
{
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com"
}

Response:
{
    "message": "User deleted"
}
```

