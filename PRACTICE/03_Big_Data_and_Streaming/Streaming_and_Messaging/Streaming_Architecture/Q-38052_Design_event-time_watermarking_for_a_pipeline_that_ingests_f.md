---
type: "practice"
difficulty: "Expert"
---

# Design event-time watermarking for a pipeline that ingests from mobile clients that can be offline for days. How do you balance allowed-lateness against unbounded state, and what product compromises do you negotiate?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Designing event-time watermarking for a real-time streaming pipeline that handles offline mobile clients requires balancing allowed-lateness with managing unbounded state. We'll address this by implementing a watermark strategy that allows late events but minimizes the impact on state management and processing latency, while also considering operational trade-offs.

#### Triage
The core challenge here is ensuring our system can handle delayed or missed data from mobile clients without incurring excessive memory usage or compromising real-time performance. We need to design a solution that can gracefully manage late events within an acceptable window of allowed lateness. This involves understanding the latency and state requirements, negotiating with stakeholders on the trade-offs, and implementing a robust watermark mechanism.

#### First-Principles Solution

Understand Allowed-Lateness Requirements: Determine the maximum amount of time that late events can be processed without impacting critical business outcomes. This could range from hours to days, depending on the use case.

Watermark Strategy Design: Implement a watermark mechanism that allows late events but ensures they are processed within the allowed lateness window. Key considerations include:

Watermark Delay Policy: Define how often watermarks are generated and when they should be applied in processing pipelines.
State Management: Employ efficient state management techniques such as event-time windows, session windows, or tumbling windows to handle late events without bloating the state.

Event Handling Strategy:

Implement a back-pressure mechanism to manage the rate of data ingestion and ensure that the system can gracefully handle spikes in incoming data.
Use buffering mechanisms to temporarily store late events until they are processed within the allowed window or discarded if they exceed it.

State Compaction: Regularly compact state to reduce memory footprint, especially for scenarios where late events significantly increase state size. This involves periodically aggregating and summarizing state elements that do not need real-time processing.

Operational Trade-Offs:

Negotiate with stakeholders on the allowed lateness window based on business requirements.

#### Cons
ider the impact of increased memory usage against reduced operational complexity and improved real-time performance.
Evaluate trade-offs in terms of additional infrastructure costs for state compaction or event handling.

Monitoring & Logging: Implement comprehensive monitoring and logging to track watermark generation, late events processing, and state management. This helps in identifying bottlenecks and optimizing the system over time.

#### Closing Summary
By carefully designing a watermark strategy that balances allowed-lateness with efficient state management, we can ensure robust handling of delayed or missed data from mobile clients without sacrificing real-time performance or operational complexity. Key factors include defining appropriate lateness windows, employing smart state management techniques, and negotiating with stakeholders to find the best compromise.

#### Common Mistakes/Follow-Ups

Overlooking the need for a watermark mechanism that can handle late events.
Ignoring state management implications when processing large volumes of late data.
Failing to negotiate with stakeholders on allowed lateness windows, leading to suboptimal performance or excessive resource utilization.
Not implementing adequate monitoring and logging mechanisms to track system behavior.

**Comparison:**

```
Client ID	Latest Login Time	Latest Logout Time
12345	2023-10-01T08:00Z	2023-10-01T10:00Z
```

**Code:**

```python
# Example Python code for watermark strategy implementation
from apache_beam.options.pipeline_options import PipelineOptions
from apache_beam.io.gcp.pubsub import ReadFromPubSub

pipeline = beam.Pipeline(options=PipelineOptions())

# Define the watermark delay policy
def watermark_delay_policy(window, trigger):
    return trigger.after_processing_time(12 * 3600)  # Allow lateness of up to 12 hours

# Implement state management and back-pressure mechanisms
class EventProcessor:
    def process(self, element):
        # Process event logic here
        pass

    def on_timer(self, timestamp, accumulated_value):
        # Handle late events processing
        pass

# Create a watermark strategy
watermark_strategy = beam.window.FixedWindows(3600).with_user_window_strategy(watermark_delay_policy)

# Define the pipeline with state management and event handling logic
read_from_pubsub = ReadFromPubSub(topic='your-topic')
pipeline | 'Read from PubSub' >> read_from_pubsub | 'Process Events' >> beam.ParDo(EventProcessor()) | 'Apply Watermarks' >> watermark_strategy

result = pipeline.run()
```

**Example input / output:**

```

Input: A stream of user events with potential late events due to mobile client offline periods.

Output: Processed and aggregated data without significant state bloat or performance degradation.

PLAINTEXT
Copy
Timestamp        Event Type      Client ID    State Update
2023-10-01T08:00  Login           12345       User logged in (State: Logged In)
2023-10-01T10:00  Logout          12345       User logged out (State: Logged Out, late event handled within allowed window)
```

**Data:**

```json
{
  "event": {
    "timestamp": "2023-10-01T08:00:00Z",
    "client_id": "12345",
    "data": "user login"
  },
  "late_event": {
    "timestamp": "2023-10-01T10:00:00Z",  // Late by 2 hours
    "client_id": "12345",
    "data": "user logout"
  }
}
```

**Flow:**

1. Yes
2. No
3. Event Ingestion
4. Watermark Generation
5. Late Event?
6. Process Late Event
7. Normal Processing
8. State Update
9. State Update

