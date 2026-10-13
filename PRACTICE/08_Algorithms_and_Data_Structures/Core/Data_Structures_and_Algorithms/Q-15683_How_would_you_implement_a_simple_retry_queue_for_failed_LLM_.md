---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a simple retry queue for failed LLM API calls with a maximum number of attempts, and which structures track pending items and attempt counts?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retry Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Retry_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a retry queue for failed Large Language Model (LLM) API calls, we need to ensure that each request can be retried up to a specified number of times if it fails. The key components include tracking pending items and their attempt counts, as well as managing retries without overwhelming the system.

#### Triage
Understanding the Problem

The core question here is about designing a data structure that efficiently handles retry logic for API calls. We need to consider:

A queue to manage incoming requests.
A mechanism to track how many times each request has been retried.
An upper limit on the number of retries.

#### First Principles Solution
Designing the Data Structures
Queue: To handle incoming API call requests, we can use a priority queue or a simple queue depending on whether order is critical (e.g., FIFO vs. highest-priority-first).
Pending Items and Attempt Counts: We need to keep track of each request's attempt count.
Step-by-Step Solution

Data Structures:

Use a dictionary to map API call IDs to their attempt counts.
Use a queue (FIFO or priority) to manage the sequence of requests.

#### Implementation
Steps:

When an API call fails, increment its retry count in the dictionary and re-enqueue it if retries are not exhausted.
If the maximum number of attempts is reached, remove the request from both the queue and the retry counter map.

Here's a simple Python implementation:

PYTHON
from collections import deque

class RetryQueue:
    def __init__(self, max_attempts=3):
        self.queue = deque()
        self.attempts_map = {}
        self.max_attempts = max_attempts

    def add_request(self, request_id):
        if request_id in self.attempts_map:
            # Increment retry count and re-enqueue
            self.attempts_map[request_id] += 1
            self.queue.append(request_id)
        else:
            # Add new request with one attempt
            self.attempts_map[request_id] = 1
            self.queue.append(request_id)

    def process_request(self):
        if not self.queue:
            return None

        request_id = self.queue.popleft()
        if self.attempts_map[request_id] > self.max_attempts:
            del self.attempts_map[request_id]
            return None

        # Simulate API call
        success = self.simulate_api_call(request_id)
        if not success:
            self.add_request(request_id)  # Retry the request
        else:
            self.attempts_map.pop(request_id, None)  # Remove from attempts map on success
            print(f"Request {request_id} successful after {self.attempts_map[request_id]} attempts")

    def simulate_api_call(self, request_id):
        # Simulate API call failure or success
        import random
        return random.choice([True, False])

# Example usage
retry_queue = RetryQueue(max_attempts=3)
retry_queue.add_request("req1")
retry_queue.add_request("req2")
for _ in range(5):  # Process requests multiple times to see retries
    retry_queue.process_request()

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: We've designed a simple retry queue that tracks the attempt count of each request and re-enqueues it for further retries until the maximum attempts are exhausted.

#### Common Mistakes
Forgetting to handle the case where an API call succeeds after multiple retries, which should remove the entry from the retry map.
Not managing edge cases like adding requests that already exist in the queue but with no additional attempt count incrementation.
Additional Considerations
Logging and Monitoring: Log failed attempts and success rates for monitoring purposes.
Rate Limiting: Prevent overwhelming the API by implementing rate limiting on retries.
Error Handling: Ensure robust error handling, especially if network issues or other failures occur during the retry process.

