---
type: "practice"
difficulty: "Deep"
---

# You built a Python webhook receiver for GitHub Actions events. Under a wave of concurrent pushes it drops events. How do you buffer, verify HMAC signatures, and process asynchronously without loss?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Graceful Shutdown](<../../../../TOOLS/Python/Python_Language/Graceful_Shutdown.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a high volume of GitHub Actions webhooks in Python while ensuring event buffering, HMAC signature verification, and asynchronous processing, we need to design a robust system that can scale under load. The key steps are to buffer incoming events, verify their integrity using HMAC signatures, and process them asynchronously without losing any data.

The question is probing your understanding of handling concurrent requests, state management, cryptography (HMAC), and asynchronous programming in Python. Key assumptions include:

We have a steady stream of webhook events from GitHub Actions.
The events are JSON payloads containing the necessary information for HMAC verification.
We need to ensure no event is lost under high load.

Let's break down each component step by step.

Buffering Events

To handle the influx of events, we can use an in-memory buffer that temporarily stores events until they can be processed. Python’s multiprocessing.Queue or a queue from the queue module can be used for this purpose. This allows us to manage concurrent access and ensure data integrity.

PYTHON
from multiprocessing import Queue

event_queue = Queue()
Verifying HMAC Signatures

GitHub Actions webhooks include an X-Hub-Signature-256 header containing the SHA-256 HMAC digest of the payload. We need to verify this signature against our secret key stored securely, such as in environment variables.

PYTHON
import hmac
import hashlib

def verify_hmac(secret_key: str, event_data: bytes, received_signature: str) -> bool:
    expected_signature = hmac.new(secret_key.encode(), msg=event_data, digestmod=hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected_signature, received_signature)
Asynchronous Processing

To ensure efficient and non-blocking processing, we can use asyncio with a TaskPoolExecutor. This approach allows us to handle multiple tasks concurrently without blocking the main thread.

PYTHON
import asyncio
from concurrent.futures import ThreadPoolExecutor

loop = asyncio.get_event_loop()
executor = ThreadPoolExecutor(max_workers=10)

@asyncio.coroutine
def process_event(event_data: bytes):
    # Process event here (e.g., log, send to another service)
    pass

async def handle_webhook():
    while True:
        event_data = await event_queue.get()
        if verify_hmac(os.getenv('GITHUB_WEBHOOK_SECRET'), event_data, os.getenv('X_HUB_SIGNATURE_256')):
            loop.run_in_executor(executor, process_event, event_data)
Full Solution

Combining these components, we get a complete solution that buffers events, verifies signatures, and processes them asynchronously.

PYTHON
import hmac
import hashlib
from multiprocessing import Queue
from concurrent.futures import ThreadPoolExecutor
import asyncio
import os

event_queue = Queue()
executor = ThreadPoolExecutor(max_workers=10)

def verify_hmac(secret_key: str, event_data: bytes, received_signature: str) -> bool:
    expected_signature = hmac.new(secret_key.encode(), msg=event_data, digestmod=hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected_signature, received_signature)

@asyncio.coroutine
def process_event(event_data: bytes):
    # Process event here (e.g., log, send to another service)
    pass

async def handle_webhook():
    while True:
        event_data = await event_queue.get()
        if verify_hmac(os.getenv('GITHUB_WEBHOOK_SECRET'), event_data, os.getenv('X_HUB_SIGNATURE_256')):
            loop.run_in_executor(executor, process_event, event_data)

# Example usage
async def main():
    asyncio.ensure_future(handle_webhook())
    # Simulate receiving events
    for _ in range(100):
        await asyncio.sleep(0.1)
        event_queue.put_nowait(b'{"action": "push"}')

loop.run_until_complete(main())

loop.close()

#### Closing Summary
and Common Mistakes

By following this approach, we ensure that our webhook receiver can handle high volumes of requests efficiently while maintaining data integrity through HMAC verification. Common mistakes include:

Not using an in-memory buffer to manage events.
Failing to verify the HMAC signature correctly.
Blocking the main thread during event processing.

**Code:**

```python
from multiprocessing import Queue
import hmac
import hashlib
from concurrent.futures import ThreadPoolExecutor
import asyncio
import os

event_queue = Queue()
executor = ThreadPoolExecutor(max_workers=10)

def verify_hmac(secret_key: str, event_data: bytes, received_signature: str) -> bool:
    expected_signature = hmac.new(secret_key.encode(), msg=event_data, digestmod=hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected_signature, received_signature)

@asyncio.coroutine
def process_event(event_data: bytes):
    # Process event here (e.g., log, send to another service)
    pass

async def handle_webhook():
    while True:
        event_data = await event_queue.get()
        if verify_hmac(os.getenv('GITHUB_WEBHOOK_SECRET'), event_data, os.getenv('X_HUB_SIGNATURE_256')):
            loop.run_in_executor(executor, process_event, event_data)

# Example usage
async def main():
    asyncio.ensure_future(handle_webhook())
    # Simulate receiving events
    for _ in range(100):
        await asyncio.sleep(0.1)
        event_queue.put_nowait(b'{"action": "push"}')

loop.run_until_complete(main())

loop.close()
```

**Flow:**

1. Valid
2. Invalid
3. Receive Event
4. Verify HMAC
5. Process Event
6. Reject Event
7. Buffer Event in Queue
8. Process Event Asynchronously

