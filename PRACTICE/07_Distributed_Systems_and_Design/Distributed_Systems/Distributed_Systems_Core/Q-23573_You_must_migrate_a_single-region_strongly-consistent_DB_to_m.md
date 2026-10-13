---
type: "practice"
difficulty: "Expert"
---

# You must migrate a single-region strongly-consistent DB to multi-region while preserving existing consistency contracts on the endpoints that need them. Explain which reads/writes you can relax to regional, which must stay global, and how you migrate without a correctness regression.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a single-region strongly-consistent database (DB) to multi-regions while preserving existing consistency contracts, we need to carefully consider the nature of reads and writes in our application. The core challenge is to ensure that critical operations remain globally consistent while allowing more tolerant operations to be regionalized. This involves understanding the trade-offs between availability, latency, and consistency.

In this scenario, we have a single-region DB with strong consistency guarantees, meaning every read operation sees the most recent write from any part of the system. To migrate to multi-regions, we need to:

Identify which parts of our application require global consistency.
Determine which operations can tolerate eventual or regional consistency.
Design an architecture that allows for a smooth migration with minimal disruption.

Firstly, let's define what constitutes reads and writes in the context of this problem:

Reads: Any operation retrieving data from the database.
Writes: Any operation updating data in the database.

**Comparison:**

```
Operation Type	Consistency Level
Read	Global
Write	Global
Local Read	Regional
Local Write	Regional
```

**Code:**

```python
# Example code to illustrate read and write operations
class DatabaseOperation:
    def global_read(self, key):
        pass

    def global_write(self, key, value):
        pass

    def regional_read(self, region_id, key):
        pass

    def regional_write(self, region_id, key, value):
        pass
```

**Example input / output:**

```text
- **Input**: A user performs a read operation on an item with a globally consistent requirement.
- **Output**: The system returns the most recent version of the data from the primary region.

- **Input**: A user performs a write operation on an item with a globally consistent requirement.
- **Output**: The system ensures that this write is replicated to all regions before confirming the transaction.

- **Input**: A read operation requested by a regional endpoint (local to the user).
- **Output**: The system returns cached data or retrieves it from the local region if available; otherwise, it performs a global read.
```

**Data:**

```json
{
  "operations": [
    {
      "operation_type": "read",
      "consistency_level": "global"
    },
    {
      "operation_type": "write",
      "consistency_level": "global"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Initiates Read
6. Is Consistency Level Global?
7. Query Primary Region
8. Query Local Region
9. No Data, Query PrimaryRegion & Cache
10. User Initiates Write
11. Is Consistency Level Global?
12. Replicate to All Regions BeforeConfirmation
13. Local Write, No ReplicationRequired

