---
type: "practice"
difficulty: "Deep"
---

# A microservice is causing a cascading slowdown but logs are inconclusive. Explain how distributed tracing (spans, trace context propagation) pinpoints it, and what you'd instrument to prove it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with microservices that exhibit unexpected behavior like cascading slowdowns, traditional logging can often be insufficient in pinpointing the root cause. Distributed tracing, through the use of spans and trace context propagation, provides a mechanism to track requests across multiple services, helping us identify performance bottlenecks and correlate them effectively.

#### Triaging the Question

The core of this question is about understanding how distributed tracing works and what specific instrumentation would be necessary to diagnose issues in microservices. The interviewer is probing your knowledge of:

Distributed Tracing Concepts: Understanding spans, trace context propagation, and their benefits.
Instrumentation Techniques: Knowing which metrics and logs should be collected to provide actionable insights.

#### First-Principles Solution
Step 1: Understand the Basics

A microservice system involves multiple services communicating with each other. When a request is made to one service, it might make several calls to downstream services. Traditional logging captures data at the service level but loses context across these boundaries. This is where distributed tracing comes in.

Step 2: Define Spans and Trace Context
Spans: Represent individual operations within a trace. Each span records information such as start time, end time, duration, name, and metadata.
Trace Context Propagation: Ensures that every service in the request path includes the trace ID and spans IDs, allowing downstream services to see this context.
Step 3: Instrumentation Strategy

To diagnose a cascading slowdown, you would need to instrument key services with distributed tracing. Here’s what needs to be done:

Set Up Tracing Frameworks: Use frameworks like Jaeger, Zipkin, or OpenTelemetry that support distributed tracing.
Capture Span Data: For every request, create a root span and sub-spans for each service it interacts with.
Correlate Requests Across Services: Ensure all services propagate the trace context so spans can be linked together.
Step 4: Collect Metrics and Logs
Duration Metrics: Track how long requests take within each span to identify bottlenecks.
Error Rates: Monitor error rates in sub-spans to detect issues early.
Service Latencies: Measure latencies between services to understand performance degradation points.

#### Closing Summary

By leveraging distributed tracing, you can effectively track and correlate the flow of a request across multiple microservices. This approach provides visibility into the overall behavior of your system, making it easier to pinpoint where delays are occurring and how they propagate through various components.

#### Common Mistakes/Follow-Ups
Overlooking Context Propagation: Failing to ensure that trace context is properly propagated can lead to gaps in tracing.
Insufficient Metrics Collection: Not collecting enough metrics (e.g., response times, error rates) can hinder the ability to diagnose issues accurately.
Ignoring Distributed Nature: Treating microservices as standalone components without considering their interactions can mislead analysis.
Triaging Follow-Up Questions:
How do you handle service failures in distributed tracing?
What are some challenges and solutions when scaling distributed tracing systems?

**Comparison:**

```
Span ID	Parent Span ID	Name	Start Time	Duration (ms)
1234	0987	Request-Root	2023-09-15T12:00:00Z	10000
0987	1234	Service-1	2023-09-15T12:00:05Z	5000
1234	0987	Service-2	2023-09-15T12:00:06Z	2000
8765	1234	Service-3	2023-09-15T12:00:06Z	1000
7654	1234	Service-4	2023-09-15T12:00:07Z	1000
6543	7654	Service-4-subtask	2023-09-15T12:00:08Z	500
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Root Service] --> B[Service 1];
    B --> C[Service 2];
    C --> D[Service 3];
    C --> E[Service 4];
```

**Example input / output:**

```text
Request to Root Service:
- Request starts at 2023-09-15T12:00:00Z

Trace Output Example:
- Span A (Root) -> B (Service 1)
  - Duration of A = 10s
  - Error rate in B = 1%
- Span B (Service 1) -> C (Service 2) and D (Service 3)
  - Duration of B = 5s, C = 2s, D = 1s
- Span C (Service 2) -> E (Service 4)
  - Duration of C = 2s, E = 1s
```

**Data:**

```yaml
trace_id: 1234567890abcdef1234567890abcdef
span_id: 1234
parent_span_id: 0987
name: Request-Root
start_time: 2023-09-15T12:00:00Z
duration: 10s

trace_id: 1234567890abcdef1234567890abcdef
span_id: 0987
parent_span_id: 1234
name: Service-1
start_time: 2023-09-15T12:00:05Z
duration: 5s

trace_id: 1234567890abcdef1234567890abcdef
span_id: 1234
parent_span_id: 0987
name: Service-2
start_time: 2023-09-15T12:00:05Z
duration: 2s

trace_id: 1234567890abcdef1234567890abcdef
span_id: 8765
parent_span_id: 1234
name: Service-3
start_time: 2023-09-15T12:00:06Z
duration: 1s

trace_id: 1234567890abcdef1234567890abcdef
span_id: 7654
parent_span_id: 1234
name: Service-4
start_time: 2023-09-15T12:00:07Z
duration: 1s

trace_id: 1234567890abcdef1234567890abcdef
span_id: 6543
parent_span_id: 7654
name: Service-4-subtask
start_time: 2023-09-15T12:00:08Z
duration: 0.5s
```

**Flow:**

1. Root Service
2. Service 1
3. Service 2
4. Service 3
5. Service 4

