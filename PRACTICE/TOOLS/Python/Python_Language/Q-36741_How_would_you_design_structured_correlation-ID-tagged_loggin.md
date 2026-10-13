---
type: "practice"
difficulty: "Deep"
---

# How would you design structured, correlation-ID-tagged logging across an async Python service so a single request's logs can be traced through concurrent tasks?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Request Logging for Models](<../../../../TOOLS/Python/Python_Language/Request_Logging_for_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design structured, correlation-ID-tagged logging in an async Python service, we need to implement a mechanism that assigns unique IDs to each incoming request and propagates these IDs throughout all asynchronous tasks related to that request. This ensures that logs can be traced back to their originating requests, making debugging and monitoring easier.

#### Triaging the Question

The core of this question is about ensuring that logs are properly correlated for each request in an async Python service. The interviewer is probing your understanding of how to implement structured logging, asynchronous programming, and how to manage state (like correlation IDs) across multiple concurrent tasks. Key assumptions include:

The service uses Python's asyncio framework.
Logs need to be tagged with a unique ID that traces the request.
The system should handle thousands of requests concurrently without loss of context.

#### First Principles Solution
Correlation ID Generation: Each incoming request will receive a unique correlation ID upon entry.
Logging Framework Integration: We'll use a logging framework (e.g., structlog) to automatically tag logs with the current correlation ID.
Async Context Management: We’ll manage the context across asynchronous tasks using thread-local storage or other mechanisms.
Step 1: Correlation ID Generation

We need to generate a unique correlation ID for each request and store it in a central place where all tasks can access it. For simplicity, we'll use an asyncio task-local variable:

PYTHON
import asyncio
from uuid import UUID, uuid4

correlation_id = asyncio.get_event_loop().set_task_info({'correlation_id': None})
Step 2: Logging with Correlation ID

Using a structured logging framework like structlog, we can automatically tag logs with the correlation ID:

PYTHON
import structlog

# Initialize structlog with correlation ID
def configure_structlog():
    structlog.configure(processors=[
        structlog.processors.KeyValueRenderer(key_order=['correlation_id']),
        # Other processors...
    ])

configure_structlog()
Step 3: Propagating Correlation ID in Async Tasks

We can use a context manager to ensure the correlation ID is set properly for each task and reset it when done:

PYTHON
class CorrelationContext:
    def __init__(self, correlation_id):
        self.correlation_id = correlation_id

    async def __aenter__(self):
        # Set correlation ID in asyncio's task info
        task_info = asyncio.get_event_loop().get_task_info(asyncio.current_task())
        task_info['correlation_id'] = str(self.correlation_id)

    async def __aexit__(self, exc_type, exc_value, traceback):
        # Reset correlation ID when exiting the context
        if 'correlation_id' in task_info:
            del task_info['correlation_id']

# Usage example
async def handle_request(request):
    with CorrelationContext(uuid4()):
        log.info('Handling request', correlation_id=uuid4())
        await some_async_operation()
Step 4: Handling Edge Cases
Concurrency Issues: Ensure the context manager correctly handles concurrent tasks.
State Management: Use thread-local storage or other mechanisms to manage state across async tasks.

#### Closing Summary
and Common Mistakes

To summarize, we've designed a structured logging system in an async Python service by generating unique correlation IDs for each request, propagating these IDs through asynchronous tasks using context managers, and integrating them into our logs. The key is ensuring the correlation ID remains consistent and can be traced throughout all related operations.

#### Common mistakes
include:

Forgetting to set or reset the correlation ID within tasks.
Not handling edge cases like concurrent tasks or large-scale deployments.
Using global state instead of thread-local storage, leading to potential race conditions.

**Code:**

```python
import asyncio
from uuid import UUID, uuid4

# Initialize task info for correlation ID
correlation_id = asyncio.get_event_loop().set_task_info({'correlation_id': None})

class CorrelationContext:
    def __init__(self, correlation_id):
        self.correlation_id = correlation_id

    async def __aenter__(self):
        task_info = asyncio.get_event_loop().get_task_info(asyncio.current_task())
        task_info['correlation_id'] = str(self.correlation_id)

    async def __aexit__(self, exc_type, exc_value, traceback):
        if 'correlation_id' in task_info:
            del task_info['correlation_id']

# Usage example
async def handle_request(request):
    with CorrelationContext(uuid4()):
        log.info('Handling request', correlation_id=uuid4())
        await some_async_operation()
```

**Flow:**

1. Request
2. Generate Correlation ID
3. Set Task Info
4. Handle Request
5. Begin Async Operations
6. Propagate Correlation ID
7. Perform Operation
8. End Async Operations
9. Reset Task Info

