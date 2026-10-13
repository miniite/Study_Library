---
type: "practice"
difficulty: "Intermediate"
---

# A user abandons a long streaming generation but your GPU keeps producing tokens no one reads, wasting capacity. Explain how you'd propagate client-disconnect cancellation from the gateway through the router to the model worker to free the KV slot promptly.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a user abandoning a long streaming generation but your GPU keeping on producing tokens, we need to ensure that client-disconnect cancellations are propagated from the gateway through the router to the model worker in a timely manner. This involves understanding the architecture and implementing proper cancellation mechanisms.

Firstly, let's break down the components involved:

Gateway: The entry point for user requests.
Router: Routes the request to appropriate workers based on some criteria (e.g., load balancing).
Model Worker: The actual worker that processes the request and generates tokens using a model.

The key challenge is ensuring that once a client disconnects, all downstream components also stop processing. This requires careful design of asynchronous flow control and cancellation propagation mechanisms.

#### Triaging the Question

What the question is probing:

Understanding distributed systems architecture.
Knowledge of how to implement robust error handling and flow control in asynchronous systems.
Ability to reason about system state and propagate changes efficiently.

#### Assumptions

The system uses an event-driven or async model for processing requests.
There is a way to detect client disconnection (e.g., HTTP connection termination, socket closure).
Cancellation tokens or mechanisms are used in the workers.

#### First-Principles Solution

Gateway Detection of Disconnection:

When the gateway detects that the user has disconnected (e.g., by timing out or receiving a disconnection signal), it needs to generate and send a cancellation token.

Router Handling Cancellation Token:

The router receives the cancellation token from the gateway and sends it to all associated workers.

Model Worker Processing Cancellation:

Each model worker must be able to receive and process this cancellation token. When a worker receives the token, it should stop any ongoing tasks, release resources (like GPU tokens), and free up the slot in the key-value store.
Detailed Steps

Gateway Implementation:

PYTHON
async def handle_request(request):
    # Start a task to process the request
    processing_task = asyncio.create_task(process_request(request))

    try:
        await processing_task  # Wait for the task to complete normally
    except asyncio.CancelledError:
        logging.info("Request was cancelled by client.")
        return "Cancelled"

Router Implementation:

PYTHON
async def route_request(request):
    worker_id = load_balancing_algorithm(request)

    # Send the request to the worker and wait for a response
    try:
        await send_to_worker(worker_id, request)
        return await get_response_from_worker(worker_id)
    except asyncio.CancelledError:
        logging.info("Request was cancelled by client.")
        return "Cancelled"

Model Worker Implementation:

PYTHON
async def process_request(request):
    try:
        # Process the request and generate tokens
        result = model.generate_tokens(input_data)
        return result
    except asyncio.CancelledError:
        logging.info("Worker received a cancellation token.")
        # Release resources and free up slot in the KV store
        release_resources()

#### Closing Summary

In summary, to handle client disconnection efficiently in a distributed system, we need to implement proper flow control mechanisms at each layer. The gateway detects the disconnection and sends a cancellation token upstream, while the router propagates it down to the model worker. Each component must be able to recognize this token and handle it appropriately by stopping processing and releasing resources.

#### Common Mistakes/Follow-Ups
Not Handling Cancellation Properly: Failing to implement or properly handle asyncio.CancelledError can lead to resource leaks.
Inefficient Resource Release: Not freeing up GPU tokens or other resources promptly after cancellation can result in unnecessary waste.
Routing Issues: If the router does not correctly propagate the token, some workers might continue processing even though they should have been cancelled.

**Code:**

```python
async def handle_request(request):
    # Start a task to process the request
    processing_task = asyncio.create_task(process_request(request))

    try:
        await processing_task  # Wait for the task to complete normally
    except asyncio.CancelledError:
        logging.info("Request was cancelled by client.")
        return "Cancelled"

async def route_request(request):
    worker_id = load_balancing_algorithm(request)

    # Send the request to the worker and wait for a response
    try:
        await send_to_worker(worker_id, request)
        return await get_response_from_worker(worker_id)
    except asyncio.CancelledError:
        logging.info("Request was cancelled by client.")
        return "Cancelled"

async def process_request(request):
    try:
        # Process the request and generate tokens
        result = model.generate_tokens(input_data)
        return result
    except asyncio.CancelledError:
        logging.info("Worker received a cancellation token.")
        # Release resources and free up slot in the KV store
        release_resources()
```

