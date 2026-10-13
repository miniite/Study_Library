---
type: "practice"
difficulty: "Deep"
---

# How do you handle bulk operations, e.g. creating or deleting many resources in one request?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Operations Behind an Endpoint](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Distributed_Operations_Behind_an_Endpoint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with bulk operations such as creating or deleting multiple resources in a single request, it's crucial to ensure that your REST API is both efficient and robust. The key challenge lies in managing concurrency, error handling, performance optimization, and maintaining the integrity of the database. Let’s break this down step by step.

Firstly, the core question revolves around how we can handle bulk operations effectively while ensuring that each resource’s state remains consistent, regardless of any failures during processing. This involves understanding the nature of bulk requests, the impact on system performance, and the importance of robust error handling mechanisms.

#### Assumptions

Each request is a collection of individual resource IDs or data.
The backend processes these resources in a transactional manner to ensure atomicity.
Failures can occur at any point during processing, necessitating proper rollback strategies.

To address this, we can take the following steps:

Designing the API Endpoint:

Create an endpoint that accepts a list of resource IDs or data in JSON format (e.g., POST /resources/bulk).
Ensure the request payload is validated to avoid processing invalid input.

Transaction Management:

Use transactions to ensure that all operations within a bulk request either succeed completely or fail entirely, maintaining database integrity.
For example, using SQL transactions or distributed transaction management for microservices.

Error Handling and Logging:

Log errors at each step of the process for debugging and auditing purposes.
Provide detailed error messages in responses to help clients understand what went wrong.

Optimizing Performance:

Batch processing can be used to reduce database write operations, improving performance.
Parallelize tasks where possible but ensure that they do not conflict with transactional integrity.

Scalability and Fault Tolerance:

Design the system to handle large-scale requests without causing service degradation.
Implement retries and fallback strategies for transient failures.

Let’s illustrate this with a simple example of bulk resource creation:

JSON
{
  "resources": [
    { "name": "Resource1", "type": "TypeA" },
    { "name": "Resource2", "type": "TypeB" },
    { "name": "Resource3", "type": "TypeC" }
  ]
}

This JSON can be processed to create the specified resources atomically. The response would indicate success or failure for each resource, with detailed error messages if necessary.

**Comparison:**

```
Resource	Status
Resource1	created
Resource2	error: validation failed
Resource3	created
```

**Code:**

```python
import json
from flask import Flask, request

app = Flask(__name__)

@app.route('/resources/bulk', methods=['POST'])
def bulk_create_resources():
    data = request.get_json()
    resources = data.get('resources', [])

    for resource in resources:
        try:
            # Simulate creating a resource (e.g., database insert)
            create_resource(resource['name'], resource['type'])

            # Record success
            result['success'] = True
        except Exception as e:
            # Log the error and record failure
            logger.error(f"Failed to create {resource}: {str(e)}")
            result['success'] = False

    return json.dumps(result), 200, {'Content-Type': 'application/json'}

def create_resource(name, resource_type):
    # Simulate database insert
    pass

result = {
    "success": True,
    "results": []
}
```

**Example input / output:**

```text
Input:
{
  "resources": [
    { "name": "Resource1", "type": "TypeA" },
    { "name": "Resource2", "type": "TypeB" },
    { "name": "Resource3", "type": "TypeC" }
  ]
}

Output:
{
  "success": true,
  "results": [
    {"name": "Resource1", "status": "created"},
    {"name": "Resource2", "status": "error: validation failed"},
    {"name": "Resource3", "status": "created"}
  ]
}
```

**Data:**

```json
{
  "resources": [
    { "name": "Resource1", "type": "TypeA" },
    { "name": "Resource2", "type": "TypeB" },
    { "name": "Resource3", "type": "TypeC" }
  ]
}
```

**Flow:**

1. Success
2. Failure
3. Receive Bulk Request
4. Validate Payload
5. Start Transaction
6. Process Each Resource
7. Commit Transaction
8. Rollback Transaction
9. Log Error and Return Response
10. Return Successful Response

