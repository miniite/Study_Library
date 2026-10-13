---
type: "practice"
difficulty: "Intermediate"
---

# How would you design REST endpoints for a blog with users, posts, and comments?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design REST endpoints for a blog application involving users, posts, and comments, we need to focus on creating a clean, efficient, and intuitive API that can handle various operations such as retrieving, creating, updating, and deleting resources. The key is to ensure that each endpoint follows the principles of REST (Representational State Transfer) and adheres to standard HTTP methods for CRUD operations.

First, let's break down what we need to achieve:

Users: Create, Read, Update, Delete (CRUD)
Posts: Create, Read, Update, Delete (CRUD), associated with users
Comments: Create, Read, Update, Delete (CRUD), associated with posts

Now, let's outline the REST endpoints from first principles:

User Endpoints

GET /users: Retrieve a list of all users.
GET /users/{userId}: Fetch details of a specific user by ID.
POST /users: Create a new user.
PUT /users/{userId}: Update the details of an existing user.
DELETE /users/{userId}: Delete a user.

Post Endpoints

GET /posts: Retrieve a list of all posts.
GET /posts/{postId}: Fetch details of a specific post by ID.
POST /posts: Create a new post, including associations with the current user or another user if needed.
PUT /posts/{postId}: Update the details of an existing post.
DELETE /posts/{postId}: Delete a post.

Comment Endpoints

GET /comments: Retrieve a list of all comments.
GET /comments/{commentId}: Fetch details of a specific comment by ID.
POST /comments: Create a new comment, associating it with the current post or another post if needed.
PUT /comments/{commentId}: Update the details of an existing comment.
DELETE /comments/{commentId}: Delete a comment.

Each endpoint should be designed to return appropriate HTTP status codes and responses. For example:

200 OK for successful GET requests
201 Created for successful POST/PUT requests
400 Bad Request if the request body is invalid
404 Not Found if the resource does not exist

To ensure robustness, consider adding query parameters and filters to allow clients to customize their requests. For instance:

GET /users?email=example@example.com to search for users by email.

Let's look at an example of what a POST request might look like:

HTTP
POST /posts
Content-Type: application/json

{
  "title": "My First Post",
  "content": "This is the first content I am sharing on my blog.",
  "authorId": 12345
}

The server would then create a new post associated with user ID 12345.

For more complex scenarios, consider pagination and sorting. For example:

HTTP
GET /posts?limit=10&offset=0&sortOrder=desc&sortBy=createdAt

This request retrieves the 10 most recent posts.

In conclusion, a well-designed REST API for a blog application should be simple, intuitive, and follow best practices to ensure scalability and maintainability. By focusing on clear endpoints and robust error handling, we can build a reliable and user-friendly interface that serves both developers and end-users effectively.

**Comparison:**

```
Endpoint	HTTP Method	Description
/users	GET	Retrieve a list of all users.
/users/{userId}	GET	Fetch details of a specific user by ID.
/users	POST	Create a new user.
/users/{userId}	PUT	Update the details of an existing user.
/users/{userId}	DELETE	Delete a user.
/posts	GET	Retrieve a list of all posts.
/posts/{postId}	GET	Fetch details of a specific post by ID.
/posts	POST	Create a new post, including associations with the current user or another user if needed.
/posts/{postId}	PUT	Update the details of an existing post.
/posts/{postId}	DELETE	Delete a post.
/comments	GET	Retrieve a list of all comments.
/comments/{commentId}	GET	Fetch details of a specific comment by ID.
/comments	POST	Create a new comment, associating it with the current post or another post if needed.
/comments/{commentId}	PUT	Update the details of an existing comment.
/comments/{commentId}	DELETE	Delete a comment.
```

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

# Example in-memory storage
users = {
    1: {"id": 1, "username": "user1", "email": "user1@example.com"},
    2: {"id": 2, "username": "user2", "email": "user2@example.com"}
}

posts = {
    100: {"id": 100, "title": "My First Post", "content": "This is the first content I am sharing on my blog.", "authorId": 1}
}

comments = {}

@app.route('/users', methods=['GET'])
def get_users():
    return jsonify(list(users.values()))

@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = users.get(user_id)
    if not user:
        return jsonify({"error": "User not found"}), 404
    return jsonify(user)

@app.route('/users', methods=['POST'])
def create_user():
    data = request.json
    new_user_id = max(users.keys()) + 1
    users[new_user_id] = {"id": new_user_id, "username": data["username"], "email": data["email"]}
    return jsonify({"message": f"User {new_user_id} created"}), 201

# Similar methods for PUT and DELETE operations on users, posts, and comments
```

**Example input / output:**

```

POST /users Content-Type: application/json

{ "username": "newuser", "email": "newuser@example.com" }

Response: 201 Created { "message": "User 3 created" }

CODE
Copy

```http
GET /posts?limit=10&offset=0&sortOrder=desc&sortBy=createdAt

Response:
200 OK
[
    {
        "id": 100,
        "title": "My First Post",
        "content": "This is the first content I am sharing on my blog.",
        "authorId": 1
    }
]
```

**Data:**

```json
{
  "users": [
    {"id": 1, "username": "user1", "email": "user1@example.com"},
    {"id": 2, "username": "user2", "email": "user2@example.com"}
  ],
  "posts": [
    {"id": 100, "title": "My First Post", "content": "This is the first content I am sharing on my blog.", "authorId": 1}
  ],
  "comments": [
    {"id": 300, "postId": 100, "text": "Great post!", "createdAt": "2023-10-01T14:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. GET /users
10. User exists?
11. Return user
12. 404 Not Found
13. POST /users
14. User data valid?
15. Create user, return 201
16. 400 Bad Request
17. GET /posts/{postId}
18. Post exists?
19. Return post
20. 404 Not Found
21. POST /comments
22. Comment data valid?
23. Create comment, return 201
24. 400 Bad Request

