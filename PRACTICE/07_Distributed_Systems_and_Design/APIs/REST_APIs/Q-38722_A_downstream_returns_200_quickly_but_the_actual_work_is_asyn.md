---
type: "practice"
difficulty: "Intermediate"
---

# A downstream returns 200 quickly but the actual work is async and sometimes fails silently later. How do you design your integration to know the real outcome, using callbacks, reconciliation, or status polling?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Long-running Operations](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a backend environment where downstream services return 200 quickly but the actual work might fail asynchronously, it's crucial to ensure that we can accurately determine the real outcome of the request. This problem often arises in microservices architectures or distributed systems where individual service failures are not immediately apparent. To address this, you can use callbacks, reconciliation, or status polling. Here, I'll focus on a robust solution using a combination of callback-based mechanisms and status polling.

The goal is to design an integration flow that ensures we get definitive feedback about the outcome, even if the initial response is 200 OK. We need to handle failures gracefully and provide meaningful error messages or logging to debug issues.

#### Triage
Understanding the Problem
What the Question is Asking: The question aims to explore how you would design an integration that can reliably track the success or failure of a downstream service even when the initial response is misleading.
What the Interviewer is Probing: The interviewer will assess your understanding of asynchronous operations, error handling, and the nuances of distributed systems. They may also evaluate your ability to balance simplicity with robustness.

#### Assumptions
We are dealing with a microservice architecture where services communicate over HTTP/REST.
Downstream services return 200 OK upon receiving a request but might fail later due to internal errors or external factors.
The backend service needs to ensure that the user receives appropriate feedback about the outcome.

#### Solution
Combining Callbacks and Status Polling
Step-by-Step First Principles Solution

Initial Request Handling:

When an API request is made, send it to the downstream service using a standard HTTP POST method.
The initial response from the downstream should be 200 OK.

Callback Mechanism:

Implement a callback mechanism where the downstream service can asynchronously notify the backend about the outcome once the actual work is completed.
Store a unique identifier (e.g., request ID) in a database or cache to associate the incoming request with the eventual response.
The callback should include this identifier and the result of the operation.

Status Polling:

If no callback is received within a reasonable time frame, implement status polling.
Periodically check the downstream service's state using an HTTP GET or POST request to the same endpoint used initially.
This can be done via a scheduled task (e.g., every 5 minutes) to avoid overwhelming the system with frequent requests.

Error Handling and Logging:

Log all attempts, both callbacks and status polling, for debugging purposes.
Ensure that any errors during these processes are captured and reported appropriately.

#### Example
Workflow

User Request -> Backend:

HTTP
POST /api/v1/downstream-service
Content-Type: application/json

{
  "request_id": "12345",
  "data": {...}
}

Backend Stores Request ID and Sends to Downstream Service.

Downstream Completes Work (Successfully or Fails):

If successful, the downstream sends a callback with the result.
HTTP
POST /api/v1/callback
Content-Type: application/json

{
  "request_id": "12345",
  "status": "success",
  "result": {...}
}
If failed, no callback is sent.

Backend Checks Status Polling:

HTTP
GET /api/v1/downstream-service/status/12345

Backend Updates Response to User Based on Callback or Polling Result:

If a successful callback is received, return 200 OK with the result.
If no callback and polling times out, return an appropriate error message.

#### Closing Summary

By combining callbacks and status polling, you can ensure that your integration handles asynchronous failures gracefully. This approach balances simplicity with robustness, making it suitable for most backend scenarios where downstream services might fail silently but need to be tracked reliably.

#### Common Mistakes/Follow-Ups

Mistake: Over-relying on a single mechanism (either callbacks or polling) without considering the other.

#### Follow-Up
Discuss how both mechanisms can complement each other and why combining them is beneficial.

Mistake: Not logging enough information to debug issues.

#### Follow-Up
Explain how detailed logging can help in diagnosing problems, especially during outages or unexpected behaviors.

By addressing these aspects, you ensure a comprehensive solution that handles both success and failure cases robustly.

