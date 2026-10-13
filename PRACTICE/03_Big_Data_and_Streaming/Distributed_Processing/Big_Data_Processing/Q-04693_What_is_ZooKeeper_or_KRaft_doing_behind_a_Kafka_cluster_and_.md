---
type: "practice"
difficulty: "Basic"
---

# What is ZooKeeper or KRaft doing behind a Kafka cluster and why should a fresher at least know it exists?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Topic Partitioning and Ordering](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_Partitioning_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Kafka cluster, ZooKeeper acts as the central coordination service that ensures the reliability and consistency of key configurations and state information. For a fresher in AIOps, understanding that ZooKeeper plays this crucial role is fundamental because it lays the groundwork for comprehending how Kafka maintains its operational integrity.

ZooKeeper's primary responsibilities within a Kafka cluster include managing metadata such as topics, partitions, and brokers' states. It also handles election processes to determine which broker will serve as the controller node, ensuring that the cluster can self-heal in case of failures. By knowing this, you grasp why ZooKeeper is essential; it ensures that all components are aware of each other's status and responsibilities.

To break down the concept from first principles:

Configuration Management: ZooKeeper maintains configuration data like topic names, partition counts, replication factors, and broker details.
State Coordination: It coordinates state across brokers to ensure consistency in cluster operations.
Leadership Election: ZooKeeper helps elect leader brokers for each partition, ensuring high availability.
Failover Mechanism: In case of failures, it enables the Kafka cluster to self-heal and continue operating.

For a fresher, knowing these roles is crucial because they form the foundational understanding needed for more advanced topics in AIOps. For example, understanding how ZooKeeper manages metadata will help you grasp concepts like broker discovery and topic management, which are critical for designing and troubleshooting Kafka clusters.

**Code:**

```text
# Example code to interact with ZooKeeper from Python
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()

# Create a node in ZooKeeper for metadata storage
zk.create('/kafka/topic/config', b'{"partition_count": 3, "replication_factor": 2}')

zk.stop()
```

**Flow:**

1. Bootstrap
2. Connect to ZooKeeper
3. Read metadata from/kafka/topic/config
4. Update configurations asneeded
5. Notify all brokers aboutchanges

