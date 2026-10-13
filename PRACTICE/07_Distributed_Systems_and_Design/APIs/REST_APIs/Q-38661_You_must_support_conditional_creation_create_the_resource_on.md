---
type: "practice"
difficulty: "Deep"
---

# You must support conditional creation: create the resource only if one with the same natural key doesn't already exist, atomically, over REST. How do you express this and what status codes cover the race?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Operations Behind an Endpoint](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Distributed_Operations_Behind_an_Endpoint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of supporting conditional creation in a REST API, we need to ensure that the resource is only created if one with the same natural key does not already exist, atomically. This involves understanding the concept of optimistic concurrency control and leveraging HTTP status codes effectively. Let's break down the problem from first principles.

#### Triage
What’s Being Probed

The core of this question revolves around ensuring atomicity and consistency in a distributed system over REST. The interviewer is probing your knowledge of HTTP semantics, specifically ETags or similar optimistic locking mechanisms, and how they can be used to prevent race conditions during resource creation. Additionally, the question tests your understanding of appropriate status codes to handle different scenarios.

#### Assumptions
We are working with a typical CRUD API for resources identified by a natural key (e.g., an ID).
The system uses HTTP as the transport protocol.
We have access to ETags or similar optimistic concurrency control mechanisms.
The backend has support for conditional operations.
Step-by-Step Solution
Client Request: When a client sends a POST request to create a resource, they typically include an ETag or some other form of versioning information (e.g., If-None-Match header).
Server Validation: The server checks if the resource with the same natural key already exists.
Conditional Creation:
If no such resource exists, it creates the new resource and returns a 201 Created status code along with an ETag for the new resource.
If a resource with the same natural key does exist, the server uses the conditional operation to check if it can create the new resource atomically. This is typically done using an If-None-Match or similar header in the request.

#### Example
Workflow

Client Request:

HTTP
POST /api/resource HTTP/1.1
Host: example.com
Content-Type: application/json
If-None-Match: "etag-of-existing-resource"

{
    "name": "example"
}

Server Validation and Conditional Creation:

The server checks if a resource with the name "example" exists.
If it does not exist, the server creates the new resource and returns a 201 Created status code along with an ETag for the new resource.
HTTP
HTTP/1.1 201 Created
Location: /api/resource/example
ETag: "new-etag-of-created-resource"
Content-Type: application/json

{
    "id": 123,
    "name": "example",
    "created_at": "2023-10-01T12:00:00Z"
}
If a resource with the name "example" already exists, the server uses the If-None-Match header to check if it can create the new resource atomically. Since the ETag in the request ("etag-of-existing-resource") does not match the current ETag of the existing resource, the server returns a 412 Precondition Failed status code.
HTTP
HTTP/1.1 412 Precondition Failed
Content-Type: application/json

{
    "error": "Resource with name 'example' already exists."
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

By leveraging conditional operations and appropriate status codes, we ensure that the creation of a resource is atomic and consistent. The key takeaway is that HTTP provides robust mechanisms to handle concurrency issues without relying solely on pessimistic locking or database transactions.

#### Common Mistakes

Not using ETags or similar versioning information.
Failing to understand the semantics of 412 Precondition Failed status code.
Ignoring the importance of atomicity in distributed systems.

#### Follow-Ups
How would you handle multiple clients trying to create resources with the same natural key simultaneously?
What about performance considerations and scaling implications?

**Comparison:**

```
HTTP Status Code	Description
201 Created	Resource created successfully.
412 Precondition Failed	Precondition in the request header did not pass.
```

**Code:**

```python
def conditional_create_resource(request_body, natural_key):
    existing_resource = get_resource_by_natural_key(natural_key)
    if not existing_resource:
        new_resource_id = generate_new_id()
        new_resource = create_new_resource(new_resource_id, request_body)
        return 201, {"id": new_resource_id, "name": request_body["name"], "created_at": datetime.now().isoformat()}

    # Check ETag or similar versioning information
    if not match_version_info(request_body["version"], existing_resource["version"]):
        return 412, {"error": f"Resource with name '{natural_key}' already exists."}

    return 200, {"message": "Resource update successful"}
```

**Example input / output:**

```text
# Client Request
POST /api/resource HTTP/1.1
Host: example.com
Content-Type: application/json
If-None-Match: "etag-of-existing-resource"

{
    "name": "example",
    "version": "etag-of-existing-resource"
}

# Server Response
HTTP/1.1 201 Created
Location: /api/resource/example
ETag: "new-etag-of-created-resource"
Content-Type: application/json

{
    "id": 456,
    "name": "example",
    "created_at": "2023-10-01T12:00:00Z"
}
```

**Data:**

```json
{
    "request_body": {
        "name": "example",
        "version": "etag-of-existing-resource"
    },
    "existing_resource": {
        "id": 1,
        "name": "example",
        "version": "current-etag"
    }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Client sends POST withIf-None-Match
6. Resource exists?
7. Server creates new resourceand returns 201
8. ETag matches current version?
9. Server returns 412Precondition Failed
10. Server updates existingresource and returns 200

