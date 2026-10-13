---
type: "practice"
difficulty: "Deep"
---

# Your alert-noise-reduction service dedupes duplicate alerts using a compacted topic keyed by alert fingerprint. How does compaction help, and what tombstone strategy clears resolved alerts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Compaction in Kafka helps prevent log explosion issues by retaining only the most recent message per unique key, effectively deduplicating duplicate messages over time. In your alert-noise-reduction service, this means that even if an alert is generated multiple times, only the latest instance will be stored, reducing storage and processing overhead.

To understand how compaction works in Kafka and how it clears resolved alerts using tombstones, we need to break down the core concepts step by step. Let's start with a brief overview of the Kafka compacted topic setup and then delve into the tombstone strategy.

**Comparison:**

```
Key	Initial Alert Value	Timestamp	Result
fingerprint1	initial_alert	123456789	retained in logs
fingerprint1	resolved_alert	123456790	retained in logs (latest)
fingerprint1	-	123456791	tombstone message
```

**Code:**

```
MERMAID
Copy
graph TB
    A[Producer] --> B(Compact Topic)
    B --> C{Compaction?}
    C -- Yes --> D{Key Match?}
    D -- No --> E
    D -- Yes --> F[Rewrite Latest Value]
    E --> G[Tombstone Message Produced]
```

**Example input / output:**

```text
Initial input:
- Producer sends an alert with key `fingerprint1` and value `initial_alert`.

Output after compaction setup but before tombstone strategy:
- Partition log retains the latest message for `fingerprint1`: `"resolved_alert"`.

Tombstone input:
- Resolve the `resolved_alert` by sending a tombstone message with key `fingerprint1` and no value.

Output after tombstone processed:
- Tombstone message is added to the end of the partition, marking the resolution.
```

**Data:**

```json
{
  "topic": "alert_fingerprint_compacted",
  "partitions": [
    {
      "partition_id": 0,
      "messages": [
        {"key": "fingerprint1", "value": "initial_alert", "timestamp": 123456789},
        {"key": "fingerprint1", "value": "resolved_alert", "timestamp": 123456790}
      ]
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Producer
6. Compact Topic
7. Compaction?
8. Message Not Compacted
9. Key Match?
10. F
11. Rewrite Latest Value
12. Tombstone Message Produced

