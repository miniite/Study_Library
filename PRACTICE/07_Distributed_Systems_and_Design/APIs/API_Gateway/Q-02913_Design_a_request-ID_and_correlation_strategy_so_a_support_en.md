---
type: "practice"
difficulty: "Intermediate"
---

# Design a request-ID and correlation strategy so a support engineer can trace one user complaint from the gateway logs through every downstream service; what do you log and where?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Security and Data Masking](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Security_and_Data_Masking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that we can effectively trace a user's complaint across multiple services, we need to implement an effective request-ID and correlation strategy within our API Gateway. This involves logging specific IDs at strategic points in the system and ensuring they propagate through every service.

The core idea is to create a unique identifier for each request that remains constant as it moves through different layers of the system. By doing so, support engineers can trace the request's journey from the gateway logs all the way down to the database or other services involved.

#### Triaging the Question

The question asks us to design a strategy for logging IDs such that if a user complains about an issue, we can pinpoint exactly where and when things went wrong. The interviewer is probing our understanding of distributed tracing, request tracking, and logging best practices.

Key Assumptions
We are dealing with a multi-service microservices architecture.
Requests may pass through multiple services before reaching their final destination (e.g., database).
There could be retries or timeouts in the system, which need to be accounted for.

#### First-Principles Solution
Step 1: Define Request-ID and Correlation Strategy

We will use a UUID or a unique string as our request-ID. This ID should be generated at the API Gateway level and passed through every service layer. Here’s how it works:

Generate Request-ID: When an HTTP request comes in, the API Gateway generates a unique request-ID.
Log the Request-ID: The request-ID is logged along with other relevant metadata (e.g., user ID, timestamp) in the gateway logs.
Pass Through Services: As each service processes the request, it reuses the same request-ID to ensure traceability.
Step 2: Logging Strategy

The logging strategy needs to capture detailed information about the flow of requests through different services. This includes:

Gateway Logs: Log the incoming request with its ID and initial metadata.
Service Logs: Each service logs relevant details (e.g., processing time, errors) using the same request-ID.
Step 3: Retrying Mechanisms

If a retry mechanism is in place, the request-ID must be preserved across retries to maintain traceability. This can be achieved by storing the ID in headers or cookies.

#### Implementation

Here’s an example of how we might implement this strategy:

API Gateway
Generate Request-ID
Gateway Logs (ID, User-ID,Time)
Service 1
Service 2
Database
Response to User

Each service logs its actions with the same request-ID:

JSON
// Gateway Log Example
{
    "id": "abc123",
    "user_id": "12345",
    "timestamp": "2023-10-01T12:00:00Z",
    "action": "Received request from user"
}

// Service 1 Log Example
{
    "id": "abc123",
    "service": "Service1",
    "timestamp": "2023-10-01T12:01:00Z",
    "action": "Processing request"
}

#### Closing Summary
and Common Mistakes
Summary

The key to tracing user complaints effectively lies in maintaining a consistent and unique ID across all services. By logging this ID at every step, we can reconstruct the entire flow of the request and identify any issues.

#### Common Mistakes/Follow-Ups
Failing to Preserve Request-ID Across Retries: Ensure that the request-ID is stored securely (e.g., in a header) so it can be reused during retries.
Logging Insufficient Information: Logging only minimal details might make it difficult to diagnose issues; ensure logs contain sufficient context.
Ignoring Edge Cases: Consider edge cases like concurrent requests or failures at different layers.

#### Example
Code
PYTHON
# Python example for logging request-ID in a Flask app
from flask import Flask, request
import uuid

app = Flask(__name__)

@app.route('/api/v1/resource', methods=['GET'])
def handle_request():
    # Generate and log the request ID
    request_id = str(uuid.uuid4())
    print(f"Request ID: {request_id}")

    # Store in a header for downstream services to pick up
    headers = {'X-Request-ID': request_id}

    # Process the request...
    response_data = {"message": "Hello, World!"}

    return response_data, 200, headers

if __name__ == '__main__':
    app.run(debug=True)

**Code:**

```python
# Example Python code for logging request-ID in a Flask app
from flask import Flask, request
import uuid

app = Flask(__name__)

@app.route('/api/v1/resource', methods=['GET'])
def handle_request():
    # Generate and log the request ID
    request_id = str(uuid.uuid4())
    print(f"Request ID: {request_id}")

    # Store in a header for downstream services to pick up
    headers = {'X-Request-ID': request_id}

    # Process the request...
    response_data = {"message": "Hello, World!"}

    return response_data, 200, headers

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
// Example Request and Response

Request:
GET /api/v1/resource HTTP/1.1
Host: example.com
X-Request-ID: abc123

Response:
HTTP/1.1 200 OK
Content-Type: application/json
X-Request-ID: abc123

{"message": "Hello, World!"}
```

**Data:**

```json
// Gateway Log Example
{
    "id": "abc123",
    "user_id": "12345",
    "timestamp": "2023-10-01T12:00:00Z",
    "action": "Received request from user"
}

// Service 1 Log Example
{
    "id": "abc123",
    "service": "Service1",
    "timestamp": "2023-10-01T12:01:00Z",
    "action": "Processing request"
}
```

