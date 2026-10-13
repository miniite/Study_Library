---
type: "practice"
difficulty: "Intermediate"
---

# For a financial-ledger ingestion topic why would you choose stronger durability settings than for a clickstream topic?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Claim Check Pattern](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Claim_Check_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
When choosing Kafka's durability settings, we need to balance between performance and data reliability based on the nature of the data being ingested. For financial-ledger topics, stronger durability settings are essential due to the criticality of maintaining accurate records, whereas for clickstream topics, lower durability can be acceptable given the high volume and non-critical nature of such data.

#### Triage
The question is asking why we would opt for stronger durability settings in a financial-ledger topic compared to a clickstream topic. This involves understanding the importance of data integrity and the trade-offs between performance and reliability.

Step-by-Step Solution:

Understanding Durability Settings in Kafka: In Kafka, durability can be managed through different configurations like min.insync.replicas, which determines the minimum number of replicas that must acknowledge a write for it to be considered successful, and replication.factor, which specifies how many partitions should replicate data.

Financial-Ledger Topic:

Importance: Financial-ledgers are crucial for accounting, auditing, and legal compliance. Ensuring no data loss is critical.
Durability Settings: To achieve stronger durability:
Increase min.insync.replicas to ensure multiple replicas acknowledge the write before considering it successful.
Use a higher replication.factor to increase redundancy.

Clickstream Topic:

Volume and Velocity: Clickstreams generate vast amounts of data at high speed, which are less critical in terms of exactness due to their nature (e.g., user browsing behavior).
Durability Settings: For lower durability:
Set a smaller min.insync.replicas as the risk of losing some clickstream data is acceptable.
Use a lower replication.factor for better performance and throughput.

#### Trade-offs

Performance vs Reliability: Financial-ledgers require higher reliability at the cost of slightly reduced performance, whereas clickstreams prioritize speed over absolute accuracy.

#### Closing Summary
By tailoring durability settings based on the criticality and nature of data, we can optimize Kafka's performance while ensuring robustness. For financial-ledger topics, stronger durability ensures data integrity, whereas for clickstream topics, lower durability settings allow for faster processing at the expense of occasional data loss.

#### Common Mistakes

Overlooking the importance of data integrity in mission-critical applications like financial-ledgers.
Ignoring performance implications when setting durable settings for high-throughput data like clickstreams.

**Comparison:**

| Topic        | Replication Factor | min.insync.replicas |
|--------------|--------------------|---------------------|
| financial-ledger-topic | 4                 | 3                   |
| clickstream-topic     | 3                 | 2                   |

**Code:**

```python
# Example code to configure Kafka topic with stronger durability settings
from kafka.admin import KafkaAdminClient, NewTopic

admin_client = KafkaAdminClient(bootstrap_servers=['localhost:9092'])

topic_configs = {'min.insync.replicas': '3', 'replication.factor': '4'}
new_topic = NewTopic(name='financial-ledger-topic', num_partitions=5, replication_factor=4, config_values=topic_configs)

admin_client.create_topics(new_topics=[new_topic])
```

**Example input / output:**

```text
Request:
- Create a financial-ledger topic with the following settings:
  - min.insync.replicas = 3
  - replication.factor = 4
  - partitions = 5

Response:
- Topic 'financial-ledger-topic' created successfully with durable settings for high-reliability data.
```

**Data:**

```json
{
  "financial_ledger_topic": {
    "min.insync.replicas": 3,
    "replication.factor": 4,
    "partitions": 5
  },
  "clickstream_topic": {
    "min.insync.replicas": 2,
    "replication.factor": 3,
    "partitions": 10
  }
}
```

**Flow:**

1. Financial-Ledger Topic
2. Strong Durability Needed
3. High min.insync.replicas
4. High replication.factor
5. Clickstream Topic
6. Lower Durability Acceptable
7. Low min.insync.replicas
8. Low replication.factor
9. Critical Data, High Reliability
10. Non-critical Data, High Volume

