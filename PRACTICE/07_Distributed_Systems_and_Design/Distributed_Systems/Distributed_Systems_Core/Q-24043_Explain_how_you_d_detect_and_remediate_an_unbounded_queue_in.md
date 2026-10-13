---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd detect and remediate an unbounded queue in the telemetry pipeline that quietly absorbed a slow consumer for hours before filling and failing all at once. What age-based and depth-based alerts surface this gradually, and why is an unbounded shock-absorber queue an anti-pattern here?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Metastable Failures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Metastable_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting and remediating an unbounded queue in a telemetry pipeline, we need to understand how slow consumers can silently absorb data over time until it reaches its capacity and fails suddenly. The key is to implement effective monitoring and alerting mechanisms that can catch this gradual buildup before it causes a catastrophic failure.

Triaging the Problem

The problem revolves around two main aspects:

Detection: How do we detect an unbounded queue where a slow consumer has been quietly absorbing data for hours?
Remediation: Once detected, how do we prevent or mitigate such failures?

#### Assumptions
and Key Concepts
The system uses a distributed architecture with multiple producers sending telemetry data to the pipeline.
There are one or more consumers processing this data at varying speeds.
Slow consumer: A consumer that processes data slower than it is being produced.

#### First-Principles Solution

Age-Based Alerts:

Monitor the age of messages in the queue using timestamps. This helps us understand how long messages have been sitting unprocessed.
Set up alerts based on message age thresholds. For example, if a message has not been processed within 30 minutes to an hour, trigger a warning.

Depth-Based Alerts:

Monitor the depth of the queue (number of pending messages). This helps us understand how much data is being accumulated.
Set up alerts based on queue depth thresholds. For example, if the queue depth exceeds 1 million messages, trigger an alert.

Queue Size and Consumer Metrics:

Collect metrics from consumers to understand their processing speed and efficiency.
Monitor CPU and memory usage of consumer nodes to identify bottlenecks or issues that might be causing slow processing.

Shock-Absorber Queue Anti-Pattern:

Avoid using a single large queue as a shock absorber, which can lead to unpredictable failures when the buffer fills up.
Instead, use smaller queues for each consumer and aggregate metrics from these smaller queues.
Detailed Implementation
Step-by-Step Solution

Set Up Metrics Collection:

Collect timestamps of message insertions and processing completion times.
Use Prometheus or similar monitoring tools to collect and visualize this data.

Implement Age-Based Alerts:

PYTHON
import time

class MessageProcessor:
    def __init__(self, queue):
        self.queue = queue
        self.last_processed_time = {}

    def process(self):
        while not self.queue.empty():
            msg = self.queue.get()
            current_time = int(time.time())
            last_processing_time = self.last_processed_time.get(msg.id)
            if last_processing_time and current_time - last_processing_time > 1800:  # 30 minutes
                send_alert(f"Message {msg.id} has been unprocessed for more than 30 minutes.")
            self.queue.task_done()
            self.last_processed_time[msg.id] = current_time

Implement Depth-Based Alerts:

PYTHON
def queue_depth_alert(queue, threshold):
    if queue.qsize() > threshold:
        send_alert(f"Queue depth {queue.qsize()} exceeded threshold of {threshold}.")

Aggregate Consumer Metrics:

Monitor CPU and memory usage using tools like Prometheus.
Set up alerts based on resource utilization thresholds.

Avoid Shock-Absorber Queue Anti-Pattern:

PYTHON
class TelemetryPipeline:
    def __init__(self, producers, consumers):
        self.producers = producers
        self.consumers = [Queue() for _ in range(len(consumers))]

    def process_data(self):
        for producer in self.producers:
            while not producer.empty():
                msg = producer.get()
                for i, consumer in enumerate(self.consumers):
                    if consumer.qsize() < 10000:  # Threshold to prevent overfilling

#### cons
umer.put(msg)
                        break
    ```

#### Closing Summary
and Common Mistakes

#### Common Mistakes

Failing to set up proper alerts for both age and depth of the queue.
Using a single large buffer as a shock absorber without monitoring its health.
Ignoring slow consumer metrics, leading to undetected processing bottlenecks.

#### Follow-Up
Questions:

How would you integrate these monitoring solutions with existing alerting systems?
What measures would you take if the queue depth exceeds the threshold despite having multiple consumers?

**Comparison:**

| Metric | Threshold | Alert |
|--------|-----------|-------|
| Queue Depth | >50000 messages | Triggered when queue depth exceeds threshold. |
| Message Age | >30 minutes | Triggered when a message has not been processed within 30 minutes. |
| CPU Usage | >80% | Triggered if consumer node's CPU usage exceeds 80%. |
| Memory Usage | >90% | Triggered if consumer node's memory usage exceeds 90%. |

**Code:**

```python
import time

class MessageProcessor:
    def __init__(self, queue):
        self.queue = queue
        self.last_processed_time = {}

    def process(self):
        while not self.queue.empty():
            msg = self.queue.get()
            current_time = int(time.time())
            last_processing_time = self.last_processed_time.get(msg.id)
            if last_processing_time and current_time - last_processing_time > 1800:  # 30 minutes
                send_alert(f"Message {msg.id} has been unprocessed for more than 30 minutes.")
            self.queue.task_done()
            self.last_processed_time[msg.id] = current_time

def queue_depth_alert(queue, threshold):
    if queue.qsize() > threshold:
        send_alert(f"Queue depth {queue.qsize()} exceeded threshold of {threshold}.")

class TelemetryPipeline:
    def __init__(self, producers, consumers):
        self.producers = producers
        self.consumers = [Queue() for _ in range(len(consumers))]

    def process_data(self):
        for producer in self.producers:
            while not producer.empty():
                msg = producer.get()
                for i, consumer in enumerate(self.consumers):
                    if consumer.qsize() < 10000:  # Threshold to prevent overfilling
                        consumer.put(msg)
                        break

def send_alert(message):
    print(message)
```

**Example input / output:**

```text
Input:
- Producer sends a message at time = 12:00 PM.
- Consumer takes 2 hours to process the message.

Output:
- At 2:00 PM, an age-based alert is triggered because the message has not been processed within 30 minutes.
- At 4:00 PM, another age-based alert is triggered if processing still hasn't occurred.
```

**Data:**

```yaml
queue_depth_threshold: 50000
processing_timeout: 3600
consumer_metrics:
  cpu_usage_threshold: 80%
  memory_usage_threshold: 90%
```

**Flow:**

1. Message Produced
2. Is Queue Full?
3. Add Message to Queue
4. Queue Depth Alert
5. Is Consumer Busy?
6. Add Message to Queue
7. Consumer CPU/Memory UsageAlert
8. Queue Depth ExceedsThreshold, Remediate
9. CPU/Memory Usage ExceedsThreshold, Remediate

