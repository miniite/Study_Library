---
type: "practice"
difficulty: "Intermediate"
---

# You need a globally-unique, roughly time-ordered ID for every alert your fleet emits so you can join it to the incident and to later feedback. Explain a Snowflake-style scheme, its clock-skew failure mode, and why you'd prefer it over a central counter that becomes an ingest bottleneck.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate globally-unique, roughly time-ordered IDs for alerts in a distributed system like AIOps, we can implement a Snowflake-style scheme. This approach uses a combination of timestamp and machine identifiers to create unique IDs that are roughly ordered by when they were generated. Here’s how it works:

Snowflake Scheme Overview: The ID is structured as epoch_ms (64-bit) | worker_id (10 bits) | sequence_number (12 bits). This scheme ensures uniqueness and ordering, making it suitable for our use case.

#### Triage
of the Question:

What the question is asking: We need to describe a distributed ID generation mechanism that avoids bottlenecks or failures due to centralized counters.
Key Probing: The interviewer might be interested in understanding how Snowflake addresses clock skew, scalability issues, and potential failure modes.

#### First-Principles Solution

Snowflake Clock: The core idea is to use a 64-bit timestamp (in milliseconds), which can represent timestamps up to the year 2106. This provides high resolution for our needs.
Worker ID and Sequence Number: The remaining bits are split between a worker_id and a sequence_number, allowing us to track different machines that emit alerts.
Clock Skew Handling: Snowflake addresses clock skew by incrementing the sequence number only when the next timestamp boundary is reached, effectively creating gaps in sequential IDs. This ensures that even if clocks are slightly out of sync, they do not interfere with each other.

#### Closing Summary
and Common Mistakes:

The Snowflake scheme is preferred over a central counter because it avoids single points of failure and bottleneck issues.
Potential pitfalls include improper handling of clock skew, which can lead to gaps in the sequence numbers or collisions if clocks are not synchronized well enough.

Closing Thoughts: Implementing such a system requires careful consideration of synchronization protocols and error handling mechanisms to ensure robustness and reliability across all nodes.

