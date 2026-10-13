---
type: "practice"
difficulty: "Deep"
---

# How would you set acks and min.insync.replicas differently for a financial-ledger ingestion topic versus a clickstream analytics topic on the same 3-broker cluster?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Durability with acks and ISR](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this Kafka configuration problem, we need to understand that acks and min.insync.replicas play critical roles in ensuring data reliability and availability in a Kafka topic. For financial-ledger ingestion, where accuracy is paramount, we'll prioritize acks. In contrast, for clickstream analytics, which can tolerate some loss or delay, we'll focus on min.insync.replicas.

The question asks us to configure these settings differently based on the nature of the data and the use case. Let's break it down step-by-step from first principles.

Firstly, let’s define what acks and min.insync.replicas do:

acks: This parameter controls how many acknowledgments a producer waits for before considering a message sent successfully.

all: All brokers (default) for the partition must confirm receipt of the message.
leader: Only the leader broker needs to confirm receipt of the message.
none: The producer does not wait for any confirmation.

min.insync.replicas: This parameter controls how many in-sync replicas must exist before a write operation is considered successful. It ensures that even if some brokers fail, the data can still be recovered from other live replicas.

Now, let's configure these settings:

Financial-Ledger Ingestion Topic

For financial-ledgers, we need to ensure absolute data integrity and fault tolerance:

acks=all: Ensure that all brokers confirm receipt of messages. This is crucial for critical transactions like financial entries.
min.insync.replicas=3: As this topic runs on a 3-broker cluster, setting min.insync.replicas to the number of nodes ensures that even if one node fails, we still have two replicas available.
Clickstream Analytics Topic

For clickstream analytics, some data loss might be acceptable due to its nature and real-time processing requirements:

acks=leader: This is a more relaxed setting where only the leader broker needs to confirm receipt. It speeds up message delivery at the expense of slightly increased risk.
min.insync.replicas=2: Given that we have 3 brokers, having two in-sync replicas allows for fault tolerance while still enabling quick recovery.
Summary and Closing

To summarize, financial-ledger ingestion requires strict data reliability (acks=all, min.insync.replicas=3) to ensure no loss of critical transactions. In contrast, clickstream analytics can tolerate more relaxed settings (acks=leader, min.insync.replicas=2) due to its nature and the real-time requirements.

#### Common Mistakes/Follow-Ups
Misunderstanding Acks: Using none for financial data could lead to undetectable losses.
Insufficient Replication: For clickstream, using a single in-sync replica (min.insync.replicas=1) might be insufficient and increase the risk of data loss.

Would you like me to elaborate on any specific points or consider additional configurations?

**Comparison:**

| Topic Type      | `acks`   | `min.insync.replicas` |
|-----------------|----------|-----------------------|
| Financial-Ledger| all      | 3                     |
| Clickstream     | leader   | 2                     |

**Code:**

```python
# Example Kafka topic configuration for financial-ledger
financial_ledger_config = {
    "acks": "all",
    "min.insync.replicas": 3
}

# Example Kafka topic configuration for clickstream analytics
clickstream_analytics_config = {
    "acks": "leader",
    "min.insync.replicas": 2
}
```

**Example input / output:**

```text
Input (financial-ledger topic configuration):
{
    "acks": "all",
    "min.insync.replicas": 3
}

Output (configuration applied to Kafka cluster for financial-ledger topic):

Input (clickstream analytics topic configuration):
{
    "acks": "leader",
    "min.insync.replicas": 2
}

Output (configuration applied to Kafka cluster for clickstream analytics topic):
```

**Data:**

```json
{
  "financial_ledger_topic": {
    "acks": "all",
    "min.insync.replicas": 3
  },
  "clickstream_analytics_topic": {
    "acks": "leader",
    "min.insync.replicas": 2
  }
}
```

**Flow:**

1. Financial-Ledger
2. acks=all
3. min.insync.replicas=3
4. Clickstream
5. acks=leader
6. min.insync.replicas=2

