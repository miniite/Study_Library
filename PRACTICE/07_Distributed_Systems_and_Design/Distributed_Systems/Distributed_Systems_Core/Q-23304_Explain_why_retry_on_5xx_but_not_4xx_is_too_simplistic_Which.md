---
type: "practice"
difficulty: "Deep"
---

# Explain why 'retry on 5xx but not 4xx' is too simplistic. Which errors are safely retryable, which are ambiguous (timeouts on non-idempotent writes), and how do you make ambiguous ones safe?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retry Budgets and Backoff](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing error handling in distributed systems, one common piece of advice is to "retry 5xx errors but not 4xx." However, this simplistic approach can lead to significant issues and security vulnerabilities. Let's break down why this rule is too simplistic and explore the nuances of retryability.

Firstly, HTTP status codes are divided into categories:

1xx Informational - Message received
2xx Success - Request succeeded
3xx Redirection - Further action needed by client
4xx Client Error - Request is flawed or cannot be fulfilled
5xx Server Error - Server failed to fulfill legitimate request

For distributed systems, it's crucial to understand the implications of retrying different types of errors. The key distinction lies in whether an error indicates a transient issue (retryable) or a permanent condition that should not be retried.

#### Triaging the Question

The question asks us to explain why "retry on 5xx but not 4xx" is too simplistic, identify which errors are safely retryable, which ones are ambiguous, and how to handle those ambiguities.

Key Concepts
Transient Errors: Short-lived conditions that can be resolved by a retry.
Permanent Errors: Conditions that cannot be fixed by simply retrying the request.
Idempotent Operations: Actions that have no side effects when retried multiple times (e.g., GET requests).
Non-idempotent Operations: Actions with potential side effects if retried more than once (e.g., POST, PUT).

#### First Principles Solution

Transient Errors and 5xx Status Codes:

503 Service Unavailable: The server is currently unable to handle the request due to a temporary overloading or maintenance of the server.
504 Gateway Timeout: The gateway did not receive a timely response from an upstream server.
These are typically retryable because they indicate short-term issues.

Permanent Errors and 4xx Status Codes:

401 Unauthorized: Authentication failed; you need to provide credentials.
403 Forbidden: Access is denied due to insufficient permissions.
409 Conflict: The request could not be completed because of a conflict with the current state on the server.
These are generally not retryable as they indicate permanent conditions.

Ambiguous Errors and Timeouts:

Timeouts on Non-idempotent Operations: Consider a scenario where a write operation (e.g., creating a record) is retried after a timeout but could result in duplicate records if the first attempt was partially successful.
This ambiguity makes it tricky to determine whether retrying will cause issues.
Handling Ambiguous Errors

To handle ambiguous errors, we need mechanisms that ensure idempotency:

Idempotent Retry Mechanism: Use unique identifiers (e.g., UUIDs) for requests. If a request with the same identifier is retried, it should not modify the state on the server.
Circuit Breaker Patterns: Implement patterns like Hystrix or Resilience4j to manage retries and failures gracefully.
Detailed Example

Let's consider an example of creating a new user in a distributed system:

HTTP
POST /users
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}
First Attempt: User creation is successful.
Timeout and Retry: A timeout occurs during the second attempt. If we simply retry, we might end up creating a duplicate user.

To handle this safely:

HTTP
POST /users
X-Request-ID: 1234567890abcdef
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}

On the server side, we check if a user with email="alice@example.com" already exists:

HTTP
GET /users?email=alice@example.com
X-Request-ID: 1234567890abcdef

If the user does not exist, proceed to create the new user. If it does, return an appropriate response (e.g., 409 Conflict) and do not retry.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, "retry on 5xx but not 4xx" is too simplistic because:

It doesn't account for the nuances of idempotent vs. non-idempotent operations.
Ambiguous errors like timeouts on writes require careful handling to avoid issues.

#### Common mistakes
include:

Failing to distinguish between transient and permanent errors.
Not implementing mechanisms (e.g., UUIDs, circuit breakers) that ensure safe retries.

#### Follow-up
questions might explore specific retry strategies or how to handle distributed transactions in more detail.

**Comparison:**

```
Status Code	Description	Retryable?
503	Service Unavailable	Yes
504	Gateway Timeout	Yes
401	Unauthorized	No
403	Forbidden	No
409	Conflict	No (Check first)
```

**Code:**

```python
# Example of handling idempotent requests with a unique identifier
def create_user(request_id, user_data):
    user_email = user_data['email']

    # Check if the user already exists
    existing_user = get_user_by_email(user_email)

    if not existing_user:
        return create_new_user(user_data)  # Safely create new user

    return "User already exists", 409

def get_user_by_email(email):
    # Logic to fetch user from database or cache
    pass

def create_new_user(data):
    # Safe creation logic
    pass
```

**Example input / output:**

```text
Input:
POST /users
X-Request-ID: 1234567890abcdef
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}

Output (First Attempt):
Status: 201 Created
Location: /users/123

Output (Second Attempt, if timeout occurs and user does not exist):
Status: 409 Conflict
```

**Data:**

```json
{
  "request_id": "1234567890abcdef",
  "user_data": {
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

**Flow:**

1. Yes
2. No
3. Error Handling
4. Yes
5. No
6. Timeout Occurs
7. Retry Request? (Transient)
8. Handle Retry Gracefully
9. Fail Request and Log Issue
10. User Create Request
11. Is User Email Unique?
12. Create New User
13. Return Conflict Response 409
14. Unique User CreatedSuccessfully

