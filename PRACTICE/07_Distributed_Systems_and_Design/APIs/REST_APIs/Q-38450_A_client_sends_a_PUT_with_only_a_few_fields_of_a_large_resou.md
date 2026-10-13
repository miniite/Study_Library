---
type: "practice"
difficulty: "Deep"
---

# A client sends a PUT with only a few fields of a large resource. What happens to the omitted fields under strict PUT semantics?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Coarse vs Fine-grained Endpoints](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Coarse_vs_Fine-grained_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a client sends a PUT request with only a few fields of a large resource, the behavior is strictly defined by REST API principles and HTTP standards. Under strict PUT semantics, any missing fields in the payload are treated as if they were explicitly set to their default or null values (depending on the data type). This can lead to unintended side effects because not all fields may have appropriate defaults, particularly for complex resources.

To understand this fully, let's break down the key concepts and reasoning from first principles. We'll start by defining what a PUT request does in RESTful APIs and then explore some of its subtleties.

**Comparison:**

| Field  | Original Value  | Updated Value |
|--------|-----------------|---------------|
| id     | 12345           | (unchanged)   |
| name   | Previous Name   | New Name      |
| description  | Old Description | Updated description |
| status | active          | (unchanged)   |
| location | SF, CA         | (unchanged)   |

**Code:**

```python
# Example Python code to illustrate handling a PUT request
def update_resource(resource_id, payload):
    # Fetch the existing resource (simplified for illustration)
    existing_resource = fetch_resource_from_db(resource_id)

    # Apply updates from the payload
    for key, value in payload.items():
        if hasattr(existing_resource, key):
            setattr(existing_resource, key, value)

    # Save the updated resource back to the database
    save_resource_to_db(existing_resource)
```

**Example input / output:**

```

Given a PUT request with the following payload:

HTTP
Copy
PUT /resource/12345 HTTP/1.1
Content-Type: application/json

{
  "name": "New Name",
  "description": "Updated description"
}

The response would be:

The name field is updated to "New Name".
The description field is updated to "Updated description".
Any other fields in the resource, such as status and location, remain unchanged from their original values.
```

**Data:**

```json
{
  "id": "12345",
  "name": "Example Resource",
  "description": "A detailed description of the resource",
  "status": "active",
  "location": "San Francisco, CA"
}

Given a PUT request with only some fields:

HTTP
Copy
PUT /resource/12345 HTTP/1.1
Content-Type: application/json

{
  "name": "New Name",
  "description": "Updated description"
}

The server would process this as follows:

The existing resource is fetched.
Only the name and description fields are updated, while all other fields remain unchanged (i.e., they retain their original values).
```

**Flow:**

1. Yes
2. No
3. Fetch Existing Resource
4. Is Field in Payload?
5. Update Field and Save
6. Skip Update for this Field
7. Check Next Field
8. Save Updated Resource to DB

