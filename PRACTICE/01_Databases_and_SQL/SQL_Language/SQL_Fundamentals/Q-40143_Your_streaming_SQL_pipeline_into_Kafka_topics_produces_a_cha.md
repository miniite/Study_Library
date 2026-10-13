---
type: "practice"
difficulty: "Deep"
---

# Your streaming SQL pipeline into Kafka topics produces a changelog stream. Explain the difference between a KTable and a KStream and how a GROUP BY over a stream becomes a compacted changelog table.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this interview question, we need to understand the fundamental differences between KStream and KTable in Kafka Streams and how GROUP BY operations on streams transform into compacted changelog tables. This explanation will delve deep into these concepts with a focus on practical applications.

First, let's break down what KStream and KTable are:

TL;DR
KStream: Represents a stream of key-value pairs.
KTable: Represents a table of key-value pairs, which can be seen as a changelog store that tracks updates over time.
GROUP BY on a KStream transforms it into a compacted changelog table, similar to a KTable.

#### Triaging the Question

The interviewer is probing your understanding of Kafka Streams' core concepts and how stateful operations like GROUP BY are implemented. We will assume that you have a basic understanding of Apache Kafka and Kafka Streams, and that we are working with a typical streaming SQL pipeline.

#### First-Principles Solution
KStream

A KStream is essentially an unbounded stream of key-value pairs. It represents the data flowing into your processing application. Each record in the KStream can be thought of as an event or update to some state, but it doesn't keep track of historical updates by default.

KTable

A KTable is a special kind of table that tracks updates over time and maintains a single value per key. It acts like a changelog store, where each new incoming record with the same key replaces the previous one. Kafka Streams automatically compacts these tables to ensure they don't grow indefinitely.

GROUP BY on a KStream

When you apply GROUP BY to a KStream, Kafka Streams effectively transforms it into a compacted changelog table (a KTable). This transformation is crucial for stateful operations like aggregations, joins, and windowing. The key idea here is that the resulting stream maintains a stable view of historical data, which helps in maintaining state consistency.

How GROUP BY Transforms a KStream
Input Stream: Consider a KStream representing a changelog of customer transactions.
GROUP BY Operation: When you perform a GROUP BY on this KStream, Kafka Streams aggregates the records by key and maintains only the most recent value for each group.
Resulting KTable: The output is now a compacted changelog table that tracks the latest state of each customer's transactions.

#### Closing Summary

In summary, KStream represents an unbounded stream of events, while KTable represents a stable view of historical data with one value per key. When you apply GROUP BY to a KStream, it effectively turns into a KTable that maintains the latest state of each group.

#### Common Mistakes
and Follow-Ups
Misunderstanding Compact Streams: Some engineers might confuse compacted streams with simple append-only logs, missing out on how they maintain historical data for stateful operations.
Ignoring State Management: Failure to realize that GROUP BY operations are fundamentally about managing state, leading to potential issues in handling updates and deletions.

To further explore this concept, the interviewer might ask you to write a sample code snippet or discuss edge cases like duplicate keys or late arrivals.

**Comparison:**

| Key | Value |
|-----|-------|
| user1 | 1     |
| user2 | 1     |

**Code:**

```java
// Example Kafka Streams application with GROUP BY transformation
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> transactions = builder.stream("transactions-topic");
KTable<String, Integer> customerBalances = transactions.groupBy((key, value) -> key)
    .reduce((existingValue, newValue) -> existingValue + 1);
```

**Example input / output:**

```text
Input:
- Transaction from user1: deposit
- Transaction from user2: withdrawal
- Transaction from user1: withdrawal

Output (KTable - customerBalances):
- User1: 1 (deposit + 1 withdrawal)
- User2: 1 (withdrawal)
```

**Data:**

```json
{
  "transactions-topic": [
    {"key": "user1", "value": "deposit"},
    {"key": "user2", "value": "withdrawal"},
    {"key": "user1", "value": "withdrawal"}
  ]
}
```

**Flow:**

1. Input Stream
2. GROUP BY
3. Compacted Changelog Table

