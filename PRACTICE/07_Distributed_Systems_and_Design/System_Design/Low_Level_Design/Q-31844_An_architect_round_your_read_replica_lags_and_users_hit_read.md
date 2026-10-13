---
type: "practice"
difficulty: "Expert"
---

# An architect round: your read replica lags and users hit read-your-writes bugs after a POST; propose an org-wide class-level pattern (version token / sticky-to-primary) that gives read-your-writes consistency without forcing all reads to primary, and its cost.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the issue of read replica lags leading to "read-your-writes" bugs after a POST operation in an organization, we need to design a mechanism that ensures consistency without forcing all reads to go through the primary database. One effective approach is to implement a version token or sticky-to-primary pattern at the class level. This solution leverages optimistic concurrency control and maintains performance by reducing the load on the primary database.

#### Triage
Revisiting the Problem

The question probes several aspects:

Understanding of Read-Replica Consistency: How read replicas can lag behind the primary.
Handling Write Operations: Ensuring that writes are consistent across all nodes, especially when reads from replicas might occur.
Pattern Implementation: Designing a class-level pattern to manage version tokens or sticky-to-primary behavior efficiently.

#### Assumptions
The system uses a distributed architecture with multiple read replicas and one primary database.
Write operations are frequent and critical for consistency.
Reads are predominantly non-critical but can occur at any time from either the primary or replicas.

#### First Principles Solution
Step 1: Version Tokens

Version tokens act as a mechanism to track write operations on the primary. Each write operation increments a version counter, which is then stored in the database along with the updated data. Replicas can check this version token before performing any read operation to ensure that they are not reading stale data.

Step 2: Sticky-to-Primary

This pattern ensures that after a successful write operation, reads for that specific class or object must be directed back to the primary until a certain period has passed, or until the replica's version catches up. This prevents "read-your-writes" bugs by ensuring that all subsequent reads are performed on an updated and consistent data set.

Step 3: Implementation Details
Version Token Storage: Store a version token with each write operation in the database.
Replica Version Check: Before performing any read operation, replicas check their current version against the latest version stored in the primary.
Sticky-to-Primary Mechanism: After writing to the primary, mark the object or class as "sticky" for a certain period (e.g., 10 minutes) to ensure all reads are directed back to the primary.
Cost Analysis

Implementing this pattern incurs some costs:

Storage Overhead: Storing version tokens and sticky status adds minimal overhead.
Network Latency: Replicas must communicate with the primary for version checks, which can introduce network latency.
Primary Load: Initially, there might be a slight increase in the load on the primary due to frequent writes and subsequent read redirections.

#### Conclusion

The version token/sticky-to-primary pattern offers a balanced approach to maintaining consistency while reducing the load on the primary. By ensuring that writes are properly tracked and reads are redirected appropriately, this solution minimizes "read-your-writes" bugs without forcing all reads to go through the primary continuously.

#### Common Mistakes/Follow-Ups
Ignoring Replication Lag: Failing to account for significant replication lag can lead to inconsistent data.
Overly Aggressive Sticky Periods: Setting sticky periods too long can increase network latency and load on the primary unnecessarily.
Lack of Graceful Degradation: Not having a fallback mechanism in case the version token or sticky state is corrupted.

**Code:**

```python
# Example Python code snippet for setting version tokens
def write_to_database(data):
    version_token = get_latest_version()
    data['version'] = version_token + 1
    db.write(data)
    mark_as_sticky(obj_id, version_token)

def read_from_database(obj_id):
    primary_version = db.get_primary_version(obj_id)
    if is_sticky(obj_id) and primary_version > get_local_version():
        redirect_to_primary_and_read()
    else:
        replica_data = db.read_replica(obj_id)
        return replica_data
```

**Example input / output:**

```text
Input:
- Write to database with data {obj_id: '123', version: 4}

Output:
- Version token incremented and marked as sticky for obj_id

Input:
- Read from replica for obj_id '123'

Output:
- Redirected to primary due to sticky state
```

**Data:**

```json
{
  "obj_id": "123",
  "version": 5,
  "sticky": true
}
```

