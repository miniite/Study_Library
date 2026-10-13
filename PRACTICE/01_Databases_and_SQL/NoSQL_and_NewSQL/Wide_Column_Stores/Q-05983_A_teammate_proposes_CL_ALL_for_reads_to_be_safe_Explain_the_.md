---
type: "practice"
difficulty: "Deep"
---

# A teammate proposes CL=ALL for reads to 'be safe'. Explain the availability cost, what happens when one replica is down, and how you'd talk them into QUORUM instead.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Wide Column Design and Migration](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Wide_Column_Design_and_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address your teammate's proposal of using CL=ALL for reads in Cassandra/DynamoDB to 'be safe,' we need to understand the implications on availability, particularly when a replica is down. We'll delve into how this setting affects read operations and discuss why choosing QUORUM might be more balanced.

#### Triage
The Question and What's Being Probed

The question revolves around understanding the trade-offs between consistency and availability in NoSQL databases like Cassandra or DynamoDB, specifically focusing on the CL=ALL vs. QUORUM configurations. Your teammate suggests CL=ALL, which means all replicas must confirm a read request before returning data. The interviewer is probing your knowledge of distributed systems, the CAP theorem, and how to make informed decisions about consistency levels.

#### Assumptions
We are dealing with a NoSQL database that uses a wide-column store like Cassandra or DynamoDB.
The concern is around read operations failing if one replica is down.
CL=ALL ensures high consistency but can degrade availability when replicas fail.
QUORUM provides a balance between consistency and availability.

#### First Principles Solution
Understanding Consistency Levels

In Cassandra, the CONSISTENCY_LEVEL (CL) determines how many replicas must acknowledge a request for it to be considered successful. Setting CL=ALL means that all replicas must respond with data before the read is deemed complete. This ensures strong consistency but comes at the cost of availability when one or more replicas are down.

Impact on Availability

When using CL=ALL, if any replica fails, no reads can proceed until the failed node recovers and responds. In a scenario where one replica is down, the system becomes unavailable for read operations that require all replicas to be responsive.

Alternative: QUORUM

QUORUM is a more balanced approach. It returns data as long as it receives responses from a majority of replicas. For example, if you have 3 replicas and set CL=QUORUM, the system will return data once two out of three replicas respond. This means that even with one replica down, read operations can still proceed, maintaining higher availability.

Why QUORUM?
Availability: With QUORUM, your application remains available for reads as long as a majority of replicas are up.
Latency: Quorum-based reads generally have lower latency compared to all-consistency-level reads because not every replica needs to confirm the request.
Risk Management: Setting CL=QUORUM is a safer and more practical choice in most scenarios, balancing between high consistency and maintaining service availability.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, setting CL=ALL for reads can lead to unavailability when any replica fails, which might not be ideal. Instead, using QUORUM provides a good balance by ensuring that the system remains highly available even with partial failures.

#### Common mistakes
include:

Overusing CL=ALL for all read operations without considering availability.
Not fully understanding the implications of different consistency levels on performance and reliability.

#### Follow-up
questions might involve discussing specific failure scenarios, how to handle write operations, or exploring other advanced configuration options like hinted handoffs and anti-entropy repair mechanisms.

**Comparison:**

```
Consistency Level	Replicas Required	Availability Impact
ALL	3 (all)	Unavailable
QUORUM	>2/3	Available
```

**Code:**

```csharp
// Example of setting consistency level in CQL for Cassandra
// SET CLIENT_CONSISTENCY ALL;
```

**Example input / output:**

```text
Input:
- Read request with CL=ALL

Output (with one replica down):
- System becomes unresponsive for read operations

Comparison:
- CL=QUORUM: read operation completes if two out of three replicas are up.
```

**Data:**

```json
{
  "replicas": {
    "total": 3,
    "down": 1,
    "quorum": 2
  },
  "readConsistency": {
    "ALL": "all replicas must respond",
    "QUORUM": "majority of replicas must respond"
  }
}
```

**Flow:**

1. Yes
2. No
3. Read Request
4. Replicas Down?
5. System Unresponsive
6. Return Data from MajorityReplicas

