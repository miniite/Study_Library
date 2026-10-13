---
type: "practice"
difficulty: "Deep"
---

# You need a batch endpoint where partial success is allowed. How do you design the request and response so the client can retry only the failed items idempotently, and what status code fits?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Operations Behind an Endpoint](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Distributed_Operations_Behind_an_Endpoint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a batch endpoint that allows for partial success and requires clients to retry only the failed items idempotently, we need to ensure the request is designed to handle failures gracefully and the response provides enough information for clients to determine what actions were successful or failed. This involves using appropriate HTTP status codes and crafting a well-structured JSON response.

The core idea here is to use 207 Multi-Status as the status code, which allows returning multiple status codes in one response, making it ideal for batch processing where some items might succeed while others fail. The request should be idempotent so that retrying the same request does not change the outcome after the first successful execution.

**Comparison:**

```
Status Code	Meaning
207	Multi-Status
Action	HTTP Method
Update	PATCH
Delete	DELETE
Create	POST
Item ID	Status	Message
1	Success	Item updated successfully.
2	Conflict	Failed to delete item. The target resource is in a conflicting state due to external data modification.
3	Created	Item created successfully.
```

**Code:**

```

POST /batch-process HTTP/1.1 Host: example.com Content-Type: application/json

[ {"id": 1, "action": "update", "data": {}}, {"id": 2, "action": "delete"}, {"id": 3, "action": "create", "data": {}} ]

CODE
Copy
```http
HTTP/1.1 207 Multi-Status
Content-Type: application/json

[
    {
        "id": 1,
        "status": "Success",
        "message": "Item updated successfully."
    },
    {
        "id": 2,
        "status": "Conflict",
        "message": "Failed to delete item. The target resource is in a conflicting state due to external data modification."
    },
    {
        "id": 3,
        "status": "Created",
        "message": "Item created successfully."
    }
]
```

**Example input / output:**

```
Input: Batch request containing multiple actions.
JSON
Copy
[
    {"id": 1, "action": "update", "data": {}},
    {"id": 2, "action": "delete"},
    {"id": 3, "action": "create", "data": {}}
]
Output: Response containing status and messages for each item.
JSON
Copy
[
    {
        "id": 1,
        "status": "Success",
        "message": "Item updated successfully."
    },
    {
        "id": 2,
        "status": "Conflict",
        "message": "Failed to delete item. The target resource is in a conflicting state due to external data modification."
    },
    {
        "id": 3,
        "status": "Created",
        "message": "Item created successfully."
    }
]
```

**Flow:**

1. Yes
2. No
3. Request
4. Verify IDs
5. Is ID valid?
6. Execute Action
7. Record Success/Failed
8. Next Item
9. Return Conflict
10. Construct Response
11. Return 207 Multi-StatusResponse

