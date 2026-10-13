---
type: "practice"
difficulty: "Deep"
---

# How do you set the platform's stance on stateful versus stateless AIOps computations: which detections must hold state in the stream (baselines, sessions) versus be recomputed statelessly? What does pushing statefulness into the platform cost you operationally?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Processing Time vs Event Time](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Processing_Time_vs_Event_Time.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In designing a real-time streaming architecture for AIOps computations, the choice between stateful and stateless processing is crucial. Stateful operations retain internal states across events, which can be essential for certain types of anomaly detection or session-based analytics. Stateless operations, on the other hand, do not maintain any persistent state, making them simpler to implement but potentially less efficient in handling complex scenarios.

Triaging:

The question asks how to decide when to use stateful versus stateless computations in AIOps processing pipelines. This involves understanding the trade-offs between these approaches and weighing operational costs against analytical benefits.

#### Assumptions
Event Frequency: High-frequency events may require more robust state management.
Computational Resources: Statefulness can be resource-intensive, especially with large datasets or complex computations.
Latency Requirements: Real-time processing often demands low latency, which might necessitate stateless operations.

#### First-Principles Solution

To determine the appropriate stance on stateful versus stateless AIOps computations:

Identify Stateful Requirements:

Anomaly Detection: Baselines and thresholds require historical data for comparison.
Session-Based Analytics: Tracking user sessions or device states over time necessitates maintaining session information.

Determine Stateless Suitability:

Real-Time Alerts: Simple threshold-based alerts can be computed statelessly.
Low-Latency Aggregations: Summarizing metrics like average CPU usage in the last minute is less dependent on historical data.

Cost Analysis:

Storage Costs: Maintaining states requires additional storage, increasing operational costs.
Processing Overhead: Stateful computations can be more computationally intensive and may require more resources.
Latency Implications: Statefulness can introduce latency due to the need for state updates.

Operational Considerations:

Scalability: Stateless operations are generally easier to scale horizontally without additional state management complexity.
Fault Tolerance: Stateful systems may require more sophisticated fault tolerance mechanisms, such as distributed consensus algorithms.
Maintenance Complexity: Managing states can add layers of complexity in terms of data consistency and recovery.

#### Closing Summary

By carefully evaluating the requirements for each computation type—whether they are state-dependent or independent—we can optimize our AIOps platform. Stateful operations provide richer insights but come with higher costs, while stateless operations offer simplicity and efficiency at the expense of potentially less detailed analysis.

#### Common Mistakes/Follow-Ups
Ignoring Latency: Failing to consider real-time constraints can lead to suboptimal solutions.
Overestimating Statelessness: Assuming all computations can be done statelessly without considering historical dependencies.
Neglecting Scalability Issues: Not accounting for the impact of state management on scaling and fault tolerance.
Operational Costs:

Pushing statefulness into the platform incurs several operational costs, including:

Increased storage requirements
Higher processing overhead
More complex maintenance and recovery procedures

**Comparison:**

```
Computation Type	Key Characteristics	Example Scenarios
Stateful	Maintains internal state; requires storage	Anomaly detection, session tracking
Stateless	No persistent state; simpler to implement	Real-time alerts, simple aggregations
```

**Code:**

```python
# Example Python code snippet to illustrate stateful vs. stateless computations
def process_event_statefully(event):
    # Retrieve or update the baseline state
    baseline = get_baseline_from_storage()
    if event['value'] > baseline:
        detect_anomaly(event)

def process_event_statelessly(event, window_size=60):
    # Calculate simple aggregations without maintaining states
    avg_value = calculate_average(event, window_size)
    if avg_value > threshold:
        send_alert(avg_value)
```

**Example input / output:**

```text
Input:
Event value: 25, Baseline: 15

Output (Stateful):
Anomaly detected.

Input:
Events in the last minute: [10, 12, 14, 26]

Output (Stateless):
Average value: 16.0, Threshold exceeded, Alert sent.
```

**Data:**

```json
{
  "baseline": {
    "value": 15,
    "timestamp": "2023-10-01T12:00:00Z"
  },
  "event": {
    "value": 25,
    "timestamp": "2023-10-01T12:01:00Z"
  }
}
```

**Flow:**

1. No
2. Yes
3. Receive Event
4. Is Stateful Required?
5. Process Statelessly
6. Maintain State
7. Perform Computation andUpdate State

