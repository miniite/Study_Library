---
type: "practice"
difficulty: "Deep"
---

# A gateway must apply different timeout and retry policies for reads vs writes on the same upstream; how do you classify request safety at the edge and avoid retrying a write that may have partially applied?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with designing an API Gateway to handle different timeout and retry policies for read and write operations on the same upstream service. The key challenge is ensuring that a write request is not retried if it has only partially succeeded, potentially leading to data corruption or inconsistencies.

#### Triage

The question asks us to classify request safety at the edge (i.e., in the API Gateway) and prevent retries of partial writes. Specifically:

Understand Request Types: Differentiate between read and write requests.
Timeouts and Retries: Apply different timeout values for reads and writes.
Partial Success Handling: Ensure that a write operation is not retried if it has only partially succeeded.

#### First-Principles Solution

Request Classification:

Read Requests: Shorter timeouts, as read operations are less critical in terms of data consistency but must complete quickly to provide timely responses.
Write Requests: Longer timeouts and stricter retry logic, since partial writes can lead to inconsistent states.

Timeouts:

Set a shorter timeout for reads (e.g., 500ms).
Set a longer timeout for writes (e.g., 3 seconds).

Retry Logic:

For write requests, implement a retry mechanism with backoff and exponential retries.
For read requests, no or minimal retry logic since they are less critical.

Partial Success Handling:

Use an optimistic concurrency control mechanism like etags to detect partial writes.
Ensure that the gateway can distinguish between successful and failed write operations based on response status codes (e.g., 200 for success, 500 for failure).

#### Implementation
Steps

Define Request Types:

PYTHON
def is_read_request(request):
    return 'read' in request.path.lower()

def is_write_request(request):
    return 'write' in request.path.lower()

Apply Timeouts and Retry Logic:

Use a middleware or framework-specific decorators to apply timeouts.
Implement retry logic using a library like tenacity for Python.

Partial Success Handling with Etags:

PYTHON
def handle_write_request(request):
    response = send_request_to_upstream(request)
    if response.status_code != 200:
        # Handle partial write failure
        return response

    etag = response.headers.get('etag')
    if not etag:
        return response

    request.headers['If-Match'] = etag
    retry = tenacity.Retrying(stop=tenacity.stop_after_attempt(3), wait=tenacity.wait_exponential(multiplier=1, min=4, max=20))
    for attempt in retry:
        with attempt:
            response = send_request_to_upstream(request)
            if 'etag' in response.headers and response.status_code == 200:
                return response

#### Closing Summary

By classifying requests at the edge of our API Gateway, we can apply different timeout and retry policies to ensure optimal performance for reads while maintaining data integrity for writes. Key steps include defining request types, applying appropriate timeouts and retries, and using optimistic concurrency controls like etags to handle partial successes gracefully.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the distinction between read and write requests, leading to inconsistent behavior.

#### Follow-Up
Discussing how to handle retries in a distributed system with multiple upstream services.
Mistake: Not considering edge cases where partial writes might occur due to network failures or other issues.

#### Follow-Up
Explaining additional resilience measures like circuit breakers and fallback strategies.

**Comparison:**

| Operation Type | Timeout (ms) | Retry Mechanism |
|----------------|-------------|----------------|
| Read           | 500         | Minimal        |
| Write          | 3000        | Exponential    |

**Code:**

```

def is_read_request(request): return 'read' in request.path.lower()

def is_write_request(request): return 'write' in request.path.lower()

CODE
Copy

```python
from tenacity import retrying, wait_exponential, stop_after_attempt

def handle_write_request(request):
    response = send_request_to_upstream(request)
    if response.status_code != 200:
        # Handle partial write failure
        return response

    etag = response.headers.get('etag')
    if not etag:
        return response

    request.headers['If-Match'] = etag
    retry = retrying.Retrying(stop=retrying.stop_after_attempt(3), wait=retrying.wait_exponential(multiplier=1, min=4, max=20))
    for attempt in retry:
        with attempt:
            response = send_request_to_upstream(request)
            if 'etag' in response.headers and response.status_code == 200:
                return response
```

**Example input / output:**

```text
# Example input and output
request = Request(path="/write/resource", method="PUT")
response = handle_write_request(request)
print(response.status_code)  # Expected: 200 if write succeeded

request = Request(path="/read/resource", method="GET")
response = handle_read_request(request)
print(response.status_code)  # Expected: <shorter timeout>, will retry if necessary
```

**Data:**

```json
{
  "request": {
    "path": "/write/resource",
    "headers": {
      "If-Match": "etag123"
    }
  },
  "response": {
    "status_code": 200,
    "headers": {
      "etag": "new_etag456"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive Request
6. Is Read?
7. Apply Short Timeout, Retry ifNecessary
8. Is Write?
9. Apply Long Timeout,Exponential Retries with EtagCheck
10. Pass Through

