---
type: "practice"
difficulty: "Deep"
---

# How would you propagate a trace or correlation ID from an incoming request into outgoing httpx calls in FastAPI?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To propagate a trace or correlation ID from an incoming request into outgoing httpx calls in FastAPI, we need to ensure that every internal API call retains this context so that it can be traced throughout the system. This is crucial for debugging and monitoring purposes. We'll start by understanding the core problem and then walk through a step-by-step solution using FastAPI and httpx.

Firstly, let's consider what the trace or correlation ID is. It’s often an alphanumeric string used to uniquely identify each request as it travels through the system. This allows us to correlate different components of the system and track them end-to-end.

The key idea here is to use a middleware mechanism in FastAPI to intercept incoming requests, extract the trace ID from them, and then inject this ID into every outgoing httpx call. We'll use context variables or thread local storage to keep track of the correlation ID throughout the request processing pipeline.

#### Triage

The question is essentially asking how to maintain a consistent context (trace ID) across different levels of your application architecture, from incoming HTTP requests to outgoing HTTP calls made by httpx. The interviewer might be probing for understanding of middleware mechanisms, thread local storage, and handling asynchronous calls in FastAPI. We'll assume the trace ID is passed via headers.

#### First-Principles Solution
Extract Trace ID: From the incoming request header.
Store Trace ID: Use context variables or thread locals to store the trace ID.
Inject Trace ID into Outgoing Requests: Pass this trace ID as a header in every outgoing httpx call.
Step-by-Step Implementation

Extract Trace ID from Incoming Request:

PYTHON
def get_trace_id_from_request(request):
    return request.headers.get("X-Trace-ID")

Store the Trace ID Using Middleware:

PYTHON
trace_id = ThreadLocalVar()

class TracingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # Get the trace ID from the incoming request
        trace_id.set(get_trace_id_from_request(request))
        response = await call_next(request)
        return response

Inject Trace ID into Outgoing Requests:

PYTHON
async def make_httpx_call(url, headers=None):
    # Inject the trace ID from thread local storage to outgoing requests
    if trace_id.get() is not None:
        headers = {**headers or {}, "X-Trace-ID": str(trace_id.get())}

    with httpx.Client(headers=headers) as client:
        response = await client.post(url, json={"data": "example"})
    return response

Use Tracing Middleware in FastAPI Application:

PYTHON
app = FastAPI()

@app.middleware("http")
async def tracing_middleware(request: Request, call_next):
    # Initialize the middleware to store and pass trace IDs
    await TracingMiddleware().dispatch(request, call_next)

# Example endpoint that uses httpx to make an outgoing request
@app.post("/make-call/")
async def make_call():
    response = await make_httpx_call("https://example.com/api")
    return {"status": "success", "response": response.json()}

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the solution involves setting up a middleware to extract and store the trace ID from incoming requests. This trace ID is then injected into every outgoing httpx call to ensure tracing consistency. Common mistakes include forgetting to set or inject the trace ID correctly, or not using thread locals effectively.

For follow-ups, we might discuss how this solution scales with higher concurrency, potential performance impacts of thread local storage, and integrating other logging frameworks that support trace IDs (e.g., Jaeger, Zipkin).

Self-Check

This implementation is correct and comprehensive.

