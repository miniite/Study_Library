---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between blocking, non-blocking, and asynchronous I/O for a backend that fans out to 50 downstreams per request. How does the concurrency model change tail latency under partial slowness?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a backend system that needs to fan out to 50 downstream services per request and understand how different concurrency models—blocking, non-blocking, and asynchronous I/O—affect tail latency under partial slowness conditions.

Blocking vs Non-Blocking vs Asynchronous I/O

Blocking: This model is straightforward but can lead to inefficiencies. When a thread performs an I/O operation (e.g., sending data to one of the 50 downstreams), it waits for a response and cannot do anything else until that operation completes.

Non-Blocking: Non-blocking models ensure that when a request is sent, the thread can continue with other tasks while waiting for the response from the downstream service. However, this doesn't necessarily mean the response will be handled immediately; it just means the thread is not blocked.

Asynchronous I/O (Async I/O): This model uses non-blocking calls but goes a step further by leveraging callbacks or coroutines to handle responses as soon as they are available. Asynchronous operations can significantly reduce waiting time for the first response, potentially improving overall system performance and responsiveness.

Concurrency Models in Practice

Let's consider how these models behave under partial slowness conditions, where some of the 50 downstream services might be slower than others.

Blocking Model: If any one of the 50 requests blocks a thread (due to a slow response), all other threads that are trying to send or receive data will also block until the response is available. This can lead to a situation where multiple threads are waiting, significantly increasing tail latency for subsequent requests.

Non-Blocking Model: With non-blocking I/O, even if one of the 50 downstream services is slow, it doesn't block the entire thread. The thread continues to process other tasks, but it still waits for the response before proceeding further with that particular task. This can help reduce overall latency, but not as effectively as asynchronous models in cases where many services are slow.

Asynchronous I/O Model: Asynchronous I/O allows the backend to handle responses as soon as they come in without waiting for all 50 responses. If one of the downstream services is slow, it doesn't block other requests from being processed. The system can start processing faster responses and complete tasks more quickly, leading to better tail latency.

Tail Latency Analysis

To understand how these models affect tail latency:

Blocking: Tail latency is high because a single slow response can block multiple threads.
Non-Blocking: Tail latency is improved over blocking but not as much as with asynchronous I/O. The system still waits for responses, though in a more efficient manner.
Asynchronous I/O: Tail latency is significantly reduced because the backend can start processing faster responses immediately and continue to handle other requests without waiting.

#### Closing Summary

In summary, under partial slowness conditions, an asynchronous I/O model generally provides the best performance in terms of tail latency. It allows for more efficient use of threads and resources by handling responses as soon as they are available. However, the choice between blocking, non-blocking, or async I/O depends on the specific requirements of your system, including the cost of thread management and the importance of minimizing tail latency.

#### Common Mistakes/Follow-Ups

Forgetting to Consider Thread Pooling: Asynchronous models require proper thread pool management to avoid excessive context switching.
Neglecting Error Handling: Slow responses or failures in downstream services need robust error handling mechanisms, especially with async I/O.
Ignoring Performance Metrics: Regularly monitoring and tuning performance metrics can help identify bottlenecks and optimize the system further.

**Comparison:**

```
Concurrency Model	Tail Latency	Thread Utilization
Blocking	High	Low
Non-blocking	Medium	Moderate
Asynchronous I/O	Low	High
```

**Code:**

```python
# Example of a simple non-blocking request using asyncio
import asyncio

async def send_request(url):
    await asyncio.sleep(1)  # Simulating an I/O operation

async def main():
    tasks = [send_request(f'http://downstream{idx}.com') for idx in range(50)]
    await asyncio.gather(*tasks)

# Example of a blocking request
def send_blocking_request(url):
    import time
    time.sleep(1)  # Simulating an I/O operation

# Example of an async I/O pattern using aiohttp
import aiohttp

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main_async():
    urls = [f'http://downstream{idx}.com' for idx in range(50)]
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        responses = await asyncio.gather(*tasks)
```

**Example input / output:**

```text
Input:
- List of 50 downstream service URLs

Output:
- Responses from all 50 downstream services in an order determined by their availability
```

**Data:**

```json
{
  "urls": ["http://downstream1.com", "http://downstream2.com", ..., "http://downstream50.com"]
}
```

**Flow:**

1. Blocking Model
2. Block Thread
3. Wait for Response
4. Continue Processing
5. Handle Partial Slowness
6. Multiple Threads Block
7. High Tail Latency
8. Non-Blocking Model
9. Continues Processing
10. Wait for Response
11. Continue Processing
12. Asynchronous I/O Model
13. Handle ResponsesImmediately
14. Low Tail Latency

