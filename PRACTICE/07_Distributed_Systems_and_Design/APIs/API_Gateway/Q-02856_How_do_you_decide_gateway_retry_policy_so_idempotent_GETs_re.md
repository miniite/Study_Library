---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide gateway retry policy so idempotent GETs retry with backoff and jitter but POSTs don't, and how do you enforce this per-route without a team accidentally retrying a payment write?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring that only POST requests are retried while GET requests with idempotent behavior get retried with backoff and jitter, we need to design an effective retry policy within our API Gateway setup. The key is to understand what constitutes a GET request and ensure it's idempotent, meaning the result of multiple identical requests should be the same as a single request.

TL;DR
For POST requests, do not allow retries.
For idempotent GET requests, implement backoff and jitter with proper retry logic.
Enforce these policies per route to avoid accidental retries on critical operations like payment writes.

#### Triage
What the Question is Asking

The question requires us to define a robust API Gateway retry policy that distinguishes between different types of HTTP methods. Specifically, GET requests should be retried with idempotent behavior using backoff and jitter strategies, whereas POST requests, particularly those involving critical operations like payments, must not be retried.

#### Assumptions
The API Gateway supports custom retry policies.
We are working in a distributed system where network disruptions can occur.
GET requests are idempotent by design, meaning they do not change the state of the server.

#### First-Principles Solution

Identify and Categorize Requests:

Use HTTP method to categorize requests:
GET: Typically idempotent.
POST: Often used for critical operations like payments or data updates, where retries could lead to double processing.

Implement Retrying Logic for GETs with Jitter Backoff:

For GET requests, implement a retry policy that involves exponential backoff and random jitter to avoid overwhelming the server with consecutive requests.

#### Example
JSON
"retryPolicy": {
  "maxRetries": 3,
  "initialIntervalMs": 1000,
  "intervalMultiplier": 2,
  "maximumIntervalMs": 60000,
  "jitterPercent": 50
}

No Retrying for POSTs:

For POST requests, especially those involving critical operations like payments, no retry logic should be applied.

#### Example
JSON
"retryPolicy": {
  "enabled": false
}

Route-Level Configuration:

Ensure that these policies are configured per route to enforce specific behavior for each API endpoint.

#### Example
configuration in AWS API Gateway:
YAML
routes:
- path: /payments/write
  methods:
    POST: {}
- path: /data/query
  methods:
    GET: {
      retryPolicy: {
        maxRetries: 3,
        initialIntervalMs: 1000,
        intervalMultiplier: 2,
        maximumIntervalMs: 60000,
        jitterPercent: 50
      }
    }

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By configuring the API Gateway to apply different retry policies based on HTTP method, we can ensure that idempotent GET requests get retried with backoff and jitter while critical POST operations like payments are not accidentally retried.

#### Common Mistakes
Forgetting to enable or disable retries for specific routes.
Not considering the idempotency of GET requests, which can lead to unexpected behavior if they are retried too frequently.
Accidentally configuring a retry policy that could cause double processing on critical operations.

#### Example
Code and Configuration
YAML
# API Gateway Route Configuration
routes:
- path: /payments/write
  methods:
    POST: {
      integration: {
        type: AWS_PROXY,
        credentials: arn:aws:iam::123456789012:role/your-integration-role
      }
    }
    OPTIONS: {}
- path: /data/query
  methods:
    GET: {
      integration: {
        type: AWS_PROXY,
        credentials: arn:aws:iam::123456789012:role/your-integration-role
      },
      retryPolicy: {
        maxRetries: 3,
        initialIntervalMs: 1000,
        intervalMultiplier: 2,
        maximumIntervalMs: 60000,
        jitterPercent: 50
      }
    }

**Comparison:**

| Route | Method | Retry Policy |
|-------|--------|-------------|
| /payments/write | POST | Disabled |
| /data/query | GET | Enabled (backoff + jitter) |

**Code:**

```yaml
# Example API Gateway Route Configuration in YAML
routes:
- path: /payments/write
  methods:
    POST: {}
    OPTIONS: {}
- path: /data/query
  methods:
    GET: {
      retryPolicy: {
        maxRetries: 3,
        initialIntervalMs: 1000,
        intervalMultiplier: 2,
        maximumIntervalMs: 60000,
        jitterPercent: 50
      }
    }
```

**Example input / output:**

```text
# Example Input and Output for GET Retry Policy Configuration
Input:
- Route: /data/query

Output:
- GET request to /data/query has a retry policy configured with backoff and jitter.
- POST requests to /payments/write are not retried.

Request Response:
GET /data/query HTTP/1.1
Host: example-api-gateway.com

Response (with retries):
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": "retrieved data"
}
```

**Data:**

```json
{
  "retryPolicy": {
    "maxRetries": 3,
    "initialIntervalMs": 1000,
    "intervalMultiplier": 2,
    "maximumIntervalMs": 60000,
    "jitterPercent": 50
  }
}
```

**Flow:**

1. GET Request
2. Retry with Backoff and Jitter
3. POST Request
4. No Retry
5. Process Request
6. Process Request

