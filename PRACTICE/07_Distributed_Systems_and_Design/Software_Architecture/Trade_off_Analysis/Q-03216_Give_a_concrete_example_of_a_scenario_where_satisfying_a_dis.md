---
type: "practice"
difficulty: "Expert"
---

# Give a concrete example of a scenario where satisfying a disaster-recovery RPO goal directly trades off against write latency, and walk through how you would present that tradeoff point with numbers rather than adjectives.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sensitivity Points](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a database system that meets a specific Recovery Point Objective (RPO) of 15 minutes in case of a disaster, while minimizing write latency. The core trade-off is between achieving a tight RPO and ensuring writes are processed as quickly as possible. Let's walk through how I would analyze and present this tradeoff.

Firstly, we need to understand the implications of an RPO goal. An RPO of 15 minutes means that if a disaster occurs, any data written in the last 15 minutes might be lost. This is critical for ensuring business continuity and regulatory compliance. On the other hand, write latency refers to how quickly changes are committed to storage, which directly impacts user experience and system responsiveness.

To present this tradeoff quantitatively, we can use a simple example with concrete numbers. Assume our database handles 1000 write operations per second (WOPS). At a RPO of 15 minutes, we must ensure that any write operation within the last 15 minutes is backed up to another system or medium.

Let’s break down the tradeoff:

Step-by-Step Analysis

Calculate the Number of Writes in 15 Minutes:

There are 900 seconds in 15 minutes.
With 1000 WOPS, this translates to ( 1000 \times 900 = 900,000 ) writes.

Determine Backup Mechanism Latency:

Assume the backup system introduces a latency of 50 milliseconds (ms). This is not an unrealistic value for network and processing delays.
For every write operation, we need to account for this additional delay before it can be considered fully committed.

Calculate Write Latency Impact:

With each write taking an extra 50 ms due to the backup process, the effective write latency increases from zero to 50 ms.
For a system with high concurrency (1000 WOPS), this small increase can significantly affect overall performance.
Presenting the Tradeoff

To present this tradeoff point, we would use a table and some basic calculations:

Metric	Without Backup	With 50ms Latency Due to Backup
WOPS	1000	998.7
Backup Latency	0 ms	50 ms
RPO	Not guaranteed (RPO > 15m)	15 minutes

In the first scenario, without a backup mechanism, we might not meet our RPO of 15 minutes. In the second scenario, with a 50ms latency due to backup, writes are committed faster but at the cost of meeting the strict RPO.

Visualizing the Tradeoff

To further illustrate this tradeoff, we could use a simple flowchart:

Yes
No
Write Operation
Is within last 15m?
Backup to Secondary System
Commit Write
Directly Commit Write

In this flowchart, the path through "C" represents the additional delay due to backup. This visual helps in understanding where and how the tradeoff occurs.

#### Closing Summary

By quantifying the write latency impact (50ms) on 1000 WOPS, we have demonstrated that achieving a strict RPO of 15 minutes introduces a noticeable increase in write latency. The tradeoff is clear: a tight RPO is achieved at the cost of increased write latency.

#### Common Mistakes
and Follow-Ups
Overlooking Network Delays: Often, developers focus solely on processing delays and ignore network latencies.
Assuming No Impact on User Experience: While 50ms might seem small, it can have significant implications in high-concurrency systems.
Neglecting Human Factors: Users may notice performance degradation even with minor increases in latency.

Understanding this tradeoff point is crucial for making informed architectural decisions and ensuring that the system meets both business and technical requirements.

**Code:**

```
MERMAID
Copy
graph TD
    A[Write Operation] --> B[Is within last 15m?]
    B -->|Yes| C[Backup to Secondary System]
    C --> D[Commit Write]
    B -->|No| E[Directly Commit Write]
```

**Example input / output:**

```text
Input:
- Write Operations Per Second (WOPS): 1000
- Seconds in 15 minutes: 900
- Writes within RPO: 900,000

Output:
- Backup Latency: 50 ms
- Effective WOPS with Backup: 998.7
```

**Data:**

```json
{
  "write_operations_per_second": 1000,
  "seconds_in_15_minutes": 900,
  "writes_within_rpo": 900000,
  "backup_latency_ms": 50,
  "effective_wops_with_backup": 998.7
}
```

