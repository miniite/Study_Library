---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd join a stream of alerts with a stream of deploy events to attribute incidents to recent changes.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multi-granularity Windows](<../../../../TOOLS/Apache_Flink/Flink_Core/Multi-granularity_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To join a stream of alerts with a stream of deploy events in Apache Flink, we aim to attribute each alert to its most recent preceding or coinciding deployment event, thereby understanding if a recently deployed change is causing an incident. This process involves several key steps and considerations.

#### Triage

The question requires us to implement a join operation between two streams: one for alerts (events indicating potential issues) and another for deploy events (changes to the system). The goal is to attribute each alert with information about the most recent deployment that could be related. Flink provides stream processing capabilities, making this task feasible through windowing, state management, and event-time processing.

#### Assumptions
Both streams are timestamped, allowing us to use event time semantics.
The alert stream contains minimal metadata (e.g., event ID, severity).
The deploy event stream includes information like deployment ID, version number, and timestamp.
We need a mechanism to handle late events in the deploy stream.

#### Solution
Define Event Time Processing: Use Flink’s built-in support for event time processing to order events by their timestamps accurately.
Create DataStreams for Both Streams: Convert the alert and deploy streams into Flink DataStream objects if they are not already.
Windowing Strategy: Implement a tumbling window or session window depending on the frequency of alerts and deployments. A tumbling window might be appropriate here, partitioning data into fixed-size intervals.
State Management: Use stateful processing to maintain information about recent deploy events as we process alert streams.
Event-Time Join: Perform an event-time join between the two streams based on a common key (e.g., deployment ID) and timestamp.
Step-by-Step Solution

Define DataStreams:

JAVA
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<AlertEvent> alertStream = env.addSource(new AlertSourceFunction());
DataStream<DeployEvent> deployStream = env.addSource(new DeployEventSourceFunction());

Windowing:

JAVA
int windowSizeMinutes = 5; // Define the window size based on your requirements.

SingleOutputStreamOperator<AlertEvent> windowedAlerts = alertStream.keyBy(AlertEvent::getDeploymentId)
                                                             .window(TumblingProcessingTimeWindows.of(Time.minutes(windowSizeMinutes)));

State Management:

JAVA
MapStateDescriptor<String, DeployEvent> deployEventsWindowState = new MapStateDescriptor<>("deployEventsWindow", String.class, DeployEvent.class);

SingleOutputStreamOperator<AlertWithDeployEvent> joinedStream = windowedAlerts.join(deployStream)
                                                                              .where(AlertEvent::getDeploymentId)
                                                                              .equalTo(DeployEvent::getId)
                                                                              .windowed(TumblingProcessingTimeWindows.of(Time.minutes(windowSizeMinutes)))
                                                                              .apply(new JoinFunction<AlertEvent, DeployEvent, AlertWithDeployEvent>() {
                                                                                  @Override
                                                                                  public AlertWithDeployEvent join(AlertEvent alert, DeployEvent deploy) throws Exception {
                                                                                      // Combine the two events and return a new object with relevant details.
                                                                                      return new AlertWithDeployEvent(alert.getEventId(), deploy.getVersionNumber());
                                                                                  }
                                                                              })
                                                                              .map(new KeyedStateFunction<AlertWithDeployEvent>() {
                                                                                      private MapState<String, DeployEvent> deployEventsWindowState;

                                                                                      @Override
                                                                                      public void open(Configuration parameters) throws Exception {
                                                                                          super.open(parameters);
                                                                                          this.deployEventsWindowState = getRuntimeContext().getMapState(deployEventsWindowState);
                                                                                      }

                                                                                      @Override
                                                                                      public AlertWithDeployEvent map(AlertWithDeployEvent alertWithDeployEvent) throws Exception {
                                                                                          // Update state with the current deploy event.
                                                                                          DeployEvent currentDeploy = alertWithDeployEvent.getDeploy();
                                                                                          this.deployEventsWindowState.put(currentDeploy.getId(), currentDeploy);

                                                                                          return alertWithDeployEvent;
                                                                                      }
                                                                                  });

Output:

JAVA
joinedStream.print(); // Print the joined stream for inspection.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we use Flink’s event time processing to align alert events with their most recent deploy events via a windowed join operation. Key considerations include choosing appropriate window sizes and handling stateful processing efficiently.

#### Common mistakes
might include:

Incorrectly setting up the window size leading to missed or duplicated joins.
Failing to properly manage state, which can lead to data inconsistencies.
Not considering late events in the deploy stream, causing delayed or incorrect attribute assignments.

For follow-ups, one could explore techniques like session windows for more complex event patterns or discuss strategies for handling different types of alerts and their correlation with deployments. Additionally, scaling this solution across a distributed cluster might be necessary depending on the volume of data.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Joining Alert and Deploy Streams in Flink
    dateFormat  YYYY-MM-DD
    section Setup
        alertStream : 2023-10-15
        deployStream : 2023-10-15
    section Windowing
        windowedAlerts : 2023-10-15T10:00:00Z - 2023-10-15T10:01:00Z
    section State Management
        deployEventsWindowState : 2023-10-15T10:00:00Z
    section Join Operation
        joinedStream : 2023-10-15T10:00:00Z - 2023-10-15T10:01:00Z
```

**Code:**

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<AlertEvent> alertStream = env.addSource(new AlertSourceFunction());
DataStream<DeployEvent> deployStream = env.addSource(new DeployEventSourceFunction());

int windowSizeMinutes = 5;
SingleOutputStreamOperator<AlertEvent> windowedAlerts = alertStream.keyBy(AlertEvent::getDeploymentId)
                                                                .window(TumblingProcessingTimeWindows.of(Time.minutes(windowSizeMinutes)));

MapStateDescriptor<String, DeployEvent> deployEventsWindowState = new MapStateDescriptor<>("deployEventsWindow", String.class, DeployEvent.class);

SingleOutputStreamOperator<AlertWithDeployEvent> joinedStream = windowedAlerts.join(deployStream)
                                                                                 .where(AlertEvent::getDeploymentId)
                                                                                 .equalTo(DeployEvent::getId)
                                                                                 .windowed(TumblingProcessingTimeWindows.of(Time.minutes(windowSizeMinutes)))
                                                                                 .apply(new JoinFunction<AlertEvent, DeployEvent, AlertWithDeployEvent>() {
                                                                                     @Override
                                                                                     public AlertWithDeployEvent join(AlertEvent alert, DeployEvent deploy) throws Exception {
                                                                                         return new AlertWithDeployEvent(alert.getEventId(), deploy.getVersionNumber());
                                                                                     }
                                                                                 })
                                                                                 .map(new KeyedStateFunction<AlertWithDeployEvent>() {
                                                                                         private MapState<String, DeployEvent> deployEventsWindowState;

                                                                                         @Override
                                                                                         public void open(Configuration parameters) throws Exception {
                                                                                             super.open(parameters);
                                                                                             this.deployEventsWindowState = getRuntimeContext().getMapState(deployEventsWindowState);
                                                                                         }

                                                                                         @Override
                                                                                         public AlertWithDeployEvent map(AlertWithDeployEvent alertWithDeployEvent) throws Exception {
                                                                                             DeployEvent currentDeploy = alertWithDeployEvent.getDeploy();
                                                                                             this.deployEventsWindowState.put(currentDeploy.getId(), currentDeploy);

                                                                                             return alertWithDeployEvent;
                                                                                         }
                                                                                     });

joinedStream.print(); // Print the joined stream for inspection.
```

**Example input / output:**

| Input: Alert Event | Input: Deploy Event |
|--------------------|---------------------|
| {"eventId": "A01", "deploymentId": "D01", "timestamp": "2023-10-15T10:00:00Z"} | {"id": "D01", "versionNumber": "v1.0.0", "timestamp": "2023-10-15T09:00:00Z"} |
| Output: Joined Event |
|----------------------|
| {"eventId": "A01", "deploymentId": "D01", "versionNumber": "v1.0.0"} |

**Data:**

```json
{
  "AlertEvent": {
    "eventId": "A01",
    "deploymentId": "D01",
    "timestamp": "2023-10-15T10:00:00Z"
  },
  "DeployEvent": {
    "id": "D01",
    "versionNumber": "v1.0.0",
    "timestamp": "2023-10-15T09:00:00Z"
  }
}
```

**Flow:**

1. Define DataStreams
2. Create Windowed Alert Stream
3. Perform Join Operation
4. Manage State Efficiently
5. Output Joined Events

