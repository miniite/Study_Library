---
type: "practice"
difficulty: "Expert"
---

# How would you architect soft multi-region writes where users write to their nearest region? Explain conflict resolution, unique-ID generation, and why a single-writer or CRDT model may be needed.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with designing a system that supports soft multi-region writes, where users can write data to their nearest region. This architecture requires handling conflicts and ensuring unique IDs are generated efficiently across regions. Let's break down the problem step-by-step.

TL;DR

To achieve soft multi-region writes, we need a distributed database design that supports concurrent writes from multiple regions while resolving conflicts through a conflict resolution strategy. Unique IDs must be generated in such a way that they remain globally unique despite being generated independently by different regions.

#### Triage

The core of this question revolves around ensuring data consistency and uniqueness across multiple geographic regions, especially when users can write to their nearest region. The key points the interviewer is probing include:

Conflict Resolution: How will conflicts be detected and resolved?
Unique-ID Generation: What mechanism ensures global uniqueness?

#### Cons
istency Model: Is a single-writer model or a conflict-free replicated data type (CRDT) required?

#### First Principles Solution
Conflict Resolution

Versioning and Timestamps

Each write operation should include a timestamp to identify the order of writes.
A version number can be incremented with each write, ensuring that newer versions overwrite older ones.

Conflict Detection Mechanism

On read operations, compare the version or timestamp of the data from different regions to detect conflicts.
If multiple writes have occurred in parallel, we need a mechanism to decide which one wins based on some predefined rules (e.g., last write wins).

Conflict Resolution Strategies

Last Write Wins (LWW): The latest version wins.
Vector Clocks: Use vector clocks for distributed systems to track the causality and detect conflicts.
Unique-ID Generation

UUIDs with Region Prefixing

Generate UUIDs with a prefix that indicates the region from which it originated, ensuring global uniqueness across regions.

Distributed ID Generators (e.g., Snowflake)

Use distributed ID generators like Snowflake or Redis's INCR command to ensure unique IDs are generated independently in different regions while maintaining global uniqueness.

Hybrid Approach

Combine region prefixes with incremental counters to generate globally unique IDs, ensuring that even if the same counter is used across multiple regions, the combination of prefix and counter remains unique.
Single-Writer or CRDT Model

Single-Writer Model

In a single-writer model, only one writer per region can write to the database at any given time. This approach simplifies conflict resolution but may introduce latency issues.

#### Example
Use distributed locks or semaphores to ensure that only one instance of an application writes data in each region.

Conflict-Free Replicated Data Type (CRDT)

CRDTs provide a model where data can be replicated and modified concurrently without conflicts, making them suitable for soft multi-region writes.

#### Example
Using sets or maps in CRDTs to manage distributed updates seamlessly.

#### Closing Summary

In summary, achieving soft multi-region writes requires a robust conflict resolution strategy that ensures consistency across regions. Unique-ID generation should account for the independence of each region while maintaining global uniqueness. Depending on the requirements, a single-writer model or a CRDT-based approach can be chosen to handle concurrent writes effectively.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring conflict resolution mechanisms leading to data corruption.

#### Follow-Up
Discussing edge cases like partial failures during write operations and how they are handled.
Mistake: Not considering the scalability implications of using a single-writer model in highly distributed systems.

#### Follow-Up
Exploring alternative strategies for handling partial writes or retries.

**Code:**

```python
# Example Python code for generating region-prefixed UUIDs
import uuid

def generate_region_prefixed_uuid(region):
    return f"{region}-{uuid.uuid4().hex}"

print(generate_region_prefixed_uuid("us-west-1"))
```

**Example input / output:**

```text
Input:
Region: us-east-1, Data: {"key": "value"}

Output:
{
    "_id": "us-east-1-098f6bcd4621d373cade4e832627b4f6",
    "timestamp": 1635626400,
    "version": 1
}
```

**Flow:**

1. Yes
2. No
3. User writes to nearest region
4. Conflict?
5. Resolve conflict (e.g., LWW)
6. Store data, generate unique ID
7. Write resolved data todatabase
8. Write data with unique ID todatabase

