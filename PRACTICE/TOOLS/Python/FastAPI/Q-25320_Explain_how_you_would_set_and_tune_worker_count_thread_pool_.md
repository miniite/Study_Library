---
type: "practice"
difficulty: "Deep"
---

# Explain how you would set and tune worker count, thread pool size, and connection limits for a FastAPI service that mixes I/O-bound and occasional CPU-bound endpoints.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limits and Backpressure](<../../../../TOOLS/Python/FastAPI/Rate_Limits_and_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure efficient performance in a FastAPI service handling both I/O-bound and CPU-bound endpoints, we need to carefully set up worker counts, thread pool sizes, and connection limits. This involves understanding the nature of your application's workloads and using tools like gunicorn or uvicorn for server setup.

#### Triaging the Question

The core of this question revolves around optimizing FastAPI performance by balancing concurrency and resource utilization. The interviewer is probing your understanding of how different configurations affect the behavior of a FastAPI service, particularly in mixed workloads where some endpoints require more CPU resources than others while most are I/O-bound.

We assume:

You have an application with both CPU and I/O-bound endpoints.
You use uvicorn or gunicorn as your WSGI server for running the FastAPI service.
Your application runs on a multi-core machine, allowing for parallel execution.

#### First-Principles Solution
Worker Count: This controls how many worker processes should be spawned to handle concurrent requests. For mixed workloads, a good starting point is to set this value slightly above the number of CPU cores available (e.g., if you have 4 cores, start with 5 workers).
Thread Pool Size: Each worker has its own thread pool for I/O-bound tasks like database access or network requests. This should be tuned based on the expected load and the type of I/O operations.
Connection Limits: These are related to both server and client-side configurations, affecting how many connections can be made simultaneously. Server limits ensure you don’t overload your backend, while client-side limits prevent overwhelming your FastAPI service with too many requests.
Step-by-Step Setup

Worker Count:

Use worker_class in uvicorn to choose between threads or processes.
For a mix of I/O and CPU-bound tasks, using multiple worker processes might be better because they can handle more tasks concurrently without blocking on I/O.

Thread Pool Size:

Use the --workers option with gunicorn if you are using process workers.
For thread workers, use the --worker-class option and set thread_pool_size.

Connection Limits:

Server-side: Configure Gunicorn settings like keep-alive to manage client connections.
Client-side: Ensure your client (e.g., a web browser or API client) does not open too many connections.

#### Example
Configuration

Here’s an example configuration in uvicorn for a FastAPI service:

BASH
# gunicorn.config.py
gunicorn --workers 5 \
         --worker-class uvicorn.workers.UvicornWorker \
         --timeout 300 \
         app:app

In this setup:

--workers 5: Starts 5 worker processes.
--worker-class UvicornWorker: Uses the default event loop with a single thread per process, which is suitable for mixed workloads.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Setting too many workers can lead to resource contention if not matched with proper CPU or I/O capabilities.
Not setting connection limits properly can result in overwhelming the server or client.

#### Follow-Ups
Discuss how to monitor and adjust these settings dynamically based on real-time performance metrics.
Talk about using uvloop for better I/O performance.
Explore other configurations like asynchronous workers if your application is highly I/O-bound.

**Code:**

```bash
# Example gunicorn command
gunicorn --workers 5 \
         --worker-class uvicorn.workers.UvicornWorker \
         --timeout 300 \
         app:app
```

