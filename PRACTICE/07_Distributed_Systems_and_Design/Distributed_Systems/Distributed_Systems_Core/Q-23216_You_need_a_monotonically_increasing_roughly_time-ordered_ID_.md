---
type: "practice"
difficulty: "Deep"
---

# You need a monotonically increasing, roughly time-ordered ID generator across many hosts (Snowflake-style). Walk through the bit layout, the clock-skew failure mode, and how you handle a backwards clock jump.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of generating monotonically increasing IDs across multiple hosts in a distributed system (similar to Snowflake's approach), we need to carefully design the bit layout, consider potential failure modes like clock skew, and implement strategies to handle backwards clock jumps.

Firstly, let’s break down the core concept: we aim for an ID generator that ensures uniqueness and monotonicity across all hosts. This is crucial in distributed systems where multiple nodes might generate IDs independently but need a globally consistent ordering.

#### Triage

The question primarily probes our understanding of:

Bit Layout: How to efficiently allocate bits to ensure uniqueness and prevent collisions.
Clock Skew Failure Mode: The scenario where clocks on different hosts are not synchronized, leading to potential issues in ID generation.
Backwards Clock Jumps: Handling cases where a host's clock jumps backward unexpectedly.
Bit Layout

A common approach is to use the Snowflake algorithm’s bit layout, which divides the 64-bit ID as follows:

10 bits for the data center ID: This helps in identifying the geographic or rack location.
5 bits for the machine ID within the data center: Helps in managing the node count.
12 bits for the sequence number: Used to generate a unique ID per millisecond.
41 bits for the timestamp: Captures the time, ensuring uniqueness over a wide range.
Clock Skew Failure Mode

Clock skew is a critical issue because if clocks are not synchronized across hosts, IDs generated at different times might overlap. For example, two hosts might think they are generating new IDs based on their current clock readings, but in reality, these timestamps could be out of order due to the skew.

Backwards Clock Jumps

Backwards clock jumps occur when a host's clock moves backward, which violates causality and can lead to ID generation anomalies. For instance, if Host A generates an ID at 10:00:00.500 and then its clock jumps backward to 9:59:59.450 before moving forward again, it might generate a timestamp that appears earlier than previously.

#### First-Principles Solution

To mitigate these issues:

Clock Synchronization: Ensure all hosts have synchronized clocks within acceptable drift limits (e.g., using NTP). However, this is often impractical in distributed systems.
Backwards Clock Handling:
Sequence Number Wraparound Detection: Implement a mechanism to detect when the sequence number would wrap around due to a backwards clock jump. If detected, refuse to generate IDs until the sequence can be reset or the clock adjusted.
Versioned Timestamps: Use versioned timestamps where each timestamp is paired with a version ID. If a host detects its clock has jumped backward, it can request a new version ID from a central authority (like an API call) and restart generating IDs with this new version.

#### Implementation

Here’s a high-level flow:

Timestamp Collection: Each host collects the current timestamp and machine ID.
Sequence Generation: If no clock skew is detected, generate the next sequence number and combine it into the full 64-bit ID.
Clock Skew Detection: Implement logic to detect if the generated timestamp is earlier than previously recorded timestamps on the same node or any other node.
Backwards Clock Handling:
If a backwards clock jump is detected, request a new version from a central authority.
Use the new version ID to reset the sequence and continue generating IDs.

#### Closing Summary

In conclusion, designing an ID generator for distributed systems requires careful consideration of bit allocation, clock synchronization, and handling potential failures like clock skew and backwards jumps. Proper implementation ensures that IDs remain unique and monotonically increasing across all nodes.

#### Common Mistakes/Follow-Ups
Overlooking Clock Synchronization: Assuming clocks are always synchronized can lead to failure modes.
Not Handling Backwards Clocks Gracefully: Failing to detect or handle backwards clock jumps properly can result in incorrect ID generation.
Ignoring Centralized Coordination: In some cases, centralized coordination might be necessary to manage sequence numbers and timestamps.

**Comparison:**

```
Data Center ID	Machine ID	Sequence Number	Timestamp
8	5	3276	1_608_779_878_000
	9	4096
```

**Code:**

```python
def generate_id(timestamp, machine_id, sequence_number):
    # Example bit layout for 64-bit ID (10+5+12+41)
    data_center_id = 0b0000101000  # Hypothetical data center ID
    sequence_bitmask = 0x3FFF  # Mask for last 12 bits
    version_bitmask = 0xFFFF_FFF8  # Mask for first 41 bits

    # Combine elements
    id_part1 = (data_center_id << 52) | (timestamp << 42)
    id_part2 = (machine_id << 30) | sequence_number
    full_id = (id_part1 & version_bitmask) | id_part2

    return full_id
```

**Example input / output:**

```text
Input:
- data_center_id = 8 (binary: 0b0000101000)
- machine_id_1 = 5, sequence_number_1 = 3276
- timestamp = 1_608_779_878_000

Output:
- ID_1: (0b0000101000 << 52) | (1_608_779_878_000 << 42) | (5 << 30) | 3276
```

**Data:**

```json
{
  "data_center": {
    "id": 8,   // Hypothetical data center ID in binary: 0b0000101000
    "machines": [
      {"id": 5, "sequence_number": 3276},  // Machine ID and sequence number for generation
      {"id": 9, "sequence_number": 4096}
    ]
  },
  "timestamp": 1_608_779_878_000,  // Hypothetical Unix timestamp in milliseconds
  "version_bitmask": 0xFFFF_FFF8,
  "sequence_bitmask": 0x3FFF
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is Clock Synchronized?
7. Generate ID
8. Request New Version fromAuthority
9. Handle with New Version andSequence Reset
10. Detect Backwards Jump?
11. Continue Generating IDs
12. Reset Sequence Number andGenerate New ID

