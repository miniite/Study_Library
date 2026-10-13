---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between deleting old data by time retention and keeping only the latest value by compaction, with a data-warehouse example.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In a data warehouse environment using Apache Kafka, managing large volumes of streaming data is crucial. Two common strategies are time retention and compaction. Time retention involves deleting old messages after a certain period, while compaction retains only the latest value for each key. This explanation will highlight the differences between these methods in an intermediate Data Engineer context.

#### Triaging the Question

The question is asking us to differentiate between two data management techniques: time retention and compaction. Time retention involves deleting old messages after a certain period, whereas compaction retains only the latest value for each key. Both strategies are used to manage the lifecycle of data in Kafka topics, but they serve different purposes and have distinct implications.

#### Assumptions
We are dealing with a Kafka topic storing events or records.
The context is within a data warehousing scenario where historical data needs careful management.
Time retention involves setting a time duration after which old messages should be deleted.
Compaction retains the latest value for each unique key, potentially overwriting older values.

#### First-Principles Solution
1. Time Retention
Definition: Time retention is a policy where Kafka automatically deletes records from a topic based on their age or timestamp. For example, if you set a retention period of one week, any messages older than that will be removed.

#### Pros
Simplifies data management by ensuring the topic does not grow indefinitely.
Reduces storage costs and complexity in managing old data.

#### Cons
Loss of historical data: You may need to periodically back up or archive data for regulatory, compliance, or analytical purposes.
Time-based policies can be inflexible if you need to analyze the exact state of the system at certain times.
2. Compaction
Definition: Compaction is a process that retains only the latest value for each unique key in a Kafka topic. This means older messages with the same key will be overwritten, leaving only the most recent ones.

#### Pros
Preserves historical data: You can still analyze and retrieve past states using the key-value pairs.
Can be more flexible than time-based retention policies as it focuses on uniqueness rather than age.

#### Cons
Increased storage requirements for topics where many keys are updated frequently, as old messages will take up space until they are overwritten.
Compaction is resource-intensive and can impact topic performance during the compaction process.

#### Example Scenario

Imagine a data warehouse that tracks user transactions. Each transaction generates an event with a unique key (e.g., user_id). Over time, this stream of events can become very large. Using time retention might mean keeping only the last month's worth of transactions, which simplifies storage and reduces costs but loses historical context. On the other hand, using compaction ensures that every transaction is stored, allowing for detailed analysis over time.

#### Closing Summary

In summary, both time retention and compaction are strategies to manage the lifecycle of data in Kafka topics. Time retention focuses on age-based deletion, which simplifies storage but may lose historical context. Compaction retains only the latest value for each key, preserving history while potentially increasing storage requirements. The choice between these methods depends on specific use cases and requirements.

#### Common Mistakes
Incorrectly setting time retention policies: This can lead to accidental data loss if not properly configured.
Overlooking compaction when it is needed: Compaction helps preserve historical data, but its setup requires careful consideration of storage costs and performance implications.
Additional Considerations
Backups and archiving: For scenarios requiring long-term storage or regulatory compliance, consider implementing periodic backups or archival solutions alongside time retention or compaction.
Performance monitoring: Regularly monitor the performance impact of compaction to ensure it does not significantly affect topic throughput.

**Comparison:**

```
Time Retention	Compaction
Deletes old messages based on age	Retains only the latest value for each key
Loss of historical context	Preserves historical data
Simpler storage management	Higher storage requirements and compaction overhead
```

**Code:**

```python
# Example Kafka configuration for time retention
{
  "log.retention.hours": "24",
  "delete.retention.ms": "1d"
}

# Example Kafka configuration for compaction
{
  "min.compaction.lag.ms": "500ms",
  "max.compaction.lag.ms": "1h"
}
```

**Example input / output:**

```text
# Time Retention Example (Input)
10:00 - User ID 12345, Transaction Amount = $10.00
10:05 - User ID 12345, Transaction Amount = $20.00

# Time Retention Example (Output)
Only the latest transaction is kept.
10:05 - User ID 12345, Transaction Amount = $20.00

# Compaction Example (Input)
10:00 - User ID 12345, Transaction Amount = $10.00
10:05 - User ID 12345, Transaction Amount = $20.00

# Compaction Example (Output)
Both transactions are kept.
10:00 - User ID 12345, Transaction Amount = $10.00
10:05 - User ID 12345, Transaction Amount = $20.00
```

**Data:**

```json
// Sample data schema for a user transaction topic
{
  "user_id": "string",
  "transaction_amount": "float",
  "timestamp": "long",
  "other_fields": "array"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Time Retention
6. Is it past retention time?
7. Delete message
8. Keep message
9. Compaction
10. Has a newer message for thesame key arrived?
11. Replace old message with newone
12. Keep current message as latest

