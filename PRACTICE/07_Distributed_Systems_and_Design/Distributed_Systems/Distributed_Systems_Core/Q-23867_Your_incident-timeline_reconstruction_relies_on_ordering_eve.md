---
type: "practice"
difficulty: "Deep"
---

# Your incident-timeline reconstruction relies on ordering events from three services whose clocks disagree by 400ms. Explain how you'd use logical or hybrid-logical timestamps stamped at ingest to reconstruct a defensible causal order instead of trusting wall clocks.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To reconstruct an incident timeline in a distributed system with misaligned clock systems, we can leverage logical or hybrid-logical timestamps at ingest. By combining these timestamps with sequence numbers and a coordination service, we ensure that events are ordered correctly according to their true causality, rather than relying on potentially skewed wall clocks.

#### Triaging the Question

The key here is ensuring causal order of events in a distributed system where clock skew exists among different services. The question asks us to design an approach using logical or hybrid-logical timestamps stamped at event ingest to create a defensible timeline reconstruction. This involves understanding the nature of logical and hybrid-logical timestamps, how they work together with sequence numbers, and integrating them into the system.

#### Assumptions
Clock Skew: The three services have clock skews of up to 400ms.
Event Ingestion: Events are ingested at unpredictable times by different services.
Causal Order: We need a method that ensures events are ordered based on their causality, not just the time they were received.

#### First Principles Solution

To solve this problem, we can use logical timestamps (LTS) or hybrid-logical timestamps (HLTs). These approaches involve:

Stamping Timestamps at Ingestion: Each event is stamped with a unique timestamp that is generated at the point of ingestion.
Sequence Numbers: Additional sequence numbers are used to ensure uniqueness and order among events from different services.
Coordination Service: A central service coordinates and resolves conflicts to ensure causal ordering.
Step-by-Step Solution

Logical Timestamps (LTS)

Each event is stamped with a logical timestamp that is generated at the time of ingestion. This can be done using a Lamport clock or a vector clock.

#### Example
If Service A ingests an event, it stamps it with its current local clock value plus a unique sequence number.

Hybrid Logical Timestamps (HLTs)

HLTs combine logical and physical timestamps to mitigate the effects of clock skew. Each service maintains a local logical timestamp and a physical wall clock.

#### Example
Service A stamps an event with its current logical timestamp and wall clock value, then sends it to the coordination service.

Coordination Service

The coordination service receives events from all services and resolves conflicts based on their timestamps.
It ensures that causally related events are ordered correctly by comparing their timestamps and sequence numbers.

#### Example
If Service A stamps an event with logical timestamp 100, Service B with 95, and Service C with 102, the coordination service will order them as 95 (Service B), then 100 (Service A), then 102 (Service C).

Conflict Resolution

If two events from different services have overlapping logical timestamps, the sequence number helps to break ties.

#### Example
If Service A and Service B both generate an event at logical timestamp 95, but A has a higher sequence number than B, it will be ordered before B.

#### Closing Summary

Using logical or hybrid-logical timestamps along with sequence numbers ensures that events are correctly ordered based on their true causality. This approach is robust against clock skew and provides a defensible incident timeline reconstruction.

#### Common Mistakes/Follow-Ups
Misunderstanding Timestamps: Assuming wall clocks alone can provide accurate order.
Neglecting Sequence Numbers: Overlooking the importance of sequence numbers in breaking ties.
Ignoring Coordination Service: Failing to implement a central coordination service for resolving conflicts.

#### Example
Code Snippet
PYTHON
def stamp_event(event, local_logical_timestamp, wall_clock):
    event['logical_timestamp'] = local_logical_timestamp
    event['wall_clock'] = wall_clock

def coordinate_events(events):
    events.sort(key=lambda x: (x['logical_timestamp'], x['sequence_number']))
    return events

#### Example
Data Schema
YAML
event:
  id: string
  logical_timestamp: int
  sequence_number: int
  wall_clock: float
Dry-Run

Input Events:

Service A: {'id': 'A1', 'logical_timestamp': 100, 'sequence_number': 1}
Service B: {'id': 'B1', 'logical_timestamp': 95, 'sequence_number': 1}
Service C: {'id': 'C1', 'logical_timestamp': 102, 'sequence_number': 1}

Output:

YAML
events_sorted:
- {'id': 'B1', 'logical_timestamp': 95, 'sequence_number': 1}
- {'id': 'A1', 'logical_timestamp': 100, 'sequence_number': 1}
- {'id': 'C1', 'logical_timestamp': 102, 'sequence_number': 1}

**Comparison:**

| Event ID | Logical Timestamp | Sequence Number |
| --- | --- | --- |
| A1 | 100 | 1 |
| B1 | 95 | 1 |
| C1 | 102 | 1 |

**Code:**

```python
def stamp_event(event, local_logical_timestamp, wall_clock):
    event['logical_timestamp'] = local_logical_timestamp
    event['wall_clock'] = wall_clock

def coordinate_events(events):
    events.sort(key=lambda x: (x['logical_timestamp'], x['sequence_number']))
    return events
```

**Example input / output:**

```

Input Events:

Service A: {'id': 'A1', 'logical_timestamp': 100, 'sequence_number': 1}
Service B: {'id': 'B1', 'logical_timestamp': 95, 'sequence_number': 1}
Service C: {'id': 'C1', 'logical_timestamp': 102, 'sequence_number': 1}

Output:

YAML
Copy
events_sorted:
- {'id': 'B1', 'logical_timestamp': 95, 'sequence_number': 1}
- {'id': 'A1', 'logical_timestamp': 100, 'sequence_number': 1}
- {'id': 'C1', 'logical_timestamp': 102, 'sequence_number': 1}
```

**Data:**

```yaml
event:
  id: string
  logical_timestamp: int
  sequence_number: int
  wall_clock: float
```

**Flow:**

1. Yes
2. True
3. False
4. Ingest Event
5. Is Logical Timestamp Set?
6. Stamp with Logical andSequence Numbers
7. Send to Coordination Service
8. Receive Events at CoordinationService
9. Resolve Conflicts Using Logicaland Sequence Numbers
10. Order Events by LogicalTimestamp, Sequence Number
11. Order Events by Wall Clock,Sequence Number

