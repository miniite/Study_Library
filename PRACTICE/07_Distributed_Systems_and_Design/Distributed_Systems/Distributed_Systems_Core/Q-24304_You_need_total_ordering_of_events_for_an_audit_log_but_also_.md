---
type: "practice"
difficulty: "Deep"
---

# You need total ordering of events for an audit log but also high throughput. Explain the tension between a single partition and many partitions and options like a sequencer service logical timestamps or reordering at read time.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Ordering and Partition Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure total ordering of events in an audit log while achieving high throughput, we need to carefully balance the tension between using a single partition versus multiple partitions and explore options like sequencers, logical timestamps, or reordering at read time. The key is understanding the trade-offs and implications on system complexity, performance, and fault tolerance.

Firstly, let's triage the core of the question: We are dealing with distributed systems where we need to maintain total ordering (i.e., every event has a globally consistent order) for audit logs, while also ensuring high throughput. This is critical because audit logs must be reliable and ordered to ensure data integrity and traceability.

#### Assumptions

The system will handle a large volume of events.
We need strong consistency for auditing purposes.
High availability and fault tolerance are important but secondary concerns at this stage.
Single Partition vs. Multiple Partitions

Single Partition:

#### Pros
Simplified architecture, easier to implement total ordering as all events go through a single point of truth.

#### Cons
Limited scalability due to the bottleneck at the single partition; can become a performance bottleneck and single point of failure.

Multiple Partitions:

#### Pros
Horizontal scalability allows handling more events without a single point of bottleneck. Partitioning can be done based on event type, time, or any other relevant criteria.

#### Cons
Maintaining total ordering across multiple partitions is complex. We need additional mechanisms to ensure global order, such as sequence numbers or timestamps.
Sequencer Service

A sequencer service can help manage total ordering by assigning a unique sequence number to each partition and ensuring that all events within a partition are ordered before any event in another partition. This approach allows for high throughput while maintaining consistency but introduces additional complexity:

#### Implementation
Each message is first passed through the sequencer, which assigns it a global sequence number.

#### Trade-offs
The sequencer can become a bottleneck, and if not designed carefully, it may introduce latency.
Logical Timestamps

Logical timestamps are another approach to maintain total ordering. They leverage the concept of timestamps but with more flexibility than physical time:

#### Implementation
Each event is assigned a logical timestamp that ensures order among all events.

#### Trade-offs
Ensuring logical consistency across partitions can be complex and may require additional coordination.
Reordering at Read Time

In scenarios where real-time total ordering isn't strictly required, we can choose to reorder events during read time. This approach sacrifices strict ordering for improved throughput:

#### Implementation
Events are written with a partition key, allowing them to be ordered locally within the same partition.

#### Trade-offs
Reordering at read time may lead to slightly out-of-order data but is more scalable and simpler to implement.

#### Example Scenario

#### Cons
ider an online transaction system that needs to maintain audit logs for financial transactions. Each transaction generates an event that must be recorded in a strict chronological order to ensure compliance with auditing regulations.

Architecture:

Event Producer
Partition Key
Partition 1
Partition 2
Sequencer Service
Sequencer Service
Order and Write to PersistentStore
Order and Write to PersistentStore

In this setup, events are partitioned by transaction type or time. Each partition has its own sequencer service that assigns logical timestamps and ensures local ordering.

#### Closing Summary

The tension between a single partition and multiple partitions in maintaining total ordering for audit logs lies in balancing the need for strict consistency with the desire for high throughput. Options like sequencers, logical timestamps, or reordering at read time each come with their own trade-offs. For critical applications requiring strong consistency, a sequencer service is often the best choice despite introducing complexity. However, for less stringent requirements, reordering at read time can provide a more scalable solution.

#### Common Mistakes
and Follow-Ups
Ignoring Latency: Overlooking the impact of additional services like sequencers can introduce unacceptable latency.
Simplistic Partitioning: Using overly simplistic partitioning strategies that do not scale well or cause hotspots.
Over-Engineering Fault Tolerance: Focusing too much on fault tolerance without considering the actual risk and impact.

In summary, understanding these trade-offs is crucial for designing robust and scalable distributed systems that meet both consistency and performance requirements.

**Comparison:**

| Event Type | Partition Key          | Logical Timestamp |
|------------|-----------------------|------------------|
| financial   | transactions           | 1                |
| payment     | bank_transactions      | 2                |
| refund      | customer_support       | 3                |

**Code:**

```python
# Example of logical timestamp assignment in a sequencer service
class SequencerService:
    def assign_timestamp(self, event):
        # Simulate assigning a unique logical timestamp
        return len(self.timestamps) + 1

    timestamps = []
```

**Example input / output:**

```text
Input (Event Producer):
{
  "event": {
    "type": "financial",
    "timestamp": "2023-10-05T14:30:00Z",
    "amount": 100,
    "partition_key": "transactions"
  }
}

Output (Sequencer Service):
{
  "event": {
    "type": "financial",
    "timestamp": "2023-10-05T14:30:00Z",
    "amount": 100,
    "partition_key": "transactions",
    "logical_timestamp": 1
  }
}
```

**Data:**

```json
{
  "event": {
    "type": "financial",
    "timestamp": "2023-10-05T14:30:00Z",
    "amount": 100,
    "partition_key": "transactions"
  }
}
```

**Flow:**

1. Event Producer
2. Partition Key
3. Partition 1
4. Partition 2
5. Sequencer Service
6. Sequencer Service
7. Order and Write to PersistentStore
8. Order and Write to PersistentStore

