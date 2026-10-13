---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between synchronous and asynchronous ways of calling an LLM API, and when would a beginner reach for the async batch approach?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this context, we're comparing two methods of interacting with an AI language model (LLM) via its REST API: synchronous calls and asynchronous (async) batch calls. Synchronous calls are straightforward and wait for a response before proceeding, whereas async calls allow the system to continue executing other tasks while waiting for responses. For beginners, understanding these concepts is crucial as they can significantly impact performance and user experience.

**Comparison:**

```
	Synchronous Call	Async Batch Call
Blocking Behavior	Blocks execution until a response is received.	Does not block execution; can handle multiple requests concurrently.
Performance	Can be inefficient if many calls are made sequentially, as it waits for each one to complete before making the next.	Efficient in handling multiple requests simultaneously, reducing overall waiting time.
Use Case	Suitable when you need immediate responses from API calls.	Ideal for scenarios where multiple API calls can occur without blocking the execution of other tasks.
```

**Code:**

```
Example of synchronous API call using Python's requests library

import requests

def sync_api_call(url): response = requests.get(url) return response.json()

CODE
Copy
```python
# Example of asynchronous batch API calls using asyncio and aiohttp
import asyncio
import aiohttp

async def async_batch_call(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]
```

**Example input / output:**

```

Here's a dry-run input and output for the async batch call:

Input:

JSON
Copy
{
  "urls": ["https://api.example.com/v1/llm", "https://api.anotherexample.com/v2/llm"]
}

Output:

JSON
Copy
[
    {"text": "Example response from first API"},
    {"text": "Response from the second API"}
]
```

**Data:**

```json
{
  "urls": ["https://api.example.com/v1/llm", "https://api.anotherexample.com/v2/llm"]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is the request part of a batch?
5. Wait for single API response
6. End
7. Send multiple requestsconcurrently
8. Receive responses as theycome
9. Process each responseindividually
10. End

