---
type: "practice"
difficulty: "Deep"
---

# What is a leader and follower for a Kafka partition, and what happens when a leader fails?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partition Keys](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's start with a brief overview: In Apache Kafka, each topic is divided into partitions, and these partitions need to be managed by brokers that act as leaders and followers. When a leader fails, one of the followers takes over the leadership role to ensure data availability and reliability.

Now, let’s dive deeper:

#### Triaging the Question

The question asks about the roles of "leader" and "follower" in Kafka partitions and what happens when a leader fails. The key aspects being probed are:

Leadership Mechanism: Understanding how brokers become leaders for different partitions.
Fault Tolerance: Exploring the failover process to ensure continuous operation.

#### Assumptions
We assume you have a basic understanding of Apache Kafka and its distributed architecture.
Each partition is replicated across multiple brokers for fault tolerance.

#### First Principles Solution

Leader Assignment:

In Apache Kafka, leaders are dynamically assigned by the Kafka cluster itself. The leader election process ensures high availability and reliability.
A broker that becomes a leader for a partition is responsible for handling all incoming writes (produced messages) and delivering them to consumers of that topic.

Follower Role:

Followers replicate data from the leader in real-time by consuming messages produced by the leader.
This replication process ensures that even if the leader fails, one of the followers can take over leadership without significant loss of data or downtime.

Leader Election:

When a leader broker goes down, the Kafka cluster performs an election to select a new leader from among the available replicas (followers).
The new leader is chosen based on the replication factor and the current state of each replica. Typically, the broker with the most recent offset or the one that has been idle for the least amount of time might be selected.

Follower Takeover:

Once a new leader is elected, it begins serving reads (messages consumed by consumers) and writes (new messages produced).
The old leader remains in a "ISR" (In-Sync Replicas) state until it synchronizes its log with the new leader. If the old leader never catches up, it will eventually be marked as an "OOSR" (Out-Of-Sync Replica) and removed from service.

#### Closing Summary

When a Kafka partition's leader fails, the cluster automatically selects one of the followers to become the new leader. This process ensures that data availability is maintained without significant disruptions. The failover mechanism is designed to be as seamless as possible to minimize downtime and loss of data integrity.

#### Common Mistakes/Follow-Ups
Failing to Understand ISR: A common mistake is not recognizing the importance of the "ISR" state, which determines whether a replica is considered available for leadership.
Misunderstanding Leader Elections: There can be confusion about how leader elections are triggered and managed within the Kafka cluster.

If you need further details on specific scenarios or configurations, feel free to ask!

**Comparison:**

| Broker ID | State    |
|-----------|----------|
| 1         | Follower |
| 2         | Leader   |
| 3         | Follower |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Broker 1] -->|Replica of Partition| B[Broker 2]
    B --> C[Leader for Partition]
    C -->|Failure| D[Failover]
    D --> E[Follower for Partition]
    E --> F[New Leader]
```

**Example input / output:**

| Input (Topic Creation) | Output (Leader Assignment) |
|------------------------|----------------------------|
| `kafka-topics.sh --create --topic example-topic --partitions 3 --replication-factor 3` | Leader assigned to Broker 2, Followers on Brokers 1 and 3. |
| Input (Leader Failure) | Output (Failover Process) |
|------------------------|----------------------------|
| Broker 2 goes down     | Broker 3 becomes the new leader after a few seconds. |

**Data:**

```json
{
  "brokers": [
    {"id": 1, "state": "follower"},
    {"id": 2, "state": "leader"},
    {"id": 3, "state": "follower"}
  ],
  "partitions": {
    "topic-0": {
      "replicas": [2, 1, 3]
    }
  }
}
```

**Flow:**

1. No
2. Yes
3. Partition
4. Leader Election
5. Leader Failure?
6. Replicas Sync with New Leader
7. New Leader Elected
8. Service Continuity Maintained

