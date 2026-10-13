---
type: "practice"
difficulty: "Deep"
---

# How do you avoid duplicated ingestion where five domains each independently pull the same upstream operational source?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Ownership vs Cross-cutting Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of avoiding duplicated ingestion in a Data Mesh architecture, we need to ensure that multiple domains pulling from an upstream operational system do not ingest the same data twice. The key idea is to implement a central coordination mechanism that tracks which parts of the data have already been ingested by each domain. This involves designing a reliable data catalog and implementing a distributed lock or versioning system.

#### Triage

The question is probing our understanding of how Data Mesh can be scaled while maintaining data integrity. We need to consider assumptions such as the scale of domains, the frequency of data updates, and the available infrastructure for tracking data ingestion. Common mistakes include overlooking the consistency requirements and assuming that each domain can independently manage its own state without causing conflicts.

#### First-Principles Solution
Centralized Data Catalog: Implement a centralized data catalog that maintains metadata about all ingested data. This catalog should provide an overview of what data is available, when it was last updated, and which domains have already processed it.
Distributed Locking Mechanism: Use a distributed locking mechanism to ensure that only one domain can process a specific piece of data at any given time. This prevents multiple domains from ingesting the same data simultaneously.
Versioned Data Ingestion: Each piece of data should be versioned and tagged with metadata such as the domain that processed it, the timestamp, and the source system. This helps in tracking provenance and avoiding duplicates.

#### Implementation
Steps

Metadata Schema Design:

Define a schema for storing metadata about each ingested piece of data.
Include fields like data_id, domain_name, ingestion_time, and source_system.

Data Ingestion Workflow:

Yes
No
Operational Source
Is Data Fresh?
Process Data
Update Metadata
Wait for Next Batch
Distribute Lock
Release Lock
The operational source sends data to a processing pipeline.
A validation step checks if the data is fresh. If not, it waits or discards.
If fresh, it processes and updates metadata in the catalog.
It acquires a distributed lock before updating the metadata to ensure only one domain can update it at a time.

Lock Management:

Use a service like Redis or Consul for managing distributed locks.
Implement a timeout mechanism to handle cases where a lock might be stuck (e.g., due to a failed domain).

Data Provenance Tracking:

No
Yes
Operational Source
Is Data Fresh?
Wait for Next Batch
Process Data
Distribute Lock
Release Lock
Update Metadata
Log Ingestion Details
Log the details of each ingestion, including which domain ingested it and when.

Monitoring and Alerting:

Set up monitoring to track lock acquisition times and ensure that no locks are stuck indefinitely.
Implement alerting for any issues in the data catalog or processing pipeline.

#### Closing Summary

By implementing a centralized metadata catalog, distributed locking, versioned data ingestion, and robust monitoring, we can effectively avoid duplicated ingestion in a Data Mesh architecture. This approach ensures data integrity while allowing multiple domains to operate independently and efficiently.

#### Common Mistakes/Follow-Ups
Mistake: Failing to implement proper monitoring and alerting for lock management.

#### Follow-Up
How would you handle scenarios where the operational source is updated more frequently than the processing capacity allows?
Mistake: Overlooking the need for a robust distributed locking mechanism, leading to race conditions.

#### Follow-Up
What strategies can be used to ensure that locks are not held indefinitely and how do they impact performance?

**Comparison:**

```
Data Ingestion	Domain 1	Domain 2
Data ID	uuid	-
Ingestion Time	2023-10-05T14:48:00Z	-
Source System	ops_source_1	-
Metadata Entry Update	Domain Name	Ingestion Time	Lock Acquired By
uuid	example_domain	2023-10-05T14:48:00Z	example_domain
```

**Code:**

```python
# Example Python code for acquiring and releasing a lock using Redis
import redis
from contextlib import contextmanager

redis_client = redis.Redis(host='localhost', port=6379, db=0)

@contextmanager
def acquire_lock(key):
    while True:
        acquired = redis_client.setnx(key, '1')
        if acquired:
            try:
                yield
            finally:
                redis_client.delete(key)
        else:
            print("Failed to acquire lock")
```

**Example input / output:**

```text
Input: Operational source sends data with ID = uuid.
Output: Metadata entry is updated in the catalog, and a lock is acquired by example_domain.
```

**Data:**

```json
{
  "data_id": "uuid",
  "domain_name": "example_domain",
  "ingestion_time": "2023-10-05T14:48:00Z",
  "source_system": "ops_source_1"
}
```

**Flow:**

1. No
2. Yes
3. Operational Source
4. Is Data Fresh?
5. Wait for Next Batch
6. Process Data
7. Distribute Lock
8. Release Lock
9. Update Metadata
10. Log Ingestion Details

