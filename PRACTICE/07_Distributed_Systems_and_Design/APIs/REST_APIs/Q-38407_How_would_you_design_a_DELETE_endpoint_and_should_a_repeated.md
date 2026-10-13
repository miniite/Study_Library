---
type: "practice"
difficulty: "Deep"
---

# How would you design a DELETE endpoint, and should a repeated DELETE return an error the second time?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent POST Requests](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Idempotent_POST_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a DELETE endpoint in RESTful API architecture, the primary goal is to ensure that resources are removed efficiently while maintaining data integrity and consistency. A DELETE request should be idempotent, meaning it can be executed multiple times without changing its outcome beyond the first execution. However, there are scenarios where repeated DELETE requests might need to be handled differently.

#### Triage

The core question here is whether a DELETE endpoint should return an error if called repeatedly on the same resource. The interviewer is likely probing your understanding of REST principles, idempotency, and best practices in API design. We'll assume that the resource identifier (e.g., URL) uniquely identifies the resource to be deleted.

#### First-Principles Solution
Idempotence: A DELETE request should ideally be idempotent. Once a resource is deleted, subsequent identical requests should not have additional side effects.
Resource State: The server must check if the resource still exists before processing the DELETE. If it does not exist, return an appropriate status code and message to inform the client.
Reasoning

Initial DELETE Request:

The server checks if the resource exists with the given identifier.
If the resource is found, it is deleted, and a success status (e.g., 204 No Content) is returned.
If the resource does not exist, an appropriate error (e.g., 404 Not Found) can be returned.

Subsequent DELETE Requests:

The server should check if the resource still exists before processing any further logic.
If the resource no longer exists, return a suitable status code and message to indicate that nothing has changed or was deleted (e.g., 204 No Content).

#### Example Scenario

#### Cons
ider a user management API where users can be deleted from an organization. The URL might look like this: /api/users/{userId}.

Initial DELETE Request:

Client sends a DELETE request to /api/users/1.
Server checks if the user with ID 1 exists.
User is found, and server deletes it.
Response: 204 No Content.

Subsequent DELETE Requests:

Client sends another DELETE request to /api/users/1.
Server checks if the user with ID 1 still exists.
The user no longer exists (already deleted).
Response: 204 No Content or 404 Not Found.

#### Closing Summary

In conclusion, a well-designed DELETE endpoint should ensure idempotency and handle repeated requests by checking the existence of the resource before performing any action. While the initial delete operation will modify the state (if the resource exists), subsequent operations on an already deleted resource should not change the state further.

#### Common Mistakes/Follow-Ups
Ignoring Idempotence: Failing to make the DELETE endpoint idempotent can lead to unintended side effects.

#### Cons
istency Issues: Not checking the existence of the resource before processing a second request might result in inconsistent states or errors.
Error Handling: Returning meaningful status codes and messages is crucial for proper client behavior.

#### Follow-ups
could include:

How would you handle partial deletions (e.g., soft delete)?
What about transactional consistency across multiple resources?
How do you ensure that the DELETE operation is atomic?

**Comparison:**

```
Endpoint	HTTP Method	Description
/api/users/{id}	DELETE	Deletes a user with the given ID
```

**Code:**

```python
@app.delete("/api/users/{user_id}")
async def delete_user(user_id: int):
    user = await get_user_by_id(user_id)
    if not user:
        return {"error": "User does not exist"}, 404

    # Perform the deletion logic (e.g., database operation)
    db_session.delete(user)
    db_session.commit()

    return {"message": "User deleted successfully"}, 204
```

**Example input / output:**

```text
- **Initial DELETE Request**:
  - Input: `DELETE /api/users/1`
  - Response: `204 No Content`

- **Subsequent DELETE Request**:
  - Input: `DELETE /api/users/1`
  - Response: `204 No Content` (or `404 Not Found`)
```

**Data:**

```json
{
  "user_id": 1,
  "username": "john_doe"
}
```

**Flow:**

1. Yes
2. No
3. Client
4. Resource Exists?
5. Delete Resource
6. Return 204 No Content
7. Return 404 Not Found

