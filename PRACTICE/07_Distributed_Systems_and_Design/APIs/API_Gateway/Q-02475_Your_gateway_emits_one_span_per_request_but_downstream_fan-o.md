---
type: "practice"
difficulty: "Deep"
---

# Your gateway emits one span per request but downstream fan-out spans aren't linked because trace context is dropped on async retries; how do you ensure trace propagation survives retries, queuing, and thread hops?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Timeouts Retries and Backoff](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Timeouts_Retries_and_Backoff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that trace propagation survives retries, queuing, and thread hops in an API Gateway setup with downstream fan-out spans, we need to focus on maintaining a consistent context across these operations. The key challenge is ensuring that the trace ID and related span context are preserved as the request flows through different components of the system.

#### Triaging the Question

The question primarily probes our understanding of:

Trace Propagation Mechanisms: How does trace propagation work in distributed systems?
Concurrency and Asynchronous Processing: What challenges arise with retries, queuing, and thread hops?
Context Preservation: How can we ensure that context is correctly carried over during these operations?

#### Assumptions
We are using a tracing system such as OpenTracing or OpenTelemetry.
The API Gateway emits one span per request.
Downstream fan-out spans do not inherit the trace context from the upstream request.

#### First-Principles Solution

Context Propagation Headers:

When an HTTP request arrives at the API Gateway, extract and store the trace context (trace ID, span ID, parent span ID) in headers.
For each downstream call, include these headers to propagate the context.

Handling Retries and Queues:

Implement a retry mechanism that re-includes the same headers with the request on subsequent attempts.
Ensure queuing systems properly handle context headers and pass them along with the request.

Thread Hops and Concurrency:

In scenarios where requests may be processed by different threads, ensure thread-local storage is used to preserve context.
Use middleware or custom logic to inject and extract context from headers in each thread.
Detailed Steps

Extract Context at Gateway:

PYTHON
def process_request(request):
    # Extract trace context from headers
    trace_id = request.headers.get('X-Trace-ID')
    span_id = request.headers.get('X-Span-ID')
    parent_span_id = request.headers.get('X-Parent-Span-ID')

    # Store in thread-local storage or custom context object
    Context.set_trace_context(trace_id, span_id, parent_span_id)

Inject Context for Downstream Calls:

PYTHON
def make_downstream_call(url):
    headers = {
        'X-Trace-ID': Context.get_trace_context().trace_id,
        'X-Span-ID': str(uuid.uuid4()),
        'X-Parent-Span-ID': Context.get_trace_context().parent_span_id
    }
    # Make the downstream call with headers

Handle Retries:

PYTHON
def retry_request(request, max_retries=5):
    for i in range(max_retries):
        try:
            response = make_downstream_call(url)
            return response
        except Exception as e:
            if i < max_retries - 1:
                # Log and re-queue the request with same headers
                pass
            else:
                raise e

Thread Hops:

PYTHON
class CustomHandler(BaseHTTPRequestHandler):
    def log_message(self, format, *args):
        # Ensure thread-safe logging
        threading.current_thread().local.trace_context = Context.get_trace_context()

    def do_GET(self):
        with Context.use_trace_context():
            process_request(self)
            self.send_response(200)

#### Closing Summary

By implementing these steps, we can ensure that trace context is preserved across retries, queuing, and thread hops. This involves managing headers effectively, leveraging thread-local storage where necessary, and ensuring context is correctly passed between components.

#### Common Mistakes/Follow-Ups
Overlooking Thread-Safe Context Propagation: Ensure that context is properly managed in multithreaded environments.
Header Overhead: Be cautious about the overhead of adding headers to requests; keep them minimal but sufficient for trace propagation.
Handling Queues and Retries: Ensure queuing systems and retry mechanisms correctly handle and pass along context.

