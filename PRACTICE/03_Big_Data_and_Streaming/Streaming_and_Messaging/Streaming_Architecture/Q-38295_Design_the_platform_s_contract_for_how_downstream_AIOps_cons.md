---
type: "practice"
difficulty: "Deep"
---

# Design the platform's contract for how downstream AIOps consumers handle retractions and corrections from the stream (a metric restated, an incident re-scored). How do you make 'provisional then final' a first-class, org-wide semantic instead of ad-hoc per team?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: The challenge is to design a robust contract for downstream AIOps consumers that handle retractions and corrections in real-time streams, ensuring consistency and reliability across the organization. We will use a "provisional then final" semantic as the first-class concept, which means metrics can be restated or incidents re-scored over time, but changes must follow a clear lifecycle to maintain trust.

#### Triage
The interviewer is probing for an understanding of how to manage state and ensure consistency in real-time data streams. They are interested in your ability to design a system that handles retractions and corrections efficiently, while also ensuring that the changes are traceable and auditable. Assumptions include: (1) the platform needs to support both provisional and final states for metrics or incidents; (2) there will be multiple consumers who need to handle these changes.

#### First-Principles Solution
To address this challenge, we'll design a system that follows a "provisional then final" semantic. This involves defining clear rules for when data can be changed from provisional to final and ensuring traceability of all changes. We will use a combination of event-driven architecture, consensus protocols, and versioning mechanisms.

Event-Driven Architecture: The platform will emit events for each change in the stream—initial values, retractions, and corrections. Consumers need to subscribe to these events and update their local state accordingly.

#### Cons
ensus Protocols: To ensure consistency across all consumers, we can use a consensus protocol like Raft or Paxos to maintain a single source of truth. Each time a change is made, it will go through the consensus process before being finalized.
Versioning Mechanism: We'll implement versioning for metrics and incidents to track historical states. This ensures that any consumer can re-fetch old versions if needed.
Traceability Logs: Every change in state (retraction or correction) should be logged with a timestamp, the responsible party, and a unique identifier. This log is crucial for auditing and troubleshooting.

#### Closing Summary
The "provisional then final" semantic ensures that changes in real-time streams are handled consistently across the organization. By using an event-driven architecture, consensus protocols, versioning mechanisms, and traceability logs, we can build a robust system that maintains data integrity while allowing flexibility during retractions or corrections.

#### Common Mistakes/Follow-Ups
Common mistakes include failing to implement proper versioning or traceability, leading to inconsistent states. Follow-ups might involve discussing how to handle conflicting changes, ensuring backwards compatibility with older consumers, and strategies for minimizing the impact of retractions on real-time alerts.

**Comparison:**

```
Event Type	Payload Fields
Metric Retraction	metric_id, retracted_value, timestamp
Metric Correction	metric_id, new_value, timestamp
```

**Code:**

```python
# Example event structure
class Event:
    def __init__(self, type, payload):
        self.type = type  # 'metric_retraction', 'metric_correction', etc.
        self.payload = payload

# Example consumer handling provisional to final state changes
def handle_event(event):
    if event.type == 'metric_correction':
        update_metric_state(event.payload['metric_id'], event.payload['new_value'])
        log_change('Metric correction', metric_id=event.payload['metric_id'], user='user123')
```

**Example input / output:**

```

Input: Stream event of type metric_correction with payload:

JSON
Copy
{
  "type": "metric_correction",
  "payload": {
    "metric_id": "1234567890",
    "new_value": 2.5,
    "timestamp": "2023-10-01T13:00:00Z"
  }
}

Output: Update metric state to 2.5 and log change:

JSON
Copy
{
  "type": "metric_correction",
  "payload": {
    "metric_id": "1234567890",
    "new_value": 2.5,
    "timestamp": "2023-10-01T13:00:00Z"
  }
}
```

**Data:**

```json
{
  "event": {
    "type": "metric_retraction",
    "payload": {
      "metric_id": "1234567890",
      "retracted_value": 1.5,
      "timestamp": "2023-10-01T12:00:00Z"
    }
  }
}
```

**Flow:**

1. Event Occurs
2. (Event Type)
3. Metric Retraction
4. Update State, Log Change
5. Metric Correction
6. Update State, Log Change,Consensus Protocol

