---
type: "practice"
difficulty: "Deep"
---

# A search-as-you-type feature fires a REST call per keystroke, and out-of-order responses cause the wrong results to render. How do you handle cancellation and response ordering robustly in the client?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Long-running Operations](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of handling cancellation and ensuring proper response ordering in a search-as-you-type feature, we need to design a system that can manage multiple requests efficiently, cancel stale ones when necessary, and handle responses in the correct order despite potential delays or out-of-order delivery.

#### Triaging the Question

The question is essentially asking how to implement robust request handling on the client side. The interviewer is probing your understanding of REST API best practices, concurrency control, and error handling strategies. We assume that each keystroke triggers a search query, and we must ensure that only the most recent requests are processed.

#### Solution
from First Principles
Unique Request Identifiers: Each request should have a unique identifier (ID) that can be used to track its status.
CancellationToken: Use CancellationToken in asynchronous operations to allow for graceful cancellation of old requests when a newer one is triggered.
Response Mapping and De-duplication: Maintain an ordered queue or map of responses, ensuring they are processed only once based on their unique IDs.
Rate Limiting and Concurrency Control: Implement rate limiting to prevent overwhelming the server and concurrency control to manage multiple simultaneous requests.
Detailed Steps

Generate a Unique ID per Request

When a user types a character, generate a unique requestId for that request.

Use CancellationToken

For each search query, use a CancellationTokenSource that is associated with the requestId.
If a new request comes in while an older one is still pending, cancel the old request using its CancellationToken.

Queue Responses by ID

Use a dictionary or map to store responses keyed by their requestId.
This ensures we only process each response once.

Asynchronous Requests with State Management

Make asynchronous REST calls and manage the state of each request.
Ensure that requests are not re-executed if already in progress (e.g., using a boolean flag).
Code Example

Here's how you might implement this logic in JavaScript:

JAVASCRIPT
class SearchService {

#### cons
tructor(httpClient) {
    this.http = httpClient;
    this.requestMap = new Map();
    this.cancellationTokenSource = new CancellationTokenSource();
  }

  async search(query, callback) {

#### cons
t requestId = `search-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

    // Clear old token if a newer request is made
    if (this.requestMap.has(requestId)) {
      this.cancellationTokenSource.cancel();
    }

    this.cancellationTokenSource = new CancellationTokenSource();

#### cons
t options = { cancelToken: this.cancellationTokenSource.token };

    try {

#### cons
t response = await this.http.get(`/api/search?query=${encodeURIComponent(query)}`, options);

      if (this.requestMap.has(requestId)) {
        // Process the response
        callback(response.data);
      }
    } catch (error) {

#### cons
ole.error('Search request failed', error);
    }

    this.cancellationTokenSource = null; // Reset for next request
  }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that only the most recent search requests are processed, and it handles cancellation of old requests gracefully. Key mistakes include not using unique IDs, ignoring response de-duplication, or failing to properly manage CancellationToken. Common follow-ups might involve discussing performance considerations, such as rate limiting and how to handle network failures.

