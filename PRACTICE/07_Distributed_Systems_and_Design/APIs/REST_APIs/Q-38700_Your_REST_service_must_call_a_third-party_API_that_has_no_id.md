---
type: "practice"
difficulty: "Expert"
---

# Your REST service must call a third-party API that has no idempotency support and occasionally times out after actually succeeding. How do you build a reconciliation loop to detect and heal these ambiguous outcomes?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Operations Behind an Endpoint](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Distributed_Operations_Behind_an_Endpoint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of handling ambiguous outcomes from a third-party API that lacks idempotency support and can occasionally timeout, we need to implement a robust reconciliation loop. The key challenge is ensuring that we correctly identify when an operation has been successfully executed despite initial timeouts or partial failures. This solution involves tracking requests, managing retries, and implementing a heuristic for determining successful outcomes.

#### Triage
Understanding the Problem

The question requires us to design a mechanism that can handle situations where:

The third-party API occasionally returns success but may timeout.
We need to ensure that our system does not re-attempt operations unnecessarily (which could lead to double spends or other issues).
We must detect when an operation has genuinely succeeded, even if the initial response was delayed or failed.

#### Assumptions
and Initial Steps
The API returns a unique identifier for each request.
There is no idempotency support in the third-party API.
Timeouts can occur but do not indicate failure.
We have control over how often we retry requests to the third-party service.
A successful outcome must be detected based on additional data or heuristics, as direct confirmation may not always be available.
Step-by-Step Solution
Track Requests: Maintain a database of all issued requests with their unique identifiers and statuses (initially set to "pending").
Implement Retries: Design a retry mechanism that re-attempts failed requests after an exponential backoff period.
Detection Mechanism: Implement a heuristic to determine if the operation has successfully completed, even if the initial response was ambiguous.
Tracking Requests

We use a simple database table to track request statuses:

RequestID	APIResponseStatus	LastAttemptedAt	SuccessDetected
12345	pending	2023-10-01T09:00	false
Retry Mechanism

Implement a retry loop with exponential backoff:

PYTHON
def send_request(request):
    response = call_third_party_api(request)
    if response.status_code == 200:
        update_db_status(request.id, "success", True)
        return response

    # Handle timeouts and other failures
    if response.status_code in [504, 503]:
        time.sleep(2 ** num_retries)  # Exponential backoff
        retry_request(request.id)

def retry_request(request_id):
    retries = 0
    while not is_success_detected(request_id) and retries < max_retries:
        response = call_third_party_api(get_request_details(request_id))
        update_db_status(request_id, "pending", False)

        if response.status_code == 200:
            update_db_status(request_id, "success", True)
            return response
        else:
            time.sleep(2 ** retries)  # Exponential backoff
            retries += 1

    # If no success detected after max retries, mark as failed
    update_db_status(request_id, "failed", False)
Heuristic for Success Detection

Implement a heuristic to determine if an operation has succeeded based on additional context or timing:

PYTHON
def is_success_detected(request_id):
    details = get_request_details(request_id)

    # Check if there's a delay in response
    if time.time() - details['last_attempted_at'] > 10:  # e.g., check after 10 seconds
        return True

    # Check for specific patterns or values in the API response
    if 'specific_value' in details and details['response'].contains('expected_pattern'):
        return True

    return False

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution involves tracking requests, implementing a retry mechanism with exponential backoff, and using heuristics to detect successful outcomes. Key considerations include avoiding double spends by ensuring each operation is executed only once, managing retries effectively, and detecting success based on context or timing.

#### Common Mistakes

Over-relying solely on the initial API response.
Failing to implement proper retry logic with exponential backoff.
Not considering additional heuristics for successful outcome detection.
Further Discussion
Rate Limiting: How do you handle rate limits imposed by the third-party service?
Scalability: What changes would you make if this system needed to scale significantly?

**Comparison:**

```
RequestID	APIResponseStatus	LastAttemptedAt	SuccessDetected
12345	success	2023-10-01T09:05:00Z	true
```

**Code:**

```python
def call_third_party_api(request):
    # Implementation of API call here
    pass

def update_db_status(request_id, status, success_detected=False):
    # Update database with new status and success detection
    pass

def get_request_details(request_id):
    # Fetch request details from the database
    pass

def retry_request(request_id):
    retries = 0
    while not is_success_detected(request_id) and retries < max_retries:
        response = call_third_party_api(get_request_details(request_id))
        update_db_status(request_id, "pending", False)

        if response.status_code == 200:
            update_db_status(request_id, "success", True)
            return response
        else:
            time.sleep(2 ** retries)  # Exponential backoff
            retries += 1

    # If no success detected after max retries, mark as failed
    update_db_status(request_id, "failed", False)

def is_success_detected(request_id):
    details = get_request_details(request_id)

    # Check if there's a delay in response
    if time.time() - details['last_attempted_at'] > 10:  # e.g., check after 10 seconds
        return True

    # Check for specific patterns or values in the API response
    if 'specific_value' in details and details['response'].contains('expected_pattern'):
        return True

    return False
```

**Example input / output:**

```text
Input:
send_request(12345)

Output:
{
  "request_id": 12345,
  "api_response_status": "success",
  "last_attempted_at": "2023-10-01T09:05:00Z"
}
```

**Data:**

```json
{
  "requests": [
    {
      "request_id": 12345,
      "api_response_status": "pending",
      "last_attempted_at": "2023-10-01T09:00:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Send Request
6. Is Response OK?
7. Mark as Success
8. Retry with Exponential Backoff
9. Has Max Retries BeenReached?
10. Mark as Failed

