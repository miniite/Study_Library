---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd architect a chargeback model that's fair when compute, storage, and data-transfer are all shared and gameable. What query-tagging, storage-attribution, and cross-domain-read accounting design makes a domain's bill reflect its true cost and can't be shifted onto a shared pool?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To architect a fair chargeback model in environments where compute, storage, and data transfer are shared and potentially gameable, we need to implement a multi-layered approach that involves query tagging, precise storage attribution, and cross-domain read accounting. This ensures each domain's bill reflects its true cost without shifting burdens onto the shared pool.

#### Triaging the Question

The question requires us to design a chargeback system that can accurately attribute costs across multiple domains in a fair manner, even when resources like compute, storage, and data transfer are shared among them. The core challenge is ensuring that each domain's bill accurately reflects its usage without allowing one domain to game the system.

Key Assumptions
Shared Resources: Compute, storage, and data transfer are shared among multiple domains.
Gameability: Domains might attempt to manipulate their chargeback by overloading certain resources or underutilizing others.
Multi-Domain Environment: There are multiple independent domains sharing the same infrastructure.

#### First-Principles Solution

To achieve a fair and accurate chargeback model, we need to address three main aspects:

Query Tagging: Ensure that each query is tagged with metadata about its originating domain.
Storage Attribution: Precisely track storage usage by individual domains.
Cross-Domain Read Accounting: Implement mechanisms to account for cross-domain read operations.
Query Tagging
Metadata Enrichment: Each query should be enriched with a unique identifier that represents the originating domain. This can be achieved using tags like domain_id or user_id.
Logging and Monitoring: Maintain detailed logs of all queries, including timestamps, query types (read/write), and associated metadata.
Storage Attribution
Granular Partitioning: Use storage partitions or directories that are uniquely identifiable to each domain. For example, each domain could have its own subdirectory in a shared file system.
Usage Metrics: Implement metrics to monitor and track the amount of data stored by each domain. This can be done through periodic scans or incremental updates.
Watermarks for Tracking: Use watermarks to mark changes in storage usage over time, ensuring accurate billing based on historical data.
Cross-Domain Read Accounting
Read Logs: Maintain detailed logs of all cross-domain read operations, recording the origin and destination domains along with timestamps.
Rate Limiting and Quotas: Implement rate limiting and quotas to control access between domains. This prevents one domain from overloading another.
Audit Trails: Ensure that every read operation is auditable, providing a record for billing purposes.

#### Implementation
Details

Database Design:

Create tables to store query logs with columns like domain_id, query_type, timestamp.
Implement partitions in storage systems based on domain identifiers.
Use audit logs and metadata to track cross-domain read operations.

Query Tagging Implementation:

Integrate tagging mechanisms at the database layer using SQL extensions or middleware.
For example, using PostgreSQL's SET command to set domain_id before executing queries.

Storage Attribution Implementation:

Use file system metadata or object storage tags to associate files with specific domains.
Implement background jobs to scan and update domain-specific usage metrics.

Cross-Domain Read Accounting Implementation:

Extend the query tagging mechanism to include read operations, logging source and destination domains.
Implement rate limiting policies using middleware or network firewalls.

#### Closing Summary

By implementing a combination of query tagging, storage attribution, and cross-domain read accounting, we can ensure that each domain’s bill accurately reflects its true cost. This approach mitigates the risk of gameability by providing detailed tracking and auditing mechanisms. The design is robust enough to handle shared resources while maintaining fairness across domains.

#### Common Mistakes/Follow-Ups
Overlooking Gameability: Failing to consider how users might manipulate resource usage to reduce their bill.
Insufficient Logging: Not maintaining comprehensive logs that can be used for accurate billing and auditing.
Lack of Granularity in Metrics: Using overly broad metrics instead of tracking fine-grained usage, leading to inaccurate chargeback.
Poor Design for Cross-Domain Operations: Failing to implement rate limiting or quotas, which can lead to abuse.

For a deeper dive, we could explore advanced techniques like machine learning models to predict and optimize resource allocation, but the core architecture remains robust as described above.

**Comparison:**

```
Timestamp	Source Domain ID	Destination Domain ID
2023-10-01T16:00:00	domain1	domain2
2023-10-01T17:00:00	domain2	domain1
```

**Code:**

```

-- Example of query tagging in PostgreSQL SET domain_id TO 'domain1'; SELECT * FROM some_table;

CODE
Copy

```python
# Example of logging cross-domain read operations in Python
def log_read(domain_id_source, domain_id_destination):
    with open('read_logs.txt', 'a') as f:
        timestamp = datetime.now().isoformat()
        f.write(f"{timestamp} | {domain_id_source} -> {domain_id_destination}\n")
```

**Example input / output:**

```text
Input:
- Domain A writes data to a shared storage.
- Domain B reads data from the same storage.

Output:
- Storage usage logs for both domains are updated accurately.
- Cross-domain read log shows that Domain B read data from Domain A.
- Query logs tag each operation with domain metadata.
```

**Data:**

```json
{
  "query_log": [
    {
      "id": 1,
      "domain_id": "domain1",
      "query_type": "write",
      "timestamp": "2023-10-01T14:00:00"
    },
    {
      "id": 2,
      "domain_id": "domain2",
      "query_type": "read",
      "timestamp": "2023-10-01T15:00:00"
    }
  ],
  "storage_usage": [
    {
      "domain_id": "domain1",
      "size_in_bytes": 1024,
      "last_updated": "2023-10-01T16:00:00"
    },
    {
      "domain_id": "domain2",
      "size_in_bytes": 512,
      "last_updated": "2023-10-01T17:00:00"
    }
  ],
  "read_logs": [
    {
      "timestamp": "2023-10-01T16:00:00",
      "source_domain_id": "domain1",
      "destination_domain_id": "domain2"
    },
    {
      "timestamp": "2023-10-01T17:00:00",
      "source_domain_id": "domain2",
      "destination_domain_id": "domain1"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query
4. Is Tagged?
5. Tagged with Domain ID
6. Execute Query
7. Log Query in Database
8. Read Operation
9. Is Cross-Domain?
10. Record Cross-Domain ReadLog
11. Bill Based on Logs and Metrics
12. Ignore or Rate Limit
13. Storage Attribution Update
14. Update Storage Usage Metrics
15. Billing System

