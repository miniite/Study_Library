---
type: "practice"
difficulty: "Deep"
---

# An intermittent bug shows a request succeeding on the server but the client reporting failure. How would you investigate this class of issue?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Progressive Results](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose an intermittent bug where a request appears to succeed on the server but fails on the client, we need to understand that the root cause could lie anywhere in the communication stack, including network issues, asynchronous behavior, and timing problems. The goal is to systematically narrow down the problem area by tracing data flows and verifying each component's reliability.

First, let’s triage the question:

The issue occurs intermittently, suggesting it might be related to timing or state inconsistencies.
The bug manifests as a discrepancy between server success and client failure.
We need to identify potential sources of failure in both the client and server implementations, possibly including network glitches or handling nuances.

Key assumptions include that we have access to logging mechanisms on both client and server sides, and that the application is reasonably well-designed with separation of concerns. The solution will involve a mix of code inspection, log analysis, and potentially, controlled testing environments.

Step-by-step from first principles:

Log Analysis:
Check logs for any discrepancies between when the request is sent (client) and received/processed (server).
Look for patterns or specific conditions under which the failure occurs.
Network Inspection:
Use network tools like Wireshark to inspect packet captures.
Ensure that the requests are being routed correctly through all layers of the network stack.
Server Response Handling:
Verify that the server is sending a consistent and correct response code (e.g., 200 OK) even when there’s an issue internally.
Client-Side Error Handling:
Ensure the client-side error handling logic correctly interprets and reports any non-2xx status codes or other errors returned by the server.
Asynchronous Behavior:
If the application is using asynchronous calls, ensure that callbacks are properly managed to avoid race conditions.

Now let's delve into a detailed solution:

Collect Logs:

On both client and server sides, enable detailed logging for the requests being made and received.
Include timestamps, request headers, payload, response codes, and any relevant error messages or logs from critical components (e.g., database queries).

Network Debugging:

Set up Wireshark to capture network traffic during the failure scenario.
Look for dropped packets, malformed requests/responses, or unexpected timing issues.

Server-side Validation:

Inspect server logs to ensure that all steps of request processing are correctly completed.
Check if there are any internal failures (e.g., database errors) that might be masked by the 200 response code.

Client-side Error Handling:

Analyze the client’s error handling logic, especially around how it interprets server responses.
Ensure that the client correctly handles edge cases like timeout errors or invalid data.

Simulate Failure Scenarios:

Use controlled testing environments to simulate network failures and timing issues.
Verify the application's resilience under these conditions by intentionally causing delays, packet loss, etc.

#### Closing Summary
By systematically analyzing logs, inspecting network traffic, verifying server responses, ensuring proper client error handling, and simulating failure scenarios, we can identify and resolve the intermittent bug. This process involves a mix of detective work to locate the discrepancy and corrective measures to ensure robustness across all components.

#### Common Mistakes/Follow-Ups

Focusing solely on one component (client or server) without considering network interactions.
Overlooking subtle state changes or race conditions in asynchronous code.
Not thoroughly validating error handling mechanisms, leading to silent failures.

**Comparison:**

```
Component	Verification Points
Network	Packet capture, timing issues
Client	Error handling logic, log analysis
Server	Response codes, internal consistency
```

**Code:**

```javascript
// Example of enhanced logging in client-side request handling

async function makeRequest(url, method = 'GET', body) {
  try {
    const response = await fetch(url, { method, body });
    console.log(`Request sent to ${url} with method ${method}`); // Log request details
    const data = await response.json();
    if (response.ok) {
      console.log('Server responded successfully:', data);
    } else {
      console.error('Error: Server failed with status', response.status, data.message || data);
    }
  } catch (error) {
    console.error('Network error:', error);
  }
}
```

**Example input / output:**

```text
Input (Client): makeRequest("http://example.com/api/data", "POST", { key: 'value' });
Expected Output (Client): Successfully received data from server.
Actual Output (Client): Error occurred, retrying request...
Server Logs:
- [2023-10-05 14:30:00] Request sent to http://example.com/api/data with method POST
- [2023-10-05 14:30:01] Server responded successfully: {"message":"Success", "result":"expected_result"}
```

**Data:**

```json
{
  "request": {
    "url": "http://example.com/api/data",
    "method": "POST",
    "headers": {"Content-Type": "application/json"},
    "body": "{\"key\":\"value\"}"
  },
  "response": {
    "status": 200,
    "data": {"message":"Success", "result":"expected_result"}
  }
}
```

**Flow:**

1. Client Request
2. Network
3. Server Response Handling
4. Server Logs
5. Client Logs
6. Network Issues
7. Internal Errors
8. Error Handling
9. Success or Retry

