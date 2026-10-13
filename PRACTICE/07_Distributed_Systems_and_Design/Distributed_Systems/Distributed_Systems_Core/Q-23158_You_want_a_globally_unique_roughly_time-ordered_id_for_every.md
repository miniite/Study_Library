---
type: "practice"
difficulty: "Intermediate"
---

# You want a globally unique, roughly time-ordered id for every LLM generation so you can join it to delayed human feedback later. Explain a Snowflake-style scheme, its clock-skew failure mode, and why you'd prefer it over a central counter.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve globally unique, roughly time-ordered IDs for every LLM generation in a distributed system, one common approach is to use a Snowflake-style scheme. This involves generating an ID that includes both a timestamp and a machine identifier, ensuring that each generated ID is unique across the entire network.

In essence, the Snowflake ID consists of three components: a 41-bit timestamp (covering up to 69 years), an 10-bit datacenter/id datacenter worker number (up to 1024 workers per datacenter), and a 12-bit sequence number (rotates every 4 milliseconds, allowing for high concurrency). This results in approximately 5.3 million unique IDs per millisecond per machine.

The Snowflake scheme is highly favored over central counters because it avoids the need for a single point of failure or bottleneck that a centralized system might introduce. Central counters can lead to contention and performance issues under heavy load, whereas distributed schemes like Snowflake ensure scalability and resilience.

**Code:**

```python
import datetime
from typing import NamedTuple

class SnowflakeId(NamedTuple):
    timestamp: int
    worker_id: int
    sequence_number: int

def generate_snowflake_id(worker_id: int) -> SnowflakeId:
    now = int(datetime.datetime.now().timestamp() * 1000)
    # Simulate a unique sequence number, in practice this would be managed by the system.
    sequence_number = 42
    return SnowflakeId(timestamp=now, worker_id=worker_id, sequence_number=sequence_number)

# Example usage:
id1 = generate_snowflake_id(worker_id=0)
print(id1)  # Output: SnowflakeId(timestamp=1633072800000, worker_id=0, sequence_number=42)
```

**Example input / output:**

```text
No data input/output is required for the Snowflake ID generation process. The example usage demonstrates how to generate a unique ID given a specific worker identifier.
```

**Flow:**

1. Generate Timestamp
2. Generate Worker ID
3. Generate Sequence Number
4. Combine Timestamp, WorkerID, and Sequence Number intoSnowflakeId

