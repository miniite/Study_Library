---
type: "practice"
difficulty: "Expert"
---

# Design a zero-downtime migration off a proprietary vendor database to an open engine at petabyte scale — how do you sequence dual-writes, parity verification, and the point of no return?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a zero-downtime migration from a proprietary vendor database to an open database system at petabyte scale, we need to focus on maintaining data consistency and availability throughout the process. The key components of this solution include dual-writes, parity verification, and establishing a point of no return. We'll approach this problem step-by-step using first principles.

Firstly, let's understand what the question is really asking:

Dual-Writes: Ensure that every write operation in the old database is mirrored to the new open engine.
Parity Verification: Verify the integrity and consistency of data during the migration process.
Point of No Return: Define a definitive point where we can no longer revert, ensuring data safety.

#### Assumptions
Both databases support high availability and replication features.
The current database schema is known and can be replicated accurately.
Network bandwidth and latency are within acceptable limits.
There is sufficient storage capacity for the open engine to handle petabyte-scale data.
Monitoring tools and logging are available to track migration progress.

#### Solution
Step 1: Dual-Writes

To achieve zero downtime, we need a mechanism where every write operation in the proprietary database is simultaneously mirrored to the new open engine. This can be achieved using a distributed log (such as Apache Kafka) or a transactional queue system that ensures idempotency and retries.

QueueSystem
OpenEngine
ProprietaryDB
QueueSystem
OpenEngine
ProprietaryDB
alt
[Idempotent write]
alt
[-
F-
a-
i-
l-
u-
r-
e
r-
e-
c-
o-
v-
e-
r-
y-
]
Write operation
Send to OpenEngine
Acknowledge
Step 2: Parity Verification

Once the dual-writes are in place, we need to verify that the data is consistent between both databases. This involves setting up a parity check system where periodic snapshots or checksums of the data are compared.

ParityChecker
OpenEngine
ProprietaryDB
ParityChecker
OpenEngine
ProprietaryDB
alt
[Success]
alt
[Failure]
Snapshot request
Get snapshot
Send snapshot
Verify parity
Notify failure
Step 3: Point of No Return

The point of no return is crucial to ensure that once the data migration reaches a certain level of completeness, it cannot be easily reverted. This can be achieved by implementing a final synchronization step where all remaining inconsistencies are resolved.

Synchronizer
OpenEngine
ProprietaryDB
Synchronizer
OpenEngine
ProprietaryDB
alt
[Success]
alt
[Failure]
Mark for sync
Sync request
Data synced
Acknowledge
Notify failure

#### Closing Summary
and Common Mistakes/Follow-Ups

The key to a successful zero-downtime migration is ensuring that the dual-writes are reliable, parity verification catches all issues early, and the point of no return prevents any backtracking. Common mistakes include:

Failing to account for edge cases: For example, handling partial failures during dual-writes.
Overlooking network latency: Ensuring that the write operations propagate within acceptable time frames.
Neglecting monitoring and logging: Properly tracking the health and progress of the migration.

#### Follow-ups
might include discussing how to handle large-scale data consistency checks, or exploring advanced techniques like differential backups for faster recovery points. Additionally, scaling considerations such as sharding and distributed transaction management would be important in a petabyte-scale environment.

**Comparison:**

| Operation | Source Database | Target Database | Action |
|-----------|-----------------|-----------------|--------|
| Dual-Writes | ProprietaryDB | OpenEngine | Send write operation to both databases. |
| Parity Verification | ProprietaryDB, OpenEngine | ParityChecker | Request and compare snapshots for consistency. |
| Point of No Return | Synchronizer | OpenEngine | Ensure all data is fully synced before migration completion. |

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant ProprietaryDB
    participant OpenEngine
    participant QueueSystem

    ProprietaryDB ->> QueueSystem: Write operation
    alt Idempotent write
    QueueSystem ->> OpenEngine: Send to OpenEngine
    OpenEngine --> QueueSystem: Acknowledge
    end
    alt Failure recovery
    OpenEngine --|> QueueSystem: Retry
    end

sequenceDiagram
    participant ProprietaryDB
    participant OpenEngine
    participant ParityChecker

    ProprietaryDB ->> ParityChecker: Snapshot request
    alt Success
    ParityChecker -->> OpenEngine: Get snapshot
    OpenEngine --> ParityChecker: Send snapshot
    ParityChecker -->> ProprietaryDB: Verify parity
    end
    alt Failure
    ParityChecker -->> ProprietaryDB: Notify failure
    end

sequenceDiagram
    participant ProprietaryDB
    participant OpenEngine
    participant Synchronizer

    ProprietaryDB ->> Synchronizer: Mark for sync
    Synchronizer -->> OpenEngine: Sync request
    alt Success
    OpenEngine --> Synchronizer: Data synced
    Synchronizer -->> ProprietaryDB: Acknowledge
    end
    alt Failure
    Synchronizer -->> ProprietaryDB: Notify failure
    end
```

**Example input / output:**

```text
Input: Write operation to ProprietaryDB
Output:
1. QueueSystem receives the write and sends it to OpenEngine.
2. OpenEngine acknowledges the write and returns a confirmation to QueueSystem.
3. ParityChecker schedules a snapshot check after a certain period.
4. ParityChecker requests snapshots from both databases for comparison.
5. If discrepancies are found, ParityChecker logs the issues and sends notifications.
6. Upon reaching the point of no return, Synchronizer triggers final data sync between databases.
```

**Data:**

```yaml
migration:
  dual_writes:
    - ProprietaryDB -> QueueSystem
    - QueueSystem -> OpenEngine
  parity_verification:
    - ParityChecker -> OpenEngine
    - OpenEngine -> ParityChecker
  point_of_no_return:
    - Synchronizer -> OpenEngine
```

**Flow:**

1. Yes
2. No
3. Success
4. Failure
5. ProprietaryDB Write
6. QueueSystem
7. Acknowledge?
8. OpenEngine Write
9. Failure Recovery
10. ParityChecker RequestSnapshot
11. Get Snapshots
12. Verify Parity
13. Log Success
14. Notify Failure
15. Synchronizer Mark for Sync
16. OpenEngine Data Sync
17. Acknowledge Sync

