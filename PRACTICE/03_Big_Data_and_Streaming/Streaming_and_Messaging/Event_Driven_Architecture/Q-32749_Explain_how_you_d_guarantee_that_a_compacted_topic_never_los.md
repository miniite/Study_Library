---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd guarantee that a compacted topic never loses the latest state for a key even though older records are deleted, and what config bugs could still lose the latest.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fat vs Thin Events](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a compacted topic in an event-driven architecture never loses the latest state for a key, despite older records being deleted, we need to deeply understand how compaction works and implement robust configurations. Compaction is essential for maintaining state while managing storage efficiently by deleting old data that does not change.

#### Triage

The question primarily focuses on:

Compaction Mechanism: Understanding why compaction deletes old records.
State Persistence: Ensuring the latest state of a key remains intact even after compactions.
Configuration Bugs: Identifying potential configuration issues that could lead to data loss.

#### Assumptions
and Key Concepts
We are using Apache Kafka, which supports compacted topics via its compact retention policy.
The goal is to maintain the latest state for a key despite periodic deletions of old records due to compaction.
Compaction works by periodically rewriting logs to remove older versions of a record with the same key.

#### First-Principles Solution

To guarantee that the latest state is never lost, we need to implement a combination of strategic configurations and possibly additional logic. Here's how:

Compaction Retention Policy:

Configure the compaction retention period to be shorter than your desired data retention time.
Use log.retention.bytes if compacted topics are expected to store large records.

Record Deletion Control:

Ensure that older records (that do not affect state) are deleted only after compaction has completed for those records.
Kafka does not support partial compactions, so full compaction must complete before deletions can occur.

Leader Election Strategy:

Implement a leader election strategy to ensure that the latest leader is always available and that it handles compaction correctly.

State Management:

Use additional state management techniques like snapshotting or external storage to back up critical data.
Periodically take snapshots of key states in an external storage system, such as HDFS or a dedicated database.

Monitoring and Alerting:

Set up monitoring to detect anomalies or potential issues with compaction.
Use alerting mechanisms to notify the team when compactions are failing or taking too long.

#### Implementation
Steps

Configure Kafka Topic for Compaction:

PROPERTIES
topic.config:
  compression.type=producer # Ensure producers compress messages
  message.format.version=2.8-IV2 # Use latest schema if needed
  retention.ms=604800000 # 7 days of data, adjusted as per requirements
  delete.retention.ms=302400000 # 5 days for compaction, must be less than retention.ms
  log.cleaner.enable=true

Add Snapshotting Logic:

Implement a background process that periodically takes snapshots of critical key states.
PYTHON
import datetime

def snapshot_state(topic_name):
    current_time = datetime.datetime.now()
    # Save state to external storage with timestamp
    save_to_storage(f"{topic_name}_{current_time.strftime('%Y%m%d_%H%M%S')}")

Leader Election and Compaction:

Use Kafka's built-in leader election mechanisms or implement a custom solution.
PROPERTIES
# Ensure proper leader election strategy
replica.election.timeout.ms=5000

Monitoring and Alerting Setup:

Configure Kafka Manager, Prometheus, and Grafana for real-time monitoring.
Set up alerts to notify the team on compaction failures or data loss risks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By carefully configuring Kafka topics with appropriate retention policies, implementing state management strategies, and setting up robust monitoring, we can ensure that compacted topics never lose the latest state. Common mistakes include misconfiguring delete.retention.ms leading to premature deletions or insufficient snapshot intervals causing data loss.

#### Follow-ups
might involve discussing specific failure modes in different scenarios or exploring alternative solutions like using a database for critical state management.

**Comparison:**

| Config Parameter | Description |
|------------------|-------------|
| `log.retention.bytes` | Retains logs up to a certain size before deleting old records. |
| `delete.retention.ms` | Controls when older records are eligible for compaction and deletion. |
| `log.cleaner.enable`  | Enables the log cleaner that performs compaction based on retention policies. |

**Code:**

```python
import datetime

def snapshot_state(topic_name):
    current_time = datetime.datetime.now()
    # Save state to external storage with timestamp
    save_to_storage(f"{topic_name}_{current_time.strftime('%Y%m%d_%H%M%S')}")
```

**Example input / output:**

```text
Input:
- Kafka topic with key-value pairs
- Compaction retention configured to delete old records after compaction

Output:
- Latest state for each key is preserved
- Older redundant records are deleted by compaction
- Monitoring alerts if compactions fail or data loss occurs
```

**Data:**

```json
{
  "topic_config": {
    "compression.type": "producer",
    "message.format.version": "2.8-IV2",
    "retention.ms": 604800000,
    "delete.retention.ms": 302400000,
    "log.cleaner.enable": true
  }
}
```

**Flow:**

1. Yes
2. No
3. Produce Key-Value
4. Is Compaction Needed?
5. Compact Old Records
6. Check Latest State
7. Publish Latest State to Storage
8. Monitor & Alert on Data LossRisks
9. Maintain External StateSnapshots

