---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a path parameter and a query parameter? When would you use each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a REST API, path parameters and query parameters serve different purposes but are both crucial for defining the structure of your URLs. Path parameters are part of the resource identifier (URL) itself, while query parameters come after a "?" in the URL. The choice between these two depends on the nature of the data you're passing to or receiving from the API.

#### Triage

The question is asking about the differences and use cases for path parameters and query parameters in REST APIs. It aims to understand your grasp of RESTful design principles, which are fundamental to building scalable and maintainable web services.

#### First-Principles Solution

Path Parameters:

Location-Specific Data: Path parameters are used when the data is part of the resource's identity. For example, if you have a /users/{userId} endpoint, {userId} is a path parameter because it defines which specific user resource we're accessing.
Immutable Data: Typically, path parameters represent immutable or unique identifiers that don't change during the lifecycle of the resource.

Query Parameters:

Filtering and Sorting: Query parameters are used for specifying filters, sorting criteria, pagination, etc., without changing the core identity of the resource. For example, /users?name=John&sort=name uses query parameters to filter users by name and sort them.
Mutable State: They can be used to control the state or behavior of a request that doesn't change the underlying data's identity.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, path parameters are essential for identifying specific resources within your API, whereas query parameters are more flexible and allow you to manipulate how those resources are retrieved without altering their core identity. Misusing these can lead to confusing URLs or making your API less intuitive.

#### Common Mistakes

Using query parameters when a path parameter would be clearer.
Overusing path parameters for too many details, leading to overly complex URL structures.

#### Example

#### Cons
ider an endpoint for retrieving user details:

Path Parameter Example: GET /users/{userId} where {userId} is the specific user's ID.
Query Parameter Example: GET /users?name=John&status=active to filter users by name and status.

**Code:**

```python
# Example path parameter in a Flask route
@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    return f"User {user_id}"

# Example query parameters in a Flask route
@app.route('/users', methods=['GET'])
def filter_users():
    name = request.args.get('name')
    status = request.args.get('status')
    # Logic to fetch users based on filters
    return f"Filtered Users: Name={name}, Status={status}"
```

**Example input / output:**

```text
- **Path Parameter Input:** `GET /users/456`
- **Query Parameter Input:** `GET /users?name=admin&sort=desc`
```

**Data:**

```json
// Example of a path parameter in URL
{
  "url": "/users/123"
}

// Example of query parameters in URL
{
  "url": "/users?name=John&status=active"
}
```

