---
type: "practice"
difficulty: "Deep"
---

# What is the practical difference between commitSync and commitAsync, and when would you combine both in one consumer?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Commits](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Offset_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the practical differences between commitSync and commitAsync in Kafka consumers, we need to first grasp how offsets are managed within a Kafka cluster. The key idea here is that commitSync guarantees immediate acknowledgment of offset updates, ensuring durability at the cost of performance, whereas commitAsync allows for asynchronous updates, reducing latency but potentially introducing risks.

The core question revolves around when and why one might choose to use both methods in a single consumer configuration. Let's break down the reasoning step by step:

Understanding Kafka Consumer Offsets: In Kafka, consumers track their position in a topic partition using offsets. These offsets are stored on the broker and are crucial for ensuring reliable consumption and recovery.

CommitSync vs CommitAsync:

CommitSync: This method ensures that the offset is committed to the log immediately after each message is processed. It guarantees durability but can lead to higher latency because the consumer must wait for the commit operation to complete before processing the next message.
CommitAsync: This method defers the commit operation until a specified number of messages have been processed or a certain amount of time has passed. This reduces latency and improves throughput, but it comes with the risk that if a consumer fails between processing messages and committing offsets, some messages might be missed.

Combining Both Methods:

To combine both methods in one consumer, you would use commitSync for critical operations where immediate acknowledgment is needed to ensure data integrity.
For less critical parts of the workflow, commitAsync can be used to improve performance and reduce latency.

Use Cases:

Critical Sections: Use commitSync when processing sensitive or mission-critical messages that must be durable and recoverable at any cost. This could include financial transactions, user authentication, etc.
Performance-Intensive Sections: Use commitAsync for less critical messages where higher throughput is preferred over immediate acknowledgment. For example, non-real-time analytics or log processing.

#### Example
Configuration:

JAVA

#### cons
umerProperties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); // Disable auto commit
while (true) {

#### Cons
umerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        processMessage(record);
        if (isCriticalSection()) { // Custom logic to determine critical sections

#### cons
umer.commitSync(); // Commit immediately in critical sections
        } else {

#### cons
umer.commitAsync(() -> {}); // Asynchronous commit elsewhere
        }
    }
}

#### Closing Summary
In summary, commitSync and commitAsync provide a way to balance durability with performance in Kafka consumers. By judiciously combining both methods, you can ensure that critical operations are handled robustly while improving the overall throughput of less critical sections.

#### Common Mistakes

Ignoring immediate commit needs for critical operations.
Overusing commitSync, which can significantly degrade performance and resource utilization.
Not considering the failure modes associated with asynchronous commits, such as missed messages or duplicate processing.

