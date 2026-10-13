---
type: "practice"
difficulty: "Intermediate"
---

# Design the analytics dashboard for a URL shortener that shows real-time click counts per link — how do you keep the frontend counters fresh without hammering the DB on every redirect?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient and scalable analytics dashboard for a URL shortener, we need to ensure that the frontend counters are updated in real-time while minimizing database writes. The key idea here is to implement a publish-subscribe pattern with event-driven architecture. This approach allows us to push updates from the backend to the frontend, thus reducing the load on the database.

#### Triage

The question is asking for a high-level design (HLD) that ensures real-time click counts are displayed in an analytics dashboard without frequently hitting the database. The interviewer is probing into understanding your knowledge of event-driven architectures and how to implement such patterns efficiently.

#### First-Principles Solution
Event Generation: Each time a user clicks on a shortened URL, an event is generated.
Event Storage & Processing: Use a message queue (like RabbitMQ or Kafka) to store these events temporarily.
Real-Time Aggregation: Use a stream processing engine like Apache Flink or Amazon Kinesis Data Streams to process these events in real-time and update the analytics dashboard.
Dashboard Update: The processed data is then sent to the frontend via web sockets or server-sent events (SSE) to keep counters fresh.
Detailed Steps
Event Source: When a user clicks on a shortened URL, an event ClickEvent is generated with metadata such as timestamp, link_id, and possibly user_id.
Message Queue: The ClickEvent is sent to a message queue.
Stream Processing: A stream processing application reads from the message queue, aggregates clicks in real-time, and stores aggregated data in a database or in-memory store (like Redis).
Real-Time Dashboard Update:
Database Updates: Aggregate data can be periodically written to a durable storage for historical analysis.
Frontend Updates: Real-time updates are pushed to the frontend using web sockets or SSE.

#### Closing Summary

By implementing this publish-subscribe pattern, we ensure that the analytics dashboard is updated in real-time without hammering the database on every redirect. This approach not only reduces the load on the database but also provides a scalable solution for handling high traffic and maintaining real-time data consistency.

#### Common Mistakes/Follow-Ups
Ignoring Message Queues: Failing to use message queues can lead to increased database writes, making the system less scalable.
Overlooking Real-Time Updates: Not using web sockets or SSE means that the frontend counters might not be as responsive and may lag behind real-time updates.
Triaging Follow-Ups

The interviewer might ask about specific technologies, trade-offs, or how to handle edge cases such as failed events or high traffic. Be prepared to discuss these aspects in detail.

**Code:**

```
Example of generating a ClickEvent and pushing it to a queue (pseudo-code)

import rabbitmq

def on_url_click(link_id): event = ClickEvent(link_id=link_id, timestamp=datetime.now()) rabbitmq.push_event(event)

Stream processing application in Flink

from flink import StreamExecutionEnvironment, KeyedProcessFunction

env = StreamExecutionEnvironment.get_execution_environment() clicks_stream = env.add_source(lambda: click_events_queue) aggregated_clicks = clicks_stream.key_by("link_id").process(AggregationFunction())

def AggregationFunction(): state = StateDescriptor("count", Types.LONG())

CODE
Copy
def process(value, ctx):
    count = ctx.get_state(state)
    new_count = count + 1
    ctx.update_state(new_count)

return KeyedProcessFunction()
CODE
Copy

```javascript
// Example of pushing events to a WebSocket server (pseudo-code)
const socket = new WebSocket("ws://localhost:8080/dashboard");

socket.onmessage = function(event) {
    const data = JSON.parse(event.data);
    updateCounter(data.link_id, data.count);
};

function onUrlClick(linkId) {
    fetch(`/click`, { method: 'POST', body: JSON.stringify({ link_id: linkId }) });
}
```

