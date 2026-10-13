---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd choose between event-time and processing-time partitioning for a CDC stream where source updates can arrive out of order.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Change Data Capture (CDC) streams in a system that processes data from a database or other sources, choosing between event-time and processing-time partitioning is crucial for ensuring accurate and efficient data handling. Event-time partitioning focuses on the timestamp of each individual record, while processing-time partitioning relies on the timestamp of when the data was processed. For an out-of-order stream where source updates can arrive at irregular intervals, understanding these two concepts will help us make a strategic decision.

**Code:**

```python
# Example function to simulate event-based and time-based partitions
def partition_by_event_time(record):
    return record['timestamp']

def partition_by_processing_time(record):
    return datetime.now().strftime('%Y-%m-%d')
```

**Example input / output:**

```

No specific input-output examples are needed for this explanation, but a dry-run might look like:

PLAINTEXT
Copy
Event-time partitioning example with out-of-order events:
Record 1 (timestamp: 2023-10-01T15:00Z) -> Partition 2023-10-01
Record 2 (timestamp: 2023-10-01T14:30Z) -> Partition 2023-10-01

Processing-time partitioning example:
Record 1 (timestamp: 2023-10-01T15:00Z) -> Partition current date
Record 2 (timestamp: 2023-10-01T14:30Z) -> Partition current date

Assuming the processing time is now 2023-10-01T16:00Z, both would end up in the same partition.
```

**Data:**

```

No specific data schemas are required for this explanation, but typical CDC records might look like:

JSON
Copy
{
  "id": "1234",
  "data": {"field1": "value1", "field2": "value2"},
  "timestamp": "2023-10-01T14:30:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Event-time Partitioning
6. Is timestamp known?
7. Use event-time for partitioning
8. Use processing-time asfallback
9. Process data with currenttimestamp
10. Processing-time Partitioning
11. Is current time available?
12. Use processing-time forpartitioning
13. Fallback to another method,e.g., hash-based partitioning

