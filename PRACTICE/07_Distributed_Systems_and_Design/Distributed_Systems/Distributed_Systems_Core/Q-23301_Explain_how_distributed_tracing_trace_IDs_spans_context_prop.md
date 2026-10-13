---
type: "practice"
difficulty: "Deep"
---

# Explain how distributed tracing (trace IDs, spans, context propagation) works and how you'd use it to find that one service in a 12-hop request path is the latency culprit. What's the sampling trade-off?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Distributed tracing is a powerful tool for monitoring and debugging distributed systems, especially when dealing with complex request paths that traverse multiple services. At its core, it involves assigning trace IDs to requests and propagating contextual information across service boundaries. This allows us to correlate operations across different services and identify bottlenecks or latency issues.

To understand how it works in practice, let's break down the key concepts: trace IDs, spans, and context propagation. We'll then explore a practical scenario where we use distributed tracing to pinpoint the root cause of high latency in a 12-hop request path. Finally, we’ll discuss the trade-offs involved in sampling traces for efficiency.

#### Triaging the Question

The question is asking us to:

Explain the fundamental components and mechanisms of distributed tracing.
Apply these concepts to trace a complex multi-service request flow.
Identify the service causing latency issues.
Discuss the trade-off between capturing detailed information versus reducing overhead through sampling.

#### Assumptions
and Context
We are working with a microservices architecture where each service is independently deployable and can be scaled horizontally.
The system uses a distributed tracing framework to track requests as they propagate across services.
Each request has a unique trace ID, which is propagated using context propagation mechanisms like HTTP headers or gRPC metadata.

#### First-Principles Solution
Trace IDs

A trace ID uniquely identifies an entire request path from the user's initial interaction. This ID remains constant throughout all service invocations and helps correlate different operations across services.

Spans

Within a trace, individual service calls are represented as spans, which capture detailed information about the operation being performed. Each span includes metadata such as start time, duration, status code, and relevant key-value pairs (KVPs).

Context Propagation

To ensure that spans can be correlated across services, we use context propagation mechanisms. These mechanisms inject a unique identifier (trace ID) into headers or message payloads whenever a service call is made.

Practical Scenario

#### Cons
ider a 12-hop request path involving services A, B, C, ..., L, where the user initiates a request with trace ID 12345.

Client Request: The client sends an initial request to Service A.
Service A creates a span and generates a unique child span for each subsequent service call, passing along the parent span's context (trace ID).
Service B receives the request with trace ID 12345 and records its own span.
This process repeats until Service L processes the final response.

Suppose you have monitoring tools in place that aggregate this data and present it in a timeline view, allowing you to see every span associated with trace 12345.

Identifying Latency Culprit

To find the service causing latency issues:

Analyze Span Durations: Look for spans where the duration exceeds your predefined thresholds.
Correlation of Spans: Use the trace ID to correlate operations between different services. This helps in identifying where requests are getting stuck or delayed.

For example, you might notice that Service G's span is significantly longer than others. You can then investigate further by:

Logging detailed logs from Service G.
Profiling the code to identify potential bottlenecks (e.g., database queries, network latency).
Sampling Trade-Off

Distributed tracing introduces overhead due to context propagation and data collection. To manage this, most systems implement sampling:

Full Tracing: Capture 100% of requests but can be resource-intensive.
Sampling: Capture only a subset of traces based on certain criteria (e.g., high-latency requests). This reduces the overhead while still providing meaningful insights.

The trade-off lies in:

Precision vs. Performance: Full tracing provides more detailed data, which is crucial for debugging but can be resource-intensive.
Cost and Scalability: Sampling reduces costs by processing fewer traces, making it suitable for large-scale systems where capturing every trace would lead to high overhead.

#### Closing Summary

In summary, distributed tracing uses unique trace IDs and spans to correlate operations across services. By analyzing span durations and correlating data, you can identify latency culprits in complex request paths. Sampling is a key technique to balance the need for detailed information with system performance constraints.

#### Common Mistakes/Follow-Ups
Overlooking Context Propagation: Ensuring that trace IDs are correctly propagated through all services.
Ignoring Full Span Durations: Relying solely on high-level metrics without diving into detailed span analysis.
Not Implementing Effective Sampling Strategies: Failing to strike a balance between capturing useful data and managing overhead.

**Comparison:**

```
Span ID	Service	Operation	Duration (ms)
67890	A	User Authentication	5
11223	B	Database Query	200
...	...	...	...
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Service A] --> B[Service B];
    B --> C[Service C];
    ...
    L[Service L];
```

**Example input / output:**

```text
Input Request:
- Trace ID: 12345

Output Traces (Example):
- Span A, Service A, Duration: 5ms
- Span B, Service B, Duration: 200ms
- ...
- Span L, Service L, Duration: 10ms
```

**Data:**

```yaml
# Example Trace Data
trace_id: 12345
spans:
- span_id: 67890
  service: A
  operation: User Authentication
  duration: 5ms
- span_id: 11223
  service: B
  operation: Database Query
  duration: 200ms
```

**Flow:**

1. Client
2. Service A
3. Service B
4. Service C
5. Service D
6. Service E
7. Service F
8. H
9. Service I
10. Service J
11. Service K
12. Service L

