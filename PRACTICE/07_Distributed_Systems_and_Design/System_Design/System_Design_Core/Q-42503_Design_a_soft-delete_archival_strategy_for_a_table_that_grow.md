---
type: "practice"
difficulty: "Deep"
---

# Design a soft-delete + archival strategy for a table that grows 50M rows/month. Compare partitioning by time, a separate cold store, and TTL, on query cost, storage cost, and GDPR deletion.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective soft-delete + archival strategy for handling rapid growth (50M rows/month) while ensuring compliance with GDPR deletion requirements, we need to consider a balance between query costs, storage costs, and operational complexity. The primary strategies to evaluate are partitioning by time, using a separate cold store, and leveraging Time-to-Live (TTL). We will compare these approaches based on the criteria of query cost, storage cost, and GDPR compliance.

First, let's triage the question: The interviewer is probing our understanding of different archival and soft-delete strategies, their trade-offs in terms of performance and cost, and how to ensure data deletion complies with regulatory requirements like GDPR. We will make the following assumptions:

The table contains sensitive user data.
The database system supports partitioning, TTL, and a cold store.
The query patterns are read-heavy for recent data but can tolerate delays for archival data.

Now, let's reason through each strategy step by step:

Time Partitioning

#### Overview
Time partitioning involves dividing the table into smaller segments based on time intervals (e.g., daily or monthly partitions).

Query Cost: Efficient due to reduced scan range.
Storage Cost: Potentially higher as each partition may have overhead.
GDPR Deletion: Easy to implement by dropping specific partitions, but requires careful management of the deletion window.

#### First Principles Solution

Partition the table into time intervals (e.g., daily).
Implement a soft-delete mechanism that marks rows for deletion without physically removing them.
Periodically clean up the marked rows during maintenance windows.
Use TTL to automatically delete records older than a certain period, but configure it carefully to avoid GDPR non-compliance.
Separate Cold Store

#### Overview
Store archival data in a separate, optimized environment with reduced query performance but lower storage costs and easier compliance management.

Query Cost: Higher due to additional latency, but can be mitigated through caching.
Storage Cost: Lower as the cold store can use less expensive storage tiers.
GDPR Deletion: Easier to manage due to separation of hot and cold data, allowing for more straightforward deletion processes.

#### First Principles Solution

Define a threshold age (e.g., 3 months) after which data should be moved to the cold store.
Implement a soft-delete mechanism in both the main table and the cold store.
Periodically move aged records from the main table to the cold store using batch processes.
Ensure GDPR compliance by regularly reviewing and deleting data beyond retention periods.
TTL

#### Overview
Use Time-to-Live (TTL) to automatically delete records after a certain period, which simplifies management but may impact performance for read-heavy workloads.

Query Cost: Varies; depends on the number of active rows at any given time.
Storage Cost: Lower as deleted data is eventually removed from storage.
GDPR Deletion: Potentially complex to manage due to varying deletion windows and risk of accidental data retention beyond GDPR requirements.

#### First Principles Solution

Set TTL for records based on the retention period required by compliance (e.g., 2 years).
Implement soft-delete within the main table to capture deletions prior to TTL expiration.
Monitor and adjust TTL settings as needed to balance performance and cost.
Use logs or metadata to track deleted data, ensuring it is removed from storage promptly.

#### Closing Summary

In summary, each strategy has its strengths and weaknesses:

Time Partitioning offers efficient query performance but requires careful management of deletion windows.
Separate Cold Store provides lower storage costs and easier compliance but introduces additional latency for queries.
TTL simplifies management by automatically deleting data, but may impact read performance.

The choice depends on the specific requirements of your application. For a backend system with high query volume and strict GDPR compliance, a combination of time partitioning and separate cold store might offer the best balance between performance, cost, and compliance.

#### Common Mistakes/Follow-Ups
Overlooking GDPR Compliance: Ensure that data deletion processes strictly adhere to regulatory requirements.
Ignoring Performance Impact: Be mindful of how different strategies affect query performance, especially in read-heavy scenarios.
Inadequate Data Tracking: Properly log and track deleted data to ensure compliance and maintain audit trails.

**Comparison:**

```
Partition Key	Type	Description
date	timestamp	Time-based partitioning
user_id	int	User identifier
Cold Store Settings	Value
Age Threshold (days)	90
Data Type	Archive
```

**Code:**

```
Example partitioning logic (simplified)

def partition_table_by_time(table, interval='daily'): # Implement partitioning based on time intervals pass

def apply_soft_delete(table): # Mark rows for deletion without physically removing them table.update(set('deleted', True))

CODE
Copy

```python
# Example TTL implementation
def set_tll_for_records(records, ttl_days=730):
    # Set Time-to-Live (TTL) on records based on retention period
    pass

def clean_up_expired_data():
    # Periodically remove data beyond the TTL window
    pass
```

**Example input / output:**

```text
# Example input -> output for time partitioning
Input:
- Table with 50M rows, 1M records per day

Output:
- Daily partitions created (50)
- Soft delete mechanism applied to mark rows for deletion

# Example input -> output for separate cold store
Input:
- Threshold age set to 90 days

Output:
- Aged data moved to cold store after 90 days
- Separate logs maintained for cold store access and deletions
```

**Data:**

```json
{
  "partition_keys": [
    { "name": "date", "type": "timestamp" },
    { "name": "user_id", "type": "int" }
  ],
  "cold_store_threshold": {
    "age_days": 90,
    "data_type": "archive"
  },
  "tll_settings": {
    "retention_period_days": 730
  }
}
```

**Flow:**

1. Start
2. Is Time Partitioning Needed?
3. Create Daily Partitions
4. Apply Soft Delete Mechanism?
5. Mark Rows for Deletion
6. Clean Up Expired Data?
7. Periodic Maintenance
8. Is Cold Store Needed?
9. Move Aged Data to Cold Store
10. Log and Monitor Cold StoreAccess?
11. Ensure GDPR Compliance

