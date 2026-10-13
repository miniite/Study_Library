---
type: "practice"
difficulty: "Intermediate"
---

# Your agent framework logs each step to a distributed trace, but tool sub-calls in a fan-out lose their parent context so traces are broken and you can't find the slow hop. Explain how context propagation should work across async tool calls and the sampling trade-off at high QPS.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that distributed traces remain intact when dealing with asynchronous tool calls, context propagation is crucial. Context includes information like a unique trace ID, span ID, parent span ID, and sampling decisions. In your scenario, the issue arises because sub-calls lose their parent context due to asynchronous behavior, leading to broken traces and difficulty in identifying performance bottlenecks.

#### Triaging the Question

The question is asking about how to propagate context across asynchronous calls in a distributed system while balancing high query per second (QPS) throughput. The key aspects are:

Context Propagation Mechanism: How to pass necessary information across function boundaries.
Sampling Trade-Offs: Balancing between ensuring all requests have complete traces and reducing the overhead of logging excessive data.
Context Propagation Across Asynchronous Calls
Key Ideas

To maintain trace integrity, context must be explicitly passed through asynchronous calls. This involves using a mechanism like Context objects in languages such as Go or ThreadLocal variables in Java to pass the necessary metadata (e.g., trace ID, span ID) between functions.

Explicit Context Passing: When making sub-calls, ensure that the context is copied and passed down with each call.
Global Context: Utilize global context objects available across threads or processes for capturing calls made from outside the immediate function scope.
Thread Safety: Ensure thread safety when using shared context objects.

#### Example
Implementation

In a language like Python, you can use the contextvars library to propagate context through asynchronous calls:

PYTHON
import contextvars

# Set up trace context variables
trace_id_var = contextvars.ContextVar('trace_id')
span_id_var = contextvars.ContextVar('span_id')

def make_async_call(parent_trace_id, parent_span_id):
    # Copy the parent context into the current context for the sub-call
    current_trace_id = trace_id_var.get()
    current_span_id = span_id_var.get()

    with contextvars.copy_context() as ctx:
        # Set the sub-call context variables before making the async call
        ctx.run(lambda: trace_id_var.set(parent_trace_id))
        ctx.run(lambda: span_id_var.set(parent_span_id))

        # Make the asynchronous call here and propagate the context within it
        result = await make_sub_call()

    return result

async def make_sub_call():
    sub_trace_id = generate_unique_id()
    sub_span_id = generate_unique_id()

    # Propagate context to sub-calls
    trace_id_var.set(sub_trace_id)
    span_id_var.set(sub_span_id)

    # Perform the actual work of the sub-call and return results
Sampling Trade-Offs at High QPS

At high QPS, logging every single request can lead to excessive overhead, compromising performance. Therefore, sampling strategies are crucial:

Random Sampling: Log a random subset of requests (e.g., 1%).
Rate Limiter: Implement rate-limiting on logs to ensure that the system does not get overwhelmed.
Hierarchical Tracing: Use a hierarchical approach where only critical paths are logged in detail, and other parts are sampled.

#### Example
Sampling Strategy
PYTHON
import random

def should_log_request():
    # Simple 1% sampling rate
    return random.random() < 0.01

async def make_async_call(parent_trace_id, parent_span_id):
    if not should_log_request():
        return

    # Copy the parent context into the current context for the sub-call
    current_trace_id = trace_id_var.get()
    current_span_id = span_id_var.get()

    with contextvars.copy_context() as ctx:
        # Set the sub-call context variables before making the async call
        ctx.run(lambda: trace_id_var.set(parent_trace_id))
        ctx.run(lambda: span_id_var.set(parent_span_id))

        result = await make_sub_call()

    return result

async def make_sub_call():
    if not should_log_request():
        return

    sub_trace_id = generate_unique_id()
    sub_span_id = generate_unique_id()

    # Propagate context to sub-calls
    trace_id_var.set(sub_trace_id)
    span_id_var.set(sub_span_id)

    # Perform the actual work of the sub-call and return results

#### Closing Summary
+ Common Mistakes/Follow-Ups

To ensure correct trace propagation in a distributed system, it’s essential to explicitly pass context through asynchronous calls. Sampling strategies must be implemented judiciously to manage high QPS while still capturing useful insights.

#### Common Mistakes

Ignoring Asynchronous Context: Not accounting for the need to pass context through async calls.
Sampling Too Aggressively: Logging too little data, leading to incomplete traces.
Overhead from Excessive Sampling: Not using efficient sampling techniques that can cause performance degradation.

#### Follow-Ups

Latency Analysis Tools: Discuss tools like Jaeger or Zipkin for visualizing and analyzing traces.
Error Handling in Context Propagation: How to handle errors when context is lost or not properly propagated.
Performance Metrics: Monitor the impact of context propagation on system performance.

**Comparison:**

```
Trace ID	Span ID
1234567890	0987654321
```

**Code:**

```python
import contextvars

# Example context variables
trace_id_var = contextvars.ContextVar('trace_id')
span_id_var = contextvars.ContextVar('span_id')

def make_async_call(parent_trace_id, parent_span_id):
    # Copy the parent context into the current context for the sub-call
    current_trace_id = trace_id_var.get()
    current_span_id = span_id_var.get()

    with contextvars.copy_context() as ctx:
        # Set the sub-call context variables before making the async call
        ctx.run(lambda: trace_id_var.set(parent_trace_id))
        ctx.run(lambda: span_id_var.set(parent_span_id))

        result = await make_sub_call()

    return result

async def make_sub_call():
    sub_trace_id = generate_unique_id()
    sub_span_id = generate_unique_id()

    # Propagate context to sub-calls
    trace_id_var.set(sub_trace_id)
    span_id_var.set(sub_span_id)

    # Perform the actual work of the sub-call and return results
```

**Example input / output:**

```text
# Example input and output for trace context propagation

# Input:
parent_trace_id = "1234567890"
parent_span_id = "0987654321"

# Output after processing the async call with sampled logging
result = make_async_call(parent_trace_id, parent_span_id)
```

**Data:**

```json
{
  "trace_id": "1234567890",
  "span_id": "0987654321"
}
```

**Flow:**

1. No
2. Yes
3. Make Async Call
4. Context Present?
5. Log Request
6. Generate Sub-Call Context
7. Perform Sub-Call
8. Copy Context to Sub-Call
9. Perform Sub-Call

