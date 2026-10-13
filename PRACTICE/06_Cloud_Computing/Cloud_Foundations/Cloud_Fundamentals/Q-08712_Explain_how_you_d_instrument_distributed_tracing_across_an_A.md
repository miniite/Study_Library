---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd instrument distributed tracing across an API gateway, three services, and a queue so a single request's full latency breakdown is visible.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency Investigation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To instrument distributed tracing across an API gateway, three services, and a queue, we need to ensure that each component captures relevant trace data and sends it along with requests. The goal is to get a full latency breakdown of a single request's journey through the system. Here’s how we can achieve this:

TL;DR

We'll use OpenTelemetry for tracing, which supports distributed context propagation via HTTP headers or environment variables. Each service will generate its own span and link these spans together using trace IDs and span IDs. The API gateway and queue consumer will also inject and extract the necessary context to maintain trace integrity.

#### Triaging the Question

The question is asking about implementing a distributed tracing system that captures detailed latency information across multiple components of a microservices architecture. The interviewer likely wants to understand your knowledge of:

OpenTelemetry: A comprehensive observability platform.
Span and Trace IDs: Concepts for linking related spans in a trace.
Context Propagation: Techniques to pass the necessary context between services.

#### First Principles Solution
1. Choose OpenTelemetry
Why: OpenTelemetry is a widely adopted standard that supports multiple programming languages and platforms, ensuring consistency across different parts of our system.

#### Implementation
BASH
# Install OpenTelemetry libraries for your language (e.g., Python, Go)
pip install opentelemetry-sdk
2. API Gateway Tracing
The API gateway will act as the entry point and exit point of our tracing.
When a request comes in:
PYTHON
from opentelemetry import trace

tracer = trace.get_tracer(__name__)
with tracer.start_as_current_span("APIGateway"):
    # Process incoming request
    response = process_request(request)
    # Send response
    return response
Key Points:
The API gateway starts a span when the request comes in and ends it after sending the response.
The span captures all relevant information about this transaction.
3. Service Tracing
Each service will also use OpenTelemetry to generate spans for each operation.
PYTHON
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

@app.route('/service1')
def service1():
    with tracer.start_as_current_span("Service1"):
        # Perform operations
        result = perform_service_operations()
        return result
Key Points:
Each service generates a span for its own internal operations.
Spans are linked via trace and span IDs to ensure correlation.
4. Queue Tracing
When using a queue, the consumer will inject the context into the message before sending it.
PYTHON
from opentelemetry import trace

def process_queue_message(message):
    with tracer.start_as_current_span("QueueConsumer"):
        # Extract and set the current span context
        tracer.set_span_in_context(span)
        result = handle_message(message)
        return result
Key Points:
The queue consumer extracts the context from the message header.
It continues the trace by setting it as the current span.
5. Context Propagation
Context can be propagated using HTTP headers or environment variables, depending on your choice of libraries and services.
PYTHON
# Set Span ID in Headers for HTTP requests
response.headers['X-Cloud-Tracing-Span-ID'] = span_id
Key Points:
Proper context propagation ensures that each service knows about the current trace.
6. Visualization and Analysis
Use a distributed tracing backend like Jaeger or Grafana to visualize the traces.
BASH
# Start Jaeger collector and UI
docker run -d -p 16686:16686 --name jaeger \
  jaegertracing/all-in-one:latest

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary

By instrumenting each component with OpenTelemetry and ensuring proper context propagation, we can achieve a full latency breakdown of the request. This setup allows us to trace requests across services and identify bottlenecks efficiently.

#### Common Mistakes
Forgetting Context Propagation: Always ensure that context is passed between components.
Incorrect Span Naming: Use meaningful span names for better trace readability.
Not Using OpenTelemetry: Leverage existing standards like OpenTelemetry to avoid vendor lock-in and get consistent results across services.

#### Follow-Ups
How would you handle errors in the tracing process?
What are some common performance trade-offs when implementing distributed tracing?
Can you walk through an example of a trace in Jaeger?

**Code:**

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

def service1():
    with tracer.start_as_current_span("Service1"):
        # Perform operations
        result = perform_service_operations()
        return result

@app.route('/service1')
def service1_route():
    with tracer.start_as_current_span("APIGateway"):
        response = process_request(request)
        return response

def process_queue_message(message):
    with tracer.start_as_current_span("QueueConsumer"):
        # Extract and set the current span context
        tracer.set_span_in_context(span)
        result = handle_message(message)
        return result
```

**Flow:**

1. API Gateway
2. Service 1
3. Perform Service Operations
4. Queue
5. Queue Consumer
6. Service 2
7. Service 3

