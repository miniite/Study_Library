---
type: "practice"
difficulty: "Expert"
---

# How would you migrate a cluster from ZooKeeper to KRaft, and what is the metadata log's role in KRaft?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ KRaft Mode](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/KRaft_Mode.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a Kafka cluster from using ZooKeeper as its coordination service to KRaft (Kafka Run-time Application Framework), we need to understand both systems and their differences, especially focusing on metadata management. This migration involves significant changes in how the cluster is managed internally by Kafka itself, without relying on an external coordinator like ZooKeeper.

TL;DR

Migrating from ZooKeeper to KRaft requires setting up a new KRaft-based cluster, ensuring all brokers are aware of the transition, and updating configuration files. The metadata log plays a crucial role in this process, as it stores essential information about topics, partitions, and other metadata that KRaft relies on for operation.

#### Triage

The question asks us to outline the steps required to migrate from ZooKeeper to KRaft and to explain the importance of the metadata log. The core of this migration involves:

Understanding the role of ZooKeeper vs. KRaft.
Preparing a new KRaft-based cluster configuration.
Ensuring smooth transition by making brokers aware of the change.
Utilizing the metadata log for critical metadata storage and management.

#### First-Principles Solution
Step 1: Preparation

Understand ZooKeeper vs. KRaft:

ZooKeeper: Traditionally used as a coordination service where Kafka brokers register topics, partitions, and offsets.
KRaft: An internal Kafka feature that provides metadata management within the cluster itself.

Configuration Preparation:

Update Kafka broker configurations to use KRaft instead of ZooKeeper.
Ensure all necessary libraries and dependencies are in place for KRaft support.
Step 2: Setup a New KRaft Cluster

Start New Brokers with KRaft Mode:

BASH
bin/kafka-server-start.sh config/server.properties --override controller.quorum.type=kraft

Ensure Broker Awareness:

During startup, each broker should be aware of the new configuration and transition to KRaft mode.
Step 3: Metadata Migration

Capture Current Metadata from ZooKeeper:

BASH
bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'retention.ms=86400000' --entity-type topics --entity-name topic_name

Create Equivalents in KRaft:

Use Kafka command-line tools to recreate the metadata.
BASH
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic new_topic --partitions 3 --replication-factor 1
Step 4: Metadata Log Role in KRaft
Metadata Log: In KRaft, the metadata log contains critical information about topics and partitions. It is essential for maintaining cluster state and facilitating operations like rebalancing.
Critical Operations:
Topic Creation/Deletion: The metadata log ensures these changes are recorded and propagated to all brokers.
Partition Assignment: Information on partition assignments, leader elections, and offsets are stored here.
Step 5: Finalize Migration
Monitor Cluster Health:
Use Kafka tools to monitor the state of the cluster and ensure that KRaft is managing metadata correctly.
Verify Metadata Consistency:
BASH
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic topic_name

#### Closing Summary
+ Common Mistakes/Follow-Ups
Closing: Successfully migrating to KRaft involves careful preparation and attention to detail, particularly around metadata management. The metadata log is a critical component that ensures cluster state consistency.

#### Common Mistakes
Failing to properly update configuration files can lead to broker startup failures.
Not capturing all necessary ZooKeeper metadata before transition can result in missing topics or partitions.
Ignoring the importance of monitoring and verification post-migration.
Questions for Follow-Up
How do you handle rolling upgrades during this migration?
What are potential failure modes, and how would you address them?

**Comparison:**

| Broker State | ZooKeeper | KRaft |
|--------------|-----------|-------|
| Coordination | Yes       | No    |
| Metadata Log | External  | Internal |
| Management   | Centralized | Decentralized |

**Code:**

```bash
# Start new broker with KRaft mode
bin/kafka-server-start.sh config/server.properties --override controller.quorum.type=kraft

# Create a topic in KRaft mode
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test-topic --partitions 3 --replication-factor 1
```

**Example input / output:**

```text
Input:
- Start new broker with KRaft mode.
Output:
- Broker starts up successfully in KRaft mode.

Input:
- Create a topic named 'test-topic' with 3 partitions and replication factor of 1.
Output:
- Topic created successfully, with correct metadata stored in the metadata log.
```

**Data:**

```json
{
  "topics": [
    {
      "name": "test-topic",
      "partitions": 3,
      "replicationFactor": 1
    }
  ]
}
```

**Flow:**

1. Yes
2. Start New Broker with KRaftMode
3. Update Configurations
4. Ensure Brokers Are Aware
5. Migrate Metadata fromZooKeeper
6. Create Topics in KRaft Mode
7. Monitor Cluster Health
8. Verify Metadata Consistency

