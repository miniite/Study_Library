---
type: "practice"
difficulty: "Deep"
---

# How do you design a REST API to return partial results progressively for a slow query, choosing between chunked JSON, NDJSON streaming, and SSE, and how does the client handle a mid-stream error?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Progressive Results](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST API that returns partial results progressively from a slow query, we need to consider several factors such as performance, scalability, reliability, and the specific requirements of our application. In this scenario, three main approaches can be considered: chunked JSON, NDJSON streaming, and Server-Sent Events (SSE). The choice among these depends on the nature of the data, network conditions, and client-side handling capabilities.

#### Triage

The core question is to design a REST API that efficiently handles slow queries by progressively returning partial results. This involves choosing between different streaming methods and ensuring proper client-side error handling. We need to assume that our backend has a query that takes a long time to execute due to complex joins, large datasets, or network latency.

#### First-Principles Solution
1. Chunked JSON
Description: This approach involves breaking down the response into smaller chunks and sending them incrementally.

#### Implementation
Set the Transfer-Encoding: chunked header in the HTTP response. Each chunk can be a portion of the JSON data.

#### Pros
Simple to implement, works well with existing REST clients.

#### Cons
May introduce additional overhead due to chunking and may not be efficient for very large datasets.
2. NDJSON Streaming
Description: NDJSON (Newline Delimited JSON) involves sending each record as a separate line in the response body, with each line being valid JSON.

#### Implementation
Use Content-Type: application/x-ndjson and send each chunk of JSON data separated by newlines.

#### Pros
Flexible and easy to process on the client side. Each partial result can be consumed independently.

#### Cons
More complex setup compared to chunked JSON.
3. Server-Sent Events (SSE)
Description: SSE allows the server to push data to the client in real-time, making it a good choice for progressive updates.

#### Implementation
Use Content-Type: text/event-stream and send events with a timestamp, allowing clients to handle each event independently.

#### Pros
Efficient for large datasets and real-time updates. Clients can resume where they left off if the connection is lost.

#### Cons
Limited browser support compared to other methods.
Choosing the Right Approach
Slow Queries: For slow queries that produce a large amount of data, SSE might be the best choice due to its real-time push capabilities and resilience against network errors.
Client-Side Complexity: If the client needs to process partial results independently and can handle complex stream handling, NDJSON could be preferred.

#### Performance Considerations
Chunked JSON is simpler but may introduce more latency compared to SSE or NDJSON.
Client-Side Handling

The client must handle mid-stream errors gracefully. This involves:

Error Detection: The client should monitor the HTTP status codes and network conditions.
Recovery Mechanisms: Implement retries with exponential backoff in case of transient errors.
State Management: Maintain state to resume processing from where it left off if a connection is lost.

#### Example
Implementation

Let's implement SSE for our use case:

PYTHON
from flask import Flask, Response

app = Flask(__name__)

@app.route('/slow-query')
def slow_query():
    def generate():
        # Simulate a slow query with sleep and send events
        for i in range(10):
            event_data = f"data: {i}\n\n"
            yield event_data
            import time
            time.sleep(1)  # simulate delay

    return Response(generate(), mimetype='text/event-stream')

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, choosing between chunked JSON, NDJSON streaming, and SSE depends on the specific requirements of your application. SSE is generally more suitable for real-time updates, while NDJSON provides flexibility in processing partial results independently. The client must handle mid-stream errors gracefully to ensure a smooth user experience.

#### Common mistakes
include:

Overlooking network error handling.
Misunderstanding the differences between these streaming methods.
Not considering state management for clients that need to resume processing from where they left off.

#### Follow-ups
could involve discussing how to optimize database queries, implementing caching strategies, or exploring other asynchronous communication patterns like WebSockets.

