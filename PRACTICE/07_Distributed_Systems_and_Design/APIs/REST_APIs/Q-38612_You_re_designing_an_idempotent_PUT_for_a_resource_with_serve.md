---
type: "practice"
difficulty: "Expert"
---

# You're designing an idempotent PUT for a resource with server-generated timestamps and computed fields. How do you define what 'same request produces same state' means when the body can't include those fields?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent POST Requests](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Idempotent_POST_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design an idempotent PUT API that ensures the same request will always result in the same state, despite the absence of server-generated timestamps and computed fields from the client's perspective. The key is to ensure idempotence by leveraging HTTP headers and potentially using a combination of request parameters.

Firstly, let's define what "same request produces same state" means: it implies that if we send the exact same request multiple times under identical conditions (same payload, same context), the resulting resource state should remain unchanged or be consistent with each invocation. The challenge here is to handle this in a way that doesn't include server-generated fields like timestamps and computed values directly in the PUT body.

#### Triage
Understanding the Question

The question probes understanding of idempotence, HTTP headers, and the limitations imposed by not including certain data (like server-generated timestamps) in the request payload. We need to ensure consistency and handle potential edge cases where multiple identical requests might differ based on context or timing.

#### First-Principles Solution
Leveraging HTTP Headers

Given that we can't include server-generated fields like timestamps directly, we will use an idempotency key in a custom header. This key uniquely identifies the request and is stored server-side to check for duplicates. Here’s how it works:

Client-Side: The client generates or uses a unique identifier (idempotency key) as part of the request headers.
Server-Side: Upon receiving the request, the server checks if this idempotency key has been used before. If not, the server processes the request and stores the idempotency key in its database or cache.
Subsequent Requests: Any subsequent requests with the same idempotency key are recognized as duplicates and ignored by the server.
Step-by-Step Solution

Define the Idempotency Key: The client must generate a unique identifier for each request. This could be based on a combination of parameters like user ID, resource ID, or even a randomly generated UUID.

HTTP Header Implementation:

HTTP
PUT /api/resource/{id} HTTP/1.1
Host: example.com
X-Idempotency-Key: 1234567890-abcd
Content-Type: application/json

{
    "field": "value"
}

Server-Side Processing:

The server checks if the X-Idempotency-Key has been used before.
If it hasn't, process the request and store the key in a database or cache with the current state.
If it has, ignore the request.

Handling Subsequent Requests: Any subsequent requests with the same idempotency key are recognized as duplicates and ignored by the server. The resource state remains consistent.

#### Closing Summary

By using an idempotency key in a custom header, we ensure that the same request produces the same state even if the body of the PUT request doesn't include server-generated fields like timestamps or computed values. This approach leverages HTTP headers to maintain consistency and handle idempotence effectively.

#### Common Mistakes/Follow-Ups
Misunderstanding Idempotency: Ensuring that all requests with the same idempotency key are truly identical.
Cache Busting: Considering edge cases where client-side cache might interfere with request uniqueness.
Security Concerns: Addressing potential security risks related to idempotency keys, such as how they are generated and stored.

**Comparison:**

```
Request	Idempotency Key Used?	Action Taken
First	No	Processed
Second	Yes	Ignored
```

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

# In-memory store for simplicity; use database in production
idempotency_key_store = {}

@app.route('/api/resource/<int:id>', methods=['PUT'])
def update_resource(id):
    idempotency_key = request.headers.get('X-Idempotency-Key')

    if not idempotency_key:
        return jsonify({"error": "Idempotency key is missing"}), 400

    # Check if the key has been used before
    if idempotency_key in idempotency_key_store:
        return jsonify({"message": "Request already processed"}), 202

    # Process request (omitting detailed logic for brevity)
    payload = request.json
    updated_resource = process_request(id, payload)

    idempotency_key_store[idempotency_key] = True
    return jsonify(updated_resource)

def process_request(resource_id, payload):
    # Simulated resource update logic
    pass

if __name__ == '__main__':
    app.run()
```

**Example input / output:**

```

Input:

HTTP
Copy
PUT /api/resource/1 HTTP/1.1
Host: example.com
X-Idempotency-Key: 1234567890-abcd
Content-Type: application/json

{
    "field": "value"
}

Output:

JSON
Copy
{
    "message": "Request already processed",
    "status_code": 202
}
```

**Data:**

```json
{
    "X-Idempotency-Key": "1234567890-abcd",
    "Content-Type": "application/json"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client Sends PUT Request
6. Is Idempotency Key Present?
7. Is Key in Store?
8. Process Request & Store Key
9. Ignored by Server
10. 400 Bad Request

