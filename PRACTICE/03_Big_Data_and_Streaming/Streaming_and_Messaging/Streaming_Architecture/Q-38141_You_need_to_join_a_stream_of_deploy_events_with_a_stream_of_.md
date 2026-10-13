---
type: "practice"
difficulty: "Deep"
---

# You need to join a stream of deploy events with a stream of error-rate spikes to auto-attribute incidents to deploys. Design that stream-stream join: what's the window, and what do you do about deploys with no matching spike?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a real-time streaming architecture for joining deploy events with error-rate spikes, we need to ensure that each deploy event is matched with potential error rate spikes within a relevant time window. This problem can be approached by setting up a sliding or tumbling window join based on timestamps. The goal is to attribute incidents (error rate spikes) to deploys efficiently and accurately.

#### Triage
Understanding the Question

The core of this question is about designing an efficient real-time stream processing system that joins two streams: one containing deploy events and another with error-rate spikes. The challenge lies in determining the appropriate window size for matching these events and handling scenarios where a deploy event does not find a corresponding spike within the window.

#### Assumptions
Deploy events have timestamps indicating when each deployment occurred.
Error rate spikes also come with timestamps, marking periods of high error rates.
There is an upper bound on how long after a deployment we expect to see an associated error rate spike (e.g., 5 minutes or 30 minutes).
The system must handle cases where no matching spike exists for a deploy event.

#### First-Principles Solution
Window Size Determination: We need to define the window size based on business requirements and expected latency. For instance, if we expect error rate spikes within 5 minutes of a deployment, our window should be set to 5 minutes.
Joining Mechanism: Use a sliding or tumbling window join mechanism to match deploy events with potential spikes. A sliding window allows for overlap between windows, which can help in capturing short-lived spikes that might not fully fit into the current window but will likely fall within an adjacent one.
Handling No-Match Scenarios: For deploys without a matching spike, we must decide on appropriate actions. These could include flagging the deploy as suspicious or further investigating why there was no corresponding error rate spike.
Detailed Steps

Define the Window:

If deployments can trigger spikes immediately after deployment, set the window to a small value like 5 minutes.
For more complex systems with delayed feedback, increase this to 30 minutes or even an hour.

Stream Processing Setup:

Use a stream processing framework like Apache Kafka Streams, Flink, or Spark Streaming.
Define the deploy event stream and error rate spike stream.

Join Logic Implementation:

Implement a window join between the two streams using the defined window size.
For each deploy event, look for matching error rate spikes within the sliding window.

Handling No-Matches:

If no matching spike is found, create an incident flag or log a warning.

#### Cons
ider adding additional logic to retry matches across multiple windows if necessary.

Action on Joined Events:

Once a match is identified, attribute the error rate spike to the deploy event and process it as needed (e.g., notify relevant teams).

#### Example
Workflow
Timestamp
Matching Spike Found?
No Match
Deploy Event
Window Join
Attribute Incident
Log Warning | Flag SuspiciousDeploy
Process Incident

#### Closing Summary
and Common Mistakes

In designing this stream join, it is crucial to correctly define the window size based on expected latency. Handling no-matches appropriately ensures that potential issues are not overlooked. Common mistakes include setting the window too small or large, failing to log or flag unattributed incidents, and neglecting edge cases where spikes might occur slightly after the deploy but within the defined window.

**Comparison:**

| Deploy Event         | Spike Event           |
|----------------------|-----------------------|
| ID: 123              | ID: 456               |
| Timestamp: 10:00:00Z | Timestamp: 10:03:00Z  |

**Code:**

```java
// Example Java code snippet for a sliding window join using Apache Flink

DataStream<DeployEvent> deployStream = env.addSource(new DeployEventSource());
DataStream<SpikeEvent> spikeStream = env.addSource(new SpikeEventSource());

DataStream<ProcessedIncident> joinedStream = deployStream
    .keyBy(deployEvent -> deployEvent.getTimestamp())
    .window(TumblingProcessingTimeWindows.of(Time.minutes(5)))
    .join(spikeStream.keyBy(spikeEvent -> spikeEvent.getTimestamp()))
    .where(deployEvent -> true)
    .equalTo(spikeEvent -> true)
    .process(new JoinFunction<DeployEvent, SpikeEvent, ProcessedIncident>() {
        @Override
        public ProcessedIncident process(DeployEvent deployEvent, SpikeEvent spikeEvent) throws Exception {
            // Attribute incident and return processed result
            return new ProcessedIncident(deployEvent, spikeEvent);
        }
    });
```

**Example input / output:**

```text
Input:
Deploy Event (ID: 123, Timestamp: 2023-10-01T10:00:00Z)
Spike Event (ID: 456, Timestamp: 2023-10-01T10:03:00Z, Error Rate: 0.8)

Output:
Processed Incident (Deploy ID: 123, Spike ID: 456, Error Rate: 0.8)
```

**Data:**

```json
{
  "deploy_event": {
    "id": "123",
    "timestamp": "2023-10-01T10:00:00Z"
  },
  "spike_event": {
    "id": "456",
    "timestamp": "2023-10-01T10:03:00Z",
    "error_rate": 0.8
  }
}
```

**Flow:**

1. Within Window?
2. No Match
3. Deploy Event
4. Timestamp
5. Attribute Incident
6. Log Warning|Flag SuspiciousDeploy
7. Process Incident

