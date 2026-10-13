---
type: "practice"
difficulty: "Deep"
---

# You need a globally unique, roughly time-ordered ID for every prediction your fleet emits, for logging and joining to labels later. Explain a Snowflake-style scheme, its clock-skew failure mode, and why you'd prefer it over a central counter.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate globally unique, roughly time-ordered IDs for predictions in your distributed system, a Snowflake-style scheme is an excellent choice. This method uses a combination of timestamps and node identifiers to create unique IDs that are monotonically increasing across nodes. However, it's important to understand its failure modes, particularly clock skew between nodes.

#### Triaging the Question

The interviewer is probing for your understanding of distributed systems, especially in generating globally unique identifiers (UUIDs) in a scalable manner. They want to see if you can explain the Snowflake algorithm and identify potential pitfalls like clock skew issues. Additionally, they are interested in comparing this approach with centralized counters.

#### First-Principles Solution
Snowflake Algorithm Overview

A Snowflake-style ID generation scheme uses three components:

Timestamp: A 40-bit integer representing the number of milliseconds since a fixed point in time (usually the Unix epoch).
Machine Identifier: An 8-bit integer that identifies the machine or node within a data center.
Sequence Number: A 12-bit integer that ensures unique IDs are generated even if there is clock skew.

The ID format looks like this: timestamp | machine_id | sequence_number.

Why Use Snowflake
Scalability: The scheme can scale horizontally by adding more nodes, as each node generates its own set of IDs.
Efficiency: It reduces the load on a central server or database since all nodes generate their own unique IDs.
Clock Skew Failure Mode

Clock skew occurs when there is a significant difference in timestamps between two or more nodes. If one node's clock lags significantly behind another, it can lead to overlapping sequence numbers and produce duplicate IDs. To mitigate this:

Time Synchronization: Nodes must be periodically synchronized with each other and an external time source.
Clock Drift Handling: Implement mechanisms to detect and handle clock drifts, such as reattempting ID generation or marking nodes temporarily as inactive.
Centralized Counter vs. Snowflake

A centralized counter system might seem simpler but can become a bottleneck under high load. In contrast, the Snowflake scheme:

Reduces Latency: ID generation is local to each node, reducing network latency.
Scalability: Easier to scale out by adding more nodes without requiring complex coordination.

#### Closing Summary
and Common Mistakes

In summary, the Snowflake algorithm provides a robust solution for generating globally unique IDs in distributed systems. However, it requires careful management of clock skew through time synchronization and drift handling mechanisms. Centralized counters can be simpler but are less scalable and introduce single points of failure.

#### Common Mistakes/Follow-Ups
Overlooking Clock Synchronization: Failing to implement or properly manage node clocks can lead to duplicate IDs.
Ignoring Sequence Number Limits: The 12-bit sequence number has a limited range, which could exhaust available unique values if nodes are not synchronized well enough.
Neglecting Performance Considerations: Not considering the performance impact of periodic time synchronization across many nodes.

**Comparison:**

```
Timestamp	Machine ID	Sequence Number	Generated ID
1609459200000	1	0	386547056
1609459200001	1	1	386547057
```

**Code:**

```python
import time

class Snowflake:
    def __init__(self, node_id):
        self.node_id = node_id  # 8-bit integer
        self.sequence_number_bits = 12
        self.max_sequence_number = (1 << self.sequence_number_bits) - 1
        self.sequence_number = 0
        self.timestamp_bits = 40
        self.time_shift = 2 ** (self.sequence_number_bits + self.timestamp_bits)
        self.last_timestamp = -1

    def generate_id(self):
        timestamp = int(time.time() * 1000)  # Milliseconds
        if timestamp < self.last_timestamp:
            raise Exception("Clock moved backwards. Refusing to generate ID.")

        if timestamp == self.last_timestamp:
            self.sequence_number = (self.sequence_number + 1) & self.max_sequence_number
            if self.sequence_number == 0:
                timestamp = self.wait_until_next_timestamp(self.last_timestamp)
        else:
            self.sequence_number = 0

        self.last_timestamp = timestamp
        return (timestamp - self.time_shift << self.sequence_number_bits) | (self.node_id << self.timestamp_bits) | self.sequence_number
```

**Example input / output:**

```text
# Example Usage
id_generator = Snowflake(node_id=1)
print(id_generator.generate_id())  # Output will depend on current timestamp and sequence number
print(id_generator.generate_id())
```

**Data:**

```json
{
   "node_id": 1,
   "sequence_number_bits": 12,
   "max_sequence_number": 4095,
   "timestamp_bits": 40,
   "time_shift": 386547056, // 2^52
   "last_timestamp": -1
 }
```

**Flow:**

1. Yes
2. No
3. Generate Timestamp
4. Is Clock Skewed?
5. Resynchronize Time
6. Generate ID Using SequenceNumber
7. Return Generated ID

