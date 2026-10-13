---
type: "practice"
difficulty: "Deep"
---

# How do you decide compaction versus time-based retention for a topic that carries the latest health state per host?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with deciding between compaction and time-based retention strategies for Kafka topics that store the latest health state per host. The key is to understand the trade-offs and context of each approach before making a decision.

To tackle this problem, let's first break down what these strategies mean:

Compaction: This involves periodically reorganizing older messages in a topic to reduce the total number of partitions or to consolidate data into fewer files.
Time-based Retention: This strategy retains events based on their timestamp, meaning that after a certain time period, older events are automatically deleted.

Now, let's consider the specific context of our scenario. The health state per host is critical and needs to be up-to-date for analysis. Here’s how we can reason through this:

Understanding the Requirement:

We need to ensure that each host has its most recent health state recorded.
This means that older records should not be retained if they are superseded by newer ones.

Compaction Strategy:

Compaction would help in reducing redundancy and improving storage efficiency.
However, compaction does not delete messages; it only consolidates them into fewer files or partitions.
For our use case, compaction might be useful if we need to reduce the number of partitions for better performance in processing large datasets.

Time-based Retention:

Time-based retention ensures that older data is deleted after a certain period.
This strategy directly controls how long historical data is retained based on timestamps.
If we set a short retention period, it might not preserve enough history for analysis, but if the period is too long, it could consume unnecessary storage.

Decision Factors:

Storage Requirements: If storage space is limited and we can tolerate some data loss, compaction or time-based retention with shorter periods might be more suitable.
Analysis Needs: If historical trends are important for analysis, a longer retention period combined with compaction could be better.
Processing Latency: Compaction might introduce additional latency during the consolidation process.

Given our need to store and analyze the latest health state per host, let’s consider the following:

We would likely want to retain at least the last 24 hours of data for detailed analysis.
For compaction, we could set up a strategy where older partitions are compacted over time, but not immediately.

#### Implementation

Set up Kafka topic retention with a 1-day period initially.
Configure compaction to run periodically (e.g., every 24 hours) on older partitions.
Monitor the storage usage and adjust these settings as needed based on actual performance and resource constraints.

Monitoring and Optimization:

Regularly review the size of partitions and the overall storage consumption.
Adjust compaction frequencies or retention periods as the system evolves.
Ensure that the chosen strategy does not impact real-time processing too much.

In summary, for this scenario, a time-based retention period with periodic compaction seems to be the most balanced approach. This ensures we have recent data available while managing storage efficiently.

**Comparison:**

```
Strategy	Description	Pros	Cons
Compaction	Consolidates older messages into fewer files	Reduces storage overhead; improves performance for large datasets	Introduces potential latency during compaction
Time-based Ret	Deletes old data based on timestamps	Ensures historical data is not retained indefinitely	May consume more initial storage if set too aggressively
```

**Code:**

```python
# Example Python code snippet to configure Kafka topic settings
from kafka.admin import KafkaAdminClient, NewTopic

admin_client = KafkaAdminClient(bootstrap_servers='localhost:9092')
topic_configs = {
    'retention.ms': 86400000, # 1 day in milliseconds
    'cleanup.policy': 'delete' # Enable log cleanup policy for compaction
}
new_topics = [NewTopic(name='health_state', num_partitions=3, replication_factor=2, topic_configs=topic_configs)]
admin_client.create_topics(new_topics)
```

**Example input / output:**

```text
Input:
- Produce health state messages for all hosts every hour.
- Consume the latest health state per host for analysis.

Output:
- Topic retention set to 1 day with log compaction enabled and run periodically.
- Data older than 24 hours is cleaned up, but recent data is stored and accessible.
```

**Data:**

```json
{
  "config": {
    "retention.ms": "86400000",
    "cleanup.policy": ["delete"]
  },
  "compaction.strategy": {
    "enabled": true,
    "type": "log"
  }
}
```

**Flow:**

1. Yes
2. No
3. Configure Topic withTime-Based Retention
4. Is Historical Data Needed?
5. Enable Compaction
6. Monitor Storage Usage andAdjust Settings as Needed
7. Maintain Recent Data, ReduceRedundancy
8. Ensure Performance is NotImpacted by Retention Policies

