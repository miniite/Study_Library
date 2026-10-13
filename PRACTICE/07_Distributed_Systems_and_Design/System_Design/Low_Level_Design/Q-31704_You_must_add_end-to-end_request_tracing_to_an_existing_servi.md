---
type: "practice"
difficulty: "Deep"
---

# You must add end-to-end request tracing to an existing service without polluting every method signature with a trace context; how do you design context propagation cleanly across async boundaries?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of adding end-to-end request tracing to an existing service without cluttering method signatures, we need to focus on designing a robust and efficient context propagation mechanism that can work seamlessly across asynchronous boundaries. The key idea is to encapsulate trace context into a lightweight object that can be easily passed through function calls and asynchronous operations.

#### Triage

The question revolves around implementing end-to-end request tracing by ensuring that the trace context is propagated correctly without modifying every method signature. This means we need to design an effective way of passing context between synchronous and asynchronous functions, which is crucial for maintaining performance while ensuring complete traceability.

#### Assumptions
The existing service involves both synchronous and asynchronous operations.
We aim to minimize impact on the codebase by avoiding polluting method signatures with tracing-related parameters.
The service uses a common framework or language (e.g., Node.js, Java) that supports asynchronous programming.

#### First-Principles Solution

Context Object Design:

Create a lightweight TraceContext object to encapsulate all trace information needed for end-to-end tracing.
Ensure the TraceContext object is thread-safe and can be serialized/deserialized as necessary.

Interception Mechanism:

Implement a global interception mechanism using middleware or decorators that automatically injects the TraceContext into functions that handle requests.
Use AOP (Aspect-Oriented Programming) techniques to inject context at the entry point of request handling and propagate it through asynchronous boundaries.

Async Boundaries Handling:

Ensure that when a synchronous function calls an asynchronous function, the TraceContext is correctly passed along using callbacks or promises.
For external services or microservices interactions, use middleware or libraries to pass the context via headers or other means.

Instrumentation and Logging:

Introduce instrumentation points in critical sections of code where spans are started and ended.
Use structured logging (e.g., JSON format) to log trace information, making it easier to correlate logs with traces.

#### Performance Considerations

Minimize the overhead by optimizing the TraceContext object and ensuring that context propagation is lightweight.
Profile and optimize as necessary to ensure performance is not significantly impacted.

#### Implementation
Steps

Define Trace Context Object:

PYTHON
class TraceContext:
    def __init__(self, trace_id, span_id):
        self.trace_id = trace_id
        self.span_id = span_id

    @staticmethod
    def from_header(header_value: str):
        # Implementation to parse header value and initialize context
        pass

Interception Middleware:

PYTHON
def with_trace_context(middleware_func):
    async def wrapper(req, res, next):
        trace_ctx = TraceContext.from_header(req.headers.get('X-Trace-ID'))
        try:
            await middleware_func(req, res, next, trace_ctx)
        except Exception as e:
            # Handle exceptions and ensure context is propagated
            pass
    return wrapper

@with_trace_context
async def handle_request(request, response, next, trace_ctx):
    # Process request with trace context
    ...
    await next(request, response, trace_ctx)  # Pass the context to the next middleware
    ...

Async Function Handling:

PYTHON
@asyncio.coroutine
def async_function(trace_ctx):
    # Perform asynchronous operations and propagate trace context
    ...
    result = yield from external_service(trace_ctx)
    ...
    return result

@with_trace_context
async def handle_request(request, response, next, trace_ctx):
    try:
        await asyncio.wait_for(async_function(trace_ctx), timeout=5)
    except asyncio.TimeoutError:
        log.error("Async operation timed out")

Instrumentation and Logging:

PYTHON
import logging

class Tracer:
    def start_span(self, trace_id, span_id):
        # Log the start of a new span with context information
        pass

    def end_span(self, span_id):
        # Log the end of a span
        pass

tracer = Tracer()

@with_trace_context
async def handle_request(request, response, next, trace_ctx):
    try:
        await asyncio.wait_for(async_function(trace_ctx), timeout=5)
    except asyncio.TimeoutError:
        log.error("Async operation timed out")

    tracer.end_span(trace_ctx.trace_id)  # Log the end of the span

#### Closing Summary
and Common Mistakes

This design ensures that trace context is propagated cleanly across both synchronous and asynchronous boundaries without polluting method signatures. Key challenges include ensuring thread safety, minimizing performance overhead, and handling edge cases such as timeouts and unhandled exceptions.

#### Common mistakes
to avoid:

Failing to handle asynchronous callbacks properly.
Not using a lightweight TraceContext object.
Ignoring the need for global interception mechanisms.
Overlooking the importance of logging detailed trace information.

**Code:**

```python
class TraceContext:
    def __init__(self, trace_id, span_id):
        self.trace_id = trace_id
        self.span_id = span_id

    @staticmethod
    def from_header(header_value: str):
        # Implementation to parse header value and initialize context
        pass

def with_trace_context(middleware_func):
    async def wrapper(req, res, next):
        trace_ctx = TraceContext.from_header(req.headers.get('X-Trace-ID'))
        try:
            await middleware_func(req, res, next, trace_ctx)
        except Exception as e:
            # Handle exceptions and ensure context is propagated
            pass
    return wrapper

@with_trace_context
async def handle_request(request, response, next, trace_ctx):
    # Process request with trace context
    ...
    await next(request, response, trace_ctx)  # Pass the context to the next middleware
    ...

@asyncio.coroutine
def async_function(trace_ctx):
    # Perform asynchronous operations and propagate trace context
    ...
    result = yield from external_service(trace_ctx)
    ...
    return result

class Tracer:
    def start_span(self, trace_id, span_id):
        # Log the start of a new span with context information
        pass

    def end_span(self, span_id):
        # Log the end of a span
        pass

tracer = Tracer()

@with_trace_context
async def handle_request(request, response, next, trace_ctx):
    try:
        await asyncio.wait_for(async_function(trace_ctx), timeout=5)
    except asyncio.TimeoutError:
        log.error("Async operation timed out")

    tracer.end_span(trace_ctx.trace_id)  # Log the end of the span
```

**Flow:**

1. Yes
2. No
3. Request Received
4. Intercept Request with TraceContext
5. Is Synchronous Call?
6. Process Synchronously,Propagate Trace Context
7. Invoke Asynchronous Functionwith Trace Context
8. Handle Async Result and LogSpan End

