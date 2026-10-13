---
type: "practice"
difficulty: "Deep"
---

# You need to fan out 500 outbound HTTP calls with per-host connection pooling and timeouts. Compare aiohttp and httpx, and how you'd bound concurrency and reuse the client.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To fan out 500 outbound HTTP calls in a Python application while leveraging connection pooling and managing concurrency, we need to compare two popular libraries: aiohttp and httpx. Both offer robust solutions but have different features and trade-offs that make them suitable for various use cases. Here, I’ll explore the differences, outline a strategy using both libraries, and discuss how to manage concurrency and connection pooling.

Firstly, let’s clarify what we mean by “fan out” 500 outbound HTTP calls. This implies creating a set of parallel requests to different hosts or services. We need to ensure that these connections are reused where possible to optimize performance and adhere to network resource constraints.

Triaging the Problem
Question: Compare aiohttp and httpx, focusing on connection pooling, handling timeouts, and managing concurrency.
Interviewer’s Probing: The interviewer is likely interested in your understanding of asynchronous programming in Python, familiarity with these libraries, and your ability to design a scalable solution that handles large numbers of concurrent requests efficiently.

#### Assumptions
We are working in an environment where both aiohttp and httpx are installed.
The 500 HTTP calls will be made to different services or endpoints.
We need to handle timeouts appropriately to avoid hanging connections.
We aim for a solution that efficiently reuses connections while managing concurrency.

#### First-Principles Solution
aiohttp

aiohttp is an asynchronous HTTP client/server framework for Python. It uses the asyncio library and provides features like connection pooling, automatic keep-alive management, and timeouts.

Key Features of aiohttp:

Connection Pooling: aiohttp.ClientSession manages a pool of connections to each host.
Timeouts: Built-in support for setting request timeouts.
Concurrency Control: Uses Python’s asyncio event loop to manage coroutines effectively.

Here is an example using aiohttp:

PYTHON
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = ["https://api.example.com/data" for _ in range(500)]
    tasks = []

    async with aiohttp.ClientSession() as session:
        for url in urls:
            task = asyncio.create_task(fetch(session, url))
            tasks.append(task)

        responses = await asyncio.gather(*tasks)
        print(responses)

asyncio.run(main())
httpx

httpx is another popular HTTP client library that supports both synchronous and asynchronous operations. It provides a similar level of functionality as aiohttp.

Key Features of httpx:

Connection Pooling: httpx.AsyncClient manages connection pooling.
Timeouts: Built-in support for setting request timeouts.
Concurrency Control: Can be used in an asynchronous context via asyncio.run.

Here is an example using httpx:

PYTHON
import httpx

async def fetch(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text

async def main():
    urls = ["https://api.example.com/data" for _ in range(500)]
    tasks = []

    for url in urls:
        task = asyncio.create_task(fetch(url))
        tasks.append(task)

    responses = await asyncio.gather(*tasks)
    print(responses)

if __name__ == "__main__":
    asyncio.run(main())
Managing Concurrency and Reuse

Both aiohttp and httpx manage concurrency through connection pooling, but the specifics can vary. For instance, in aiohttp, you might need to configure the pool size explicitly:

PYTHON
async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(limit=100)) as session:
    # ...

In contrast, httpx automatically manages connection pooling and does not require explicit configuration:

PYTHON
async with httpx.AsyncClient() as client:
    # ...

#### Closing Summary

Both aiohttp and httpx are excellent choices for making 500 concurrent HTTP requests. However, if you prioritize simplicity and ease of use, httpx is a good choice due to its straightforward API and built-in features. If you need more granular control over the connection pool or additional async capabilities, aiohttp might be the better option.

#### Common Mistakes/Follow-Ups
Not Configuring Connection Limits: Failing to set appropriate limits on the number of concurrent connections can lead to resource exhaustion.
Ignoring Timeouts: Not setting timeouts can result in hanging requests that block your application.
Lack of Error Handling: Failing to handle exceptions or edge cases properly can cause your application to crash.

If asked follow-ups, be ready to discuss the performance implications of different settings and how you would monitor and test the system under load.

**Code:**

```python
# aiohttp example
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = ["https://api.example.com/data" for _ in range(500)]
    tasks = []

    # Set a connection limit of 100
    async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(limit=100)) as session:
        for url in urls:
            task = asyncio.create_task(fetch(session, url))
            tasks.append(task)

        responses = await asyncio.gather(*tasks)
        print(responses)

asyncio.run(main())

# httpx example
import httpx

async def fetch(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text

async def main():
    urls = ["https://api.example.com/data" for _ in range(500)]
    tasks = []

    for url in urls:
        task = asyncio.create_task(fetch(url))
        tasks.append(task)

    responses = await asyncio.gather(*tasks)
    print(responses)

if __name__ == "__main__":
    asyncio.run(main())
```

**Example input / output:**

```text
# Example input and output for main()
input:
- urls = ["https://api.example.com/data"] * 500

output:
- responses = [response.text() for _ in range(500)]
```

**Data:**

```json
{
  "urls": [
    "https://api.example.com/data",
    "https://api.example.com/data",
    ...
    "https://api.example.com/data" // 500 times
  ]
}
```

