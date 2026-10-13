---
type: "practice"
difficulty: "Deep"
---

# How do isolation levels shape your choice between a queue-in-the-database pattern and a dedicated broker? Where does SKIP LOCKED stop scaling and force the broker decision?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Isolation levels are critical in determining whether to use a queue-in-the-database (queue-ITD) pattern or a dedicated broker for message queuing. The key factors include transactional consistency, concurrency, and scalability. SKIP LOCKED can introduce challenges at high throughput, necessitating the choice of a dedicated broker.

#### Triaging the Question

The question asks about choosing between two queue implementations: queue-in-the-database (queue-ITD) versus a dedicated message broker, considering isolation levels such as Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable. The specific focus is on where SKIP LOCKED can lead to scalability issues, pushing for the use of a broker.

#### Assumptions
and Context
Queue-in-the-database: A pattern where messages are stored in a database table with a queue-like structure.
Dedicated Broker: A separate system responsible for managing message queues, typically using protocols like AMQP or MQTT.
Isolation Levels:
RU: Allows reading uncommitted data.
RC: Ensures that once a transaction commits, all subsequent transactions will see the changes.
RR: Provides stronger consistency guarantees but can be more complex to implement.
Serializable: The highest isolation level, ensuring strict consistency at the cost of performance.

#### First-Principles Solution
Isolation Levels and Queue-in-the-database
RU (Read Uncommitted): Not suitable for queue-ITD as it allows reading uncommitted data, leading to inconsistent results.
RC (Read Committed): Suitable but may introduce serialization issues with concurrent transactions modifying the same message.
RR (Repeatable Read): Provides stronger guarantees and can handle more complex transactional requirements but at a cost of performance.
Serializable: Ensures strict consistency, making it ideal for queue-ITD where message integrity is crucial.
Concurrency Considerations

In an ITD pattern, concurrent transactions must be managed carefully to avoid conflicts. READ COMMITTED ensures that no uncommitted data is read, but this might not always be sufficient given the high concurrency requirements of modern applications. REPEATABLE READ and SERIALIZABLE provide stronger guarantees at the cost of reduced performance.

SKIP LOCKED

When using a queue-ITD pattern with repeatable reads or serializable isolation levels, the SKIP LOCKED mechanism can become problematic as it locks rows during read operations to prevent duplicate processing. However, this locking can scale poorly:

Lock Contention: High concurrency might lead to excessive lock contention, causing performance bottlenecks.
Deadlocks and Lock Escalation: Locks on a large number of rows can lead to deadlocks or require more complex lock management strategies, increasing the risk of failure.
Dedicated Broker Approach

Using a dedicated broker like RabbitMQ or Kafka allows for better scalability:

Scalability with Distributed Queues: Brokers handle message routing and distribution across multiple nodes, reducing single-point-of-failure risks.
Dynamic Partitioning: Brokers can automatically partition messages based on load and failover strategies, ensuring high availability.
Message Acknowledgment: Brokers provide reliable acknowledgment mechanisms that are easier to manage than custom solutions in a database.

#### Conclusion

The choice between queue-ITD and a dedicated broker depends heavily on the isolation requirements and expected concurrency:

For moderate to low transactional throughput with strict consistency needs, a queue-ITD pattern using REPEATABLE READ or SERIALIZABLE might be sufficient.
At high transaction volumes, particularly where SKIP LOCKED may introduce scalability issues, a dedicated broker is generally more appropriate.

#### Common Mistakes
and Follow-ups
Ignoring Concurrency: Overlooking the need for strong isolation levels in highly concurrent environments can lead to data inconsistencies.
Overreliance on Locking Mechanisms: Relying too heavily on SKIP LOCKED without understanding its scalability limits can result in performance bottlenecks.
Neglecting Message Durability: Failing to ensure reliable message delivery and acknowledgment can lead to message loss or duplication.

#### Closing Summary

In conclusion, isolation levels significantly influence the choice between queue-ITD and a dedicated broker. While SKIP LOCKED can work for low-to-moderate concurrency scenarios, its limitations at scale often necessitate the use of brokers for reliable and scalable messaging.

**Comparison:**

| ID | Content     | Processed   |
|----|-------------|------------|
| 1  | Message A   | true       |
| 2  | Message B   | false      |

**Code:**

```sql
-- Example SQL to illustrate READ COMMITTED isolation level in an ITD pattern
CREATE TABLE messages (
    id INT PRIMARY KEY,
    content TEXT,
    processed BOOLEAN DEFAULT FALSE
);

BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT * FROM messages WHERE processed = FALSE LIMIT 1 FOR UPDATE; -- Locks the row

-- Simulate processing
UPDATE messages SET processed = TRUE WHERE id = <locked_message_id>;

COMMIT;
```

**Example input / output:**

```text
Input:
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT * FROM messages WHERE processed = FALSE LIMIT 1 FOR UPDATE; -- Locks the row

Output:
+----+----------------+--------------+
| id | content        | processed    |
+----+----------------+--------------+
| 1  | Message A      | false        |
+----+----------------+--------------+

Input:
UPDATE messages SET processed = TRUE WHERE id = 1;

Output:
Query completed successfully.
```

**Data:**

```json
{
    "messages": [
        {"id": 1, "content": "Message A", "processed": false},
        {"id": 2, "content": "Message B", "processed": false}
    ]
}
```

**Flow:**

1. Low Concurrency
2. High Concurrency
3. Lock Contention
4. Very High Concurrency
5. Deadlocks
6. Queue-in-the-database
7. Dedicated Broker
8. RU - Read Uncommitted
9. RC - Read Committed
10. RR - Repeatable Read
11. Serializable

