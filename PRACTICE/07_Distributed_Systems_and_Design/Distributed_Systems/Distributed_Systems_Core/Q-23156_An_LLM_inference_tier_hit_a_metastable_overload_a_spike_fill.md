---
type: "practice"
difficulty: "Deep"
---

# An LLM inference tier hit a metastable overload: a spike filled the queue, clients timed out and retried, and the queue never drained even after traffic dropped. Explain the sustaining feedback loop specific to token generation and how admission control plus request cancellation recover it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The metastable overload in an LLM inference tier results from a negative feedback loop where backlogged requests cause increased retries, exacerbating queue congestion. Admission control and request cancellation are effective strategies to mitigate this issue by balancing the system's load and reducing unnecessary processing.

#### Triage

This question probes your understanding of distributed systems, specifically focusing on how token generation processes can lead to metastable states under heavy load. The key aspect is identifying the sustaining feedback loop that causes the queue never to drain even after traffic drops. Additionally, the interviewer may be evaluating your knowledge of admission control and request cancellation mechanisms.

#### First-Principles Solution
Understanding Token Generation and Queues

Token generation in LLMs involves multiple steps: receiving a request, processing it through models, generating responses, and sending back results to clients. The system typically uses queues to manage these requests efficiently. When the load is high, the queue backs up, leading to a situation where more incoming requests are added than can be processed.

Identifying the Sustaining Feedback Loop
Initial Overload: An initial spike in traffic causes the request queue to fill up faster than it can be drained.
Client Retries: Clients receive timeouts and retry their requests, adding new entries to the already backlogged queue.
Queue Congestion: The backlog continues to grow as more retries are issued, causing further delays and increased client pressure to retry.
Sustaining Feedback Loop: This cycle is self-reinforcing because each additional retry adds more load on an already strained system, making it harder for the queue to clear.
Admission Control

Admission control involves limiting the number of requests allowed into the system based on current capacity and availability. By implementing a threshold, such as dropping or delaying new requests when the queue exceeds a certain level, we can prevent further congestion:

PYTHON
def process_request(request):
    global request_queue_length

    if request_queue_length < max_queue_length:
        request_queue.append(request)
        request_queue_length += 1
    else:
        # Log and optionally delay or drop the request
        log('Queue is full, dropping request.')
Request Cancellation

Request cancellation involves automatically canceling or aborting requests that have been in the queue for too long. This ensures that only relevant and timely requests are processed, reducing unnecessary load on the system:

PYTHON
def process_request(request):
    global request_queue_length

    if request_queue_length < max_queue_length:
        request_queue.append(request)
        request_queue_length += 1
    else:
        # Log and optionally delay or drop the request
        log('Queue is full, dropping request.')

    start_time = time.time()
    while True:
        elapsed_time = time.time() - start_time
        if elapsed_time > max_wait_time:
            cancel_request(request)
            break

#### Closing Summary

Admission control and request cancellation are crucial mechanisms to manage the metastable state of a distributed system, especially under heavy load. By limiting new requests when the queue is full and cancelling stale requests, we can mitigate the sustaining feedback loop that leads to persistent congestion.

#### Common Mistakes
and Follow-Ups
Overlooked Feedback Loops: Failing to recognize how client retries exacerbate the problem.
Insufficient Admission Control: Not setting appropriate thresholds for queue length or request rate.
Lack of Request Cancellation Logic: Ignoring the importance of aborting stale requests.

#### Follow-ups
might include:

Discussing alternative strategies such as dynamic scaling or prioritization based on request urgency.
Exploring trade-offs between admission control and the user experience (e.g., increased latency due to delays).

#### Cons
idering edge cases like retries with exponential backoff, which can further complicate the feedback loop.

**Comparison:**

```
Request	Start Time	Max Wait Time	Status
Req1	0	60	Processed
Req2	10	60	Processed
Req3	59	60	Canceled
```

**Code:**

```python
def process_request(request):
    global request_queue_length

    if request_queue_length < max_queue_length:
        request_queue.append(request)
        request_queue_length += 1
    else:
        # Log and optionally delay or drop the request
        log('Queue is full, dropping request.')
```

**Example input / output:**

```text
**Input**
- `max_queue_length = 50`
- Initial queue length: 48

**Output**
1. New request arrives.
2. Queue length is 49 (less than max).
3. Process the new request.

**Input**
- `max_queue_length = 50`
- Initial queue length: 50

**Output**
1. New request arrives.
2. Queue length is 51 (exceeds max).
3. Log and drop the new request.

**Input**
- `max_queue_length = 50`
- Request with `start_time` = 0, `max_wait_time` = 60

**Output**
1. Processed a request.
2. Wait time is 10 seconds (within max).
3. Continue processing.
4. Wait time reaches 60 seconds (exceeds max).
5. Cancel the request.
```

**Data:**

```json
{
  "request_queue": [],
  "max_queue_length": 50,
  "request_queue_length": 0
}
```

**Flow:**

1. Yes
2. No
3. New Request
4. Queue Full?
5. Drop Request
6. Process Request
7. Log and Drop
8. Process Request
9. Cancel Request if Wait TimeExceeds Max
10. Abort Processing

